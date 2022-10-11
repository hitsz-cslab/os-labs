# xv6中Lock的实现（选读）

这部分深入探讨了锁的基本原理、实现方法和使用方法，有些难以理解，超出了本实验的水平要求，但仍然在知识范围内，因此可供感兴趣的同学进行参考。

## 1. 锁的基本实验原理

kalloc.c中调用acquire()和release()来获取锁和释放锁。首先需要知道的是锁acquire（或者叫P）和release（V）的实现并不是一件简单的事情。我们先来看个最naive的实现：

```c
1. 	void acquire(struct spinlock *lk) // does not work!
2. 	{
3. 		for(;;) {
4.			if(lk->locked == 0) {
5.				lk->locked = 1;
6.				break;
7.			}
8.		}
1. 	}
```

这个代码非常容易理解，如果lk被锁上了，那就继续循环，如果lk未被锁上，那就它加锁并退出。但这个实现是有缺陷的，让我们试想一下以下场景：

!!! note   "某个错误场景"
    - lk一开始没有被锁上，即lk->locked = 0; 
    - 然后CPU0在跑进程A而CPU1在跑进程B，A和B同时对这个锁发起了acquire；
    - 我们所希望看到的是，锁被且只被A或B中的一个进程锁住，而另一个进程则继续for中循环等待。
    - 但是，假设CPU0和CPU1同时执行到了第6行，那么会发生什么呢？
    - 没错，两个CPU都发现lk->locked == 0，因为另一个CPU还没有来得及改lk->locked;
    - 这就会导致A和B同时获得锁，并从acquire中返回，这显然不是我们希望看到的。


那acquire到底是怎么实现的呢？这个仅凭软件是无法实现的，需要硬件参与辅助现。
在这里我们先介绍一条特殊的CPU指令amoswap。amoswap实现了在一条指令完成一次 load 和 store，更具体的说，就是可以将一个寄存器的值和某一内存地址的值做交换，将指定内存地址的值放入寄存器，同时将寄存器的值放入内存。此外，CPU还会保证某一CPU核在执行这一指令时，其他CPU核不能读或写对应的内存地址。

!!! tip   "amoswap指令和__sync_lock_test_and_set"

    在C语言中使用这一汇编指令的语句调用是`__sync_lock_test_and_set()`。
    这一函数接收两个参数，要写入的地址和要写入的值，返回值是写入地址原来值。
    请仔细比较amoswap指令和__sync_lock_test_and_set行为上的相似之处，
    确保你已经大致知道__sync_lock_test_and_set是如何使用amoswap实现功能。

了解了__sync_lock_test_and_set之后，我们在回到acquire的问题上来。先看xv6中是如何实现acquire的。

```c
1.    // Acquire the lock.
2.    // Loops (spins) until the lock is acquired.
3.    void
4.    acquire(struct spinlock *lk)
5.    {
6.        push_off(); // disable interrupts to avoid deadlock.
7.        if(holding(lk))
8.            panic("acquire");
9.        __sync_fetch_and_add(&(lk->n), 1);
10.        // On RISC-V, sync_lock_test_and_set turns into anatomic swap:
11.        //   a5 = 1
12.        //   s1 = &lk->locked
13.        //   amoswap.w.aq a5, a5, (s1)
14.        while(__sync_lock_test_and_set(&lk->locked, 1) != 0) {
15.            __sync_fetch_and_add(&lk->nts, 1);
16.        }
        
17.        // Tell the C compiler and the processor to not moveloads or stores
18.        // past this point, to ensure that the criticalsection's memory
19.        // references happen after the lock is acquired.
20.        __sync_synchronize();
21.        // Record info about lock acquisition for holding() anddebugging.
22.        lk->cpu = mycpu();
23.    }
```

我们需要注意的是14-16行的while循环。这里使用了`__sync_lock_test_and_set()`（使用amoswap），保证了对lk->locked读写的一致性。下面我们来分析以下具体的工流程。

