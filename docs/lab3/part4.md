# xv6中Lock的实现（选读）

&emsp;&emsp;这部分深入探讨了锁的基本原理、实现方法和使用方法，可能有些难以理解，超出了本实验的水平要求，但仍然在知识范围内，因此可供感兴趣的同学进行参考。

## 1. 锁的基本实验原理

&emsp;&emsp;`kalloc.c`中调用`acquire()`和`release()`来获取锁和释放锁。首先需要知道的是锁的`acquire`（或者叫`P`）和`release`（`V`）的实现并不是一件简单的事情。我们先来看一个最`naive`的实现：

```c   linenums="1"
void acquire(struct spinlock *lk) // does not work!
{
	for(;;) {
		if(lk->locked == 0) {
			lk->locked = 1;
			break;
		}
	}
}
```

&emsp;&emsp;这个代码非常容易理解，如果`lk`被锁上了，那就继续循环，如果`lk`未被锁上，那就给它加锁并退出。但这个实现是有缺陷的，让我们试想一下以下场景：

!!! note   "某个错误场景"
    - `lk`一开始没有被锁上，即`lk->locked = 0`; 
    - 然后`CPU0`在跑进程`A`而`CPU1`在跑进程`B`，`A`和`B`同时对这个锁发起了`acquire`；
    - 我们所希望看到的是，锁被且只被`A`或`B`中的一个进程锁住，而另一个进程则继续在`for`中循环等待。
    - 但是，假设`CPU0`和`CPU1`同时执行到了第6行，那么会发生什么呢？
    - 没错，两个`CPU`都发现`lk->locked == 0`，因为另一个`CPU`还没有来得及改动`lk->locked`;
    - 这就会导致`A`和`B`同时获得锁，并从`acquire`中返回，这显然不是我们希望看到的。


&emsp;&emsp;那`acquire`到底是怎么实现的呢？这个仅凭软件是无法实现的，需要硬件参与辅助实现。
在这里我们先介绍一条特殊的`CPU`指令`amoswap`。`amoswap`实现了在一条指令完成一次 `load` 和 `store`，更具体的说，就是可以将一个寄存器的值和某一内存地址的值做交换，将指定内存地址的值放入寄存器，同时将寄存器的值放入内存。此外，`CPU`还会保证某一`CPU`核在执行这一指令时，其他`CPU`核不能读或写对应的内存地址。

!!! tip   "`amoswap`指令和`__sync_lock_test_and_set`"

    在C语言中使用这一汇编指令的语句调用是`__sync_lock_test_and_set()`。
    这一函数接收两个参数，要写入的地址和要写入的值，返回值是写入地址原来的值。
    请仔细比较`amoswap`指令和`__sync_lock_test_and_set`行为上的相似之处，
    确保你已经大致知道`__sync_lock_test_and_set`是如何使用amoswap实现功能。

&emsp;&emsp;了解了`__sync_lock_test_and_set`之后，我们在回到`acquire`的问题上来。先看看xv6中是如何实现`acquire`的。

```c   linenums="1"
// Acquire the lock.
// Loops (spins) until the lock is acquired.
void
acquire(struct spinlock *lk)
{
    push_off(); // disable interrupts to avoid deadlock.
    if(holding(lk))
        panic("acquire");
    __sync_fetch_and_add(&(lk->n), 1);
     // On RISC-V, sync_lock_test_and_set turns into anatomic swap:
     //   a5 = 1
     //   s1 = &lk->locked
     //   amoswap.w.aq a5, a5, (s1)
     while(__sync_lock_test_and_set(&lk->locked, 1) != 0) {
         __sync_fetch_and_add(&lk->nts, 1);
     }
  
     // Tell the C compiler and the processor to not moveloads or stores
     // past this point, to ensure that the criticalsection's memory
     // references happen after the lock is acquired.
     __sync_synchronize();
     // Record info about lock acquisition for holding() anddebugging.
     lk->cpu = mycpu();
}
```

&emsp;&emsp;我们需要注意的是14-16行的`while`循环。这里使用了`__sync_lock_test_and_set()`（使用`amoswap`），保证了对`lk->locked`读写的一致性。下面我们来分析以下具体的工作流程。

- 当`lk->locked==0`，即当前锁空闲时，`__sync_lock_test_and_set()`在返回0的同时，会往`lk->locked`写入1，即上锁。整个过程因为由`amoswap`实现，所以不会被其的CPU核所干扰。返回0之后退出`while`循环，符合我们的预期。
  
- 当`lk->locked==1`，即当前锁被锁定时，`__sync_lock_test_and_set`在返回1的同时，往`lk->locked`里写入了1。事实上在锁被锁定后我们不应该往`lk->locked`里写东西（当然，除了解锁的时候），但是因为锁本来就是1，再写入一个1相当于没有改变内容，所以也没差。同时，因为返回了1，我们知道当前锁被锁住了，所以会继续在`while`中循环。
  
- 至于第15行代码所做的事，这个和锁的功能无关，但当然它有另外的作用。这次不做解释，有兴趣的同学可以自行研究。

&emsp;&emsp;到这里对Lock的实现解释就结束了。在进行下一步的阅读前，强烈建议对上面的两个实现做比较和揣摩，直到你对为什么锁要使用`__sync_lock_test_and_set`有了清晰的认识。
    