- 当lk->locked==0，即当前锁空闲时，__sync_lock_test_and_set()在返回0的时，会往lk->locked写入1，即上锁。整个过程因为由amoswap实现，所以不会被其的CPU所干扰。返回0之后退出while循环，符合我们的预期。
  
- 当lk->locked==1，即当前锁被锁定时，__sync_lock_test_and_set在返回1同时，往lk->locked里写入了1。事实上在锁被锁定后我们不应该往lk->locked里写东西（当然，除了解锁的时候），但是因为锁本来就是1，再写入一个1相当于没有改变内容，所以也没差。同时，因为返回了1，我们知道当前锁被锁住了，所以会继续在while中循环。
  
- 至于第15行代码所做的事，这个和锁的功能无关，但当然它有另外的作用。这次不做解释，有兴趣的同学可以自行研究。

到这里对Lock的实现解释就结束了。在进行下一步的阅读前，强烈建议对上面的两个实现做比较和揣摩，直到你对为什么锁要使用__sync_lock_test_and_set有了清晰的认识。
    
注：以上解释可在xv6的Locking章节中找到。
    


## 2. 锁与kalloc

!!! note   "`kalloc`在什么情况下使用了锁？"
    查阅`kalloc.c`代码可知，`kalloc`只在`kalloc()`和`kfree()`中使用了锁，那这两个用锁的情况有什么共性呢？没错，他们都是把对freelist的操作锁了起来。`kfree()`在往`freelist`里加入空闲页前锁了一下，操作完之后解锁了。`kalloc()`在移除`freelist`第一个元素时也同样加了锁，操作完成再释放锁。所以对于内存分配器中需要锁保护的只有对`freelist`的操作。
      
那为什么要在操作freelist的时候上锁呢？上锁必然是为了防止并行运行的时候问题，如果我们能设想出一场景（这个场景显然是并行情况下的）让没上锁的链表作出问题，那就可以解释为什么要上锁了。

```c
// Allocate one 4096-byte page of physical memory.
// Returns a pointer that the kernel can use.
// Returns 0 if the memory cannot be allocated.
68 void *
69 kalloc(void)
70 {
71  struct run *r;
72
73  acquire(&kmem.lock);
74  r = kmem.freelist;
75  if(r)
76    kmem.freelist = r->next;
77  release(&kmem.lock);
78
79  if(r)
80    memset((char*)r, 5, PGSIZE); // fill with junk
81  return (void*)r;
82 }
```

!!! note   "某个错误场景"
    以kalloc()函数为例（见kernel/kalloc.c）。假设第73和77行的锁操作被注释掉了，那么就可能有两CPU，记为CPU0和CPU1，同时执行到第74行。然后，两个CPU就会从freelist中拿出一个内块，这就会导致两个进程共用一块内存，但对于进程来说它认为它得到的内存是独享的，这样A进程再往这块内存中写数据时，会把B之前写进去的数据破坏掉，这显然不我们所希望的。所以kalloc()里从freelist中取内存块的操作需要锁，CPU0在取的时候CPU1陷等待，直到CPU0把freelist更新完解锁后，CPU1再进去取，从而保证每个内存块只被一个进程取走。

对kfree函数，请同学们自行想象一个没有锁会出现错误的场景，在此不再做讨论。
    
## 3. 锁与bio

 在bio.c中，一共使用两种类型的锁：自旋锁`spinlock`（bcache.lock）和睡眠锁`sleeplock`（b.lock）。

### 3.1 **自旋锁**

*bcache.lock* 用于表示当前访问的bcache缓存块数据结构是否被锁住。 当bcache.lock为0时表示未锁住，能够访问当前数据结构bcache，如果为1，即暂时无法获得锁，则不断循环、自旋、等待锁重新可用。


- *bget()* 在操作bcache数据结构（修改refcnt、dev、blockno、valid）时，需要取到自旋锁 bcache.lock，操作完成后再释放该锁。