注：以上解释可在[xv6 book](https://pdos.csail.mit.edu/6.828/2020/xv6/book-riscv-rev1.pdf)的Locking章节中找到。
    


## 2. 锁与kalloc

&emsp;&emsp;在前面的实验原理部分，我们已经解释了“`kalloc`在什么情况下使用了锁？”这个问题。那为什么要在操作`freelist`的时候上锁呢？上锁必然是为了防止并行运行的时候出问题，如果我们能设想出一场景（这个场景显然是并行情况下的）让没上锁的链表操作出问题，那就可以解释为什么要上锁了。

```c   linenums="65"
// Allocate one 4096-byte page of physical memory.
// Returns a pointer that the kernel can use.
// Returns 0 if the memory cannot be allocated.
void *
kalloc(void)
{
 struct run *r;

 acquire(&kmem.lock);
 r = kmem.freelist;
 if(r)
   kmem.freelist = r->next;
 release(&kmem.lock);

 if(r)
   memset((char*)r, 5, PGSIZE); // fill with junk
 return (void*)r;
}
```

!!! note   "某个错误场景"
    以`kalloc()`函数为例（见`kernel/kalloc.c`）。假设第73和77行的锁操作被注释掉了，那么就可能有两个`CPU`，记为`CPU0`和`CPU1`，同时执行到第74行。然后，两个`CPU`就会从`freelist`中拿出同一个内存块，这就会导致两个进程共用一块内存，但对于进程来说它认为它得到的内存是独享的，这样`A`进程再往这块内存中写数据时，会把`B`之前写进去的数据破坏掉，这显然不是我们所希望的。所以`kalloc()`里从`freelist`中取内存块的操作需要锁，`CPU0`在取的时候`CPU1`陷入等待，直到`CPU0`把`freelist`更新完解锁后，`CPU1`再进去取，从而保证每个内存块只被一个进程取走。

&emsp;&emsp;对`kfree`函数，请同学们自行想象一个没有锁会出现错误的场景，在此不再做讨论。
    
## 3. 锁与bio

&emsp;&emsp;在`bio.c`中，一共使用两种类型的锁：自旋锁`spinlock`（`bcache.lock`）和睡眠锁`sleeplock`（`b.lock`）。

### 3.1 **自旋锁**

&emsp;&emsp;*bcache.lock* 用于表示当前访问的`bcache`缓存块数据结构是否被锁住。 当`bcache.lock`为0时表示未锁住，能够访问当前数据结构`bcache`，如果为1，即暂时无法获得锁，则不断循环、自旋、等待锁重新可用。


- *bget()* 在操作`bcache`数据结构（修改`refcnt`、`dev`、`blockno`、`valid`）时，需要获取到自旋锁 `bcache.lock`，操作完成后再释放该锁。


- *brelse()* 需要获取到自旋锁 `bcache.lock`，才能将`refcnt`（引用计数）减1，且只有在`refcnt`为0时，将该数据缓存块插入到`bcache.head`链表后面，操作完成后再释放该锁。


- *bpin()和bunpin()* 获取到锁后，才能修改`refcnt`，操作完成后再释放该锁。
  
###  3.2 **睡眠锁** 

&emsp;&emsp;`b.lock`用于表示`bcache`缓存块数据结构中的当前缓存数据块`buf`是否被锁住，当`b.lock为1`时，则调用`sleep()`睡眠等待锁重新可用，为0则表示锁已经被释放。睡眠锁的三个接口函数如下：

- *acquiresleep()*：查询`b.lock`是否被锁，如果被锁了，就睡眠，让出CPU，直到`wakeup()`唤醒后，获取到锁，并将`b.lock`置1。

- *releasesleep()*：释放锁，并调用`wakeup()`。

- *holdingsleep()*：返回锁的状态（1：锁住或0：未锁）。

&emsp;&emsp;睡眠锁的使用：  

- *bget()* 在获取到缓存块（命中的缓存块，或者，未命中时通过`LRU`算法替换出来缓存中的缓存块）后，调用`acquiresleep()`获取睡眠锁。
- *bwrite()* 在写入到磁盘之前，先调用`holdingsleep()`查询是否已经获取到该睡眠锁，确保有带锁而入。
- *brelse()* 也先调用`holdingsleep()`查询是否已经获取到该睡眠锁，确保有带锁后，才调用`releasesleep()`释放该锁。
  

!!! note   "自旋锁和睡眠锁的区别"
    Spinlock  
          1. 在短时间内进行轻量级加锁;  
          2. 获取过程一直进行忙循环—旋转—等待锁重新可用（占用CPU时间长）；  
          3. xv6要求在持有`spinlock`期间，中断不允许发生。    

    sleep-lock  
          1. 适合长时间持有；  
          2. 获取锁期间可以让出`CPU`；    
          3. 持有`Sleep-lock`期间允许中断发生，但不允许在中断程序中使用。  



## 4. 测评程序kalloctest对锁的检测

&emsp;&emsp;测评程序会检查锁被阻塞的情况，还记得上文提到的`acquire`代码吗？

```c  linenums="1"
// Acquire the lock.
// Loops (spins) until the lock is acquired.
void
acquire(struct spinlock *lk)
{
    push_off(); // disable interrupts to avoid deadlock.
    if(holding(lk))
        panic("acquire");
    __sync_fetch_and_add(&(lk->n), 1);
     // On RISC-V, sync_lock_test_and_set turns into anatomic swap:
     //   a5 = 1
     //   s1 = &lk->locked
     //   amoswap.w.aq a5, a5, (s1)
     while(__sync_lock_test_and_set(&lk->locked, 1) != 0) {
         __sync_fetch_and_add(&lk->nts, 1);
     }
  
     // Tell the C compiler and the processor to not moveloads or stores
     // past this point, to ensure that the criticalsection's memory
     // references happen after the lock is acquired.
     __sync_synchronize();
     // Record info about lock acquisition for holding() anddebugging.
     lk->cpu = mycpu();
}
```

`lk->n`：即\#acquire，想要获取锁的次数。

`lk->nts`：即\#fetch-and-add，自旋（CPU不停循环等待解锁）的次数。

每当执行一次`acquire()`函数，第9行`__sync_fetch_and_add(&(lk->n), 1)`就会将`lk->n`加1。

当执行到第14行时，如果 `__sync_lock_test_and_set(&lk->locked, 1)`的返回值为1（即锁已被其他进程锁上），`lk->nts`的值就会加1。

测评程序会检查`lk->nts`的值。当`lk->nts`值接近0时，才能通过测试。所以这次实验中需要做的就是减少进程中的锁冲突，尽量让进程在获取锁时，锁的状态都是未被锁上的。
    