- *brelse()* 需要获取到自旋锁 bcache.lock，才能将refcnt（引用计数）减1，且有在refcnt为0时，将该数据缓存块插入到bcache.head链表后面，操作完成后再放该锁。


- *bpin()和bunpin()* 获取到锁后，才能修改refcnt，操作完成后再释放该锁。
  
###  3.2 **睡眠锁** 

b.lock用于表示bcache缓存块数据结构中的当前缓存数据块buf是否被锁住，当b.lock为1时，则调用sleep()睡眠等待锁重新可用，为0则表示锁已经被释放。睡眠锁的三个接口函数如下：

- *acquiresleep()*：查询b.lock是否被锁，如果被锁了，就睡眠，让出CPU，直到wakeup()唤醒后，获取到锁，并将b.lock置1。

- *releasesleep()*：释放锁，并调用wakeup()。

- *holdingsleep()*：返回锁的状态（1：锁住或0：未锁）。

睡眠锁的使用：  

- *bget()* 在获取到缓存块（命中的缓存块，或者，未命中时通过LRU算法替换出来缓中的缓存块）后，调用acquiresleep()获取睡眠锁。
- *bwrite()* 在写入到磁盘之前，先调用holdingsleep()查询是否已经获取到该睡锁，确保有带锁而入。
- *brelse()* 也先调用holdingsleep()查询是否已经获取到该睡眠锁，确保有带锁后才调用releasesleep()释放该锁。
  

!!! note   "自旋锁和睡眠锁的区别"
    Spinlock  
          1. 在短时间内进行轻量级加锁;  
          2. 获取过程一直进行忙循环—旋转—等待锁重新可用（占用CPU时间长）；  
          3. xv6要求在持有spinlock期间，中断不允许发生。    

    sleep-lock  
          1. 适合长时间持有；  
          2. 获取锁期间可以让出CPU；    
          3. 持有Sleep-lock期间允许中断发生，但不允许在中断程序中使用。  



## 4. 测评程序kalloctest对锁的检测

测评程序会检查锁被阻塞的情况，还记得“XV6中Lock的实现”中提到的acquire代码吗？

```c
1.    // Acquire the lock.
2.    // Loops (spins) until the lock is acquired.
3.    void
4.    acquire(struct spinlock *lk)
5.    {
6.        push_off(); // disable interrupts to avoid deadlock.
7.        if(holding(lk))
8.            panic("acquire");
9.        __sync_fetch_and_add(&(lk->n), 1);
10.        // On RISC-V, sync_lock_test_and_set turns into anatomic swap:
11.        //   a5 = 1
12.        //   s1 = &lk->locked
13.        //   amoswap.w.aq a5, a5, (s1)
14.        while(__sync_lock_test_and_set(&lk->locked, 1) != 0) {
15.            __sync_fetch_and_add(&lk->nts, 1);
16.        }
        
17.        // Tell the C compiler and the processor to not moveloads or stores
18.        // past this point, to ensure that the criticalsection's memory
19.        // references happen after the lock is acquired.
20.        __sync_synchronize();
21.        // Record info about lock acquisition for holding() anddebugging.
22.        lk->cpu = mycpu();
23.    }
```

`lk->n`：即\#acquire，获取锁的次数。

`lk->nts`：即\#fetch-and-add，没有获取到锁的次数。

每当执行一次acquire()函数，第9行`__sync_fetch_and_add(&(lk->n), 1);`就会将lk->n加1，当执行到第14行时，如果 `__sync_lock_test_and_set(&lk->locked, 1)`的返回值为1（即锁已被其他进程锁上），lk->nts的值就会加1，而测评程序会检查lk->nts的值。当lk->nts值接近0时，才能通过测试。所以这次实验中需要做的就是减少进程中的锁冲突，尽量让进程在获取锁时，锁的状态都是未被锁上的。
    

