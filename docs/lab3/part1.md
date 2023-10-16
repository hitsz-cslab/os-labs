# 实验概述

!!! warning "温（守）馨（住）提（红）示（线）"
    
    本课程实验已引入代码自动查重系统，请同学们保持[学术诚信](https://integrity.mit.edu/)！
    

!!! tip "关于第二次实验课考核"
    我们将在第四次实验课进行第二次实验现场验收：  

    - **验收内容** ：前三次实验
        - Lab1：XV6与Unix实用程序
        - Lab2：进程与系统调用
        - Lab3：锁机制的应用  
      
    - **验收目的** ：考察同学们是否是自己完成的实验代码，而不是“复制拷贝”。  
      
    - **验收形式** ：助教与同学们进行一对一问答  
      
    请同学们认真对待！
    

!!! note   "课程复习和预习要求"
    本节实验与理论课的 **“多线程”** 、  **“信号量”**   和 **“死锁”** 这三章课程内容相关，请同学们复习这三章课程内容。

    在做实验之前，请同学们阅读[xv6 book](https://pdos.csail.mit.edu/6.828/2020/xv6/book-riscv-rev1.pdf)的以下章节及相关源代码：
    
    - [1] xv6 book, Chapter 6 Locking (锁)
    - [2] xv6 book, 3.5 Code: Physical memory allocator（物理内存分配器）
    - [3] xv6 book, Chapter 8 File system：8.1 Overview ~ 8.3 Code: Buffer cache（磁盘缓存）

!!! info   "实验背景"
    锁起到了保护作用，让一些关键的数据结构在并行环境下仍能保持一致性。
    

    在这个实验中，我们需要为多CPU核的xv6内存分配器和磁盘缓存（buffer cache）重新设计代码以提高并行性。这两个实验分别用到了两种 **不同** 的设计方法，来适用自身的结构，大大减少锁争用（在保证功能正确的前提下）。


## 1.  实验目的

   1. 了解Lock的实现原理，理解CPU为Lock的实现提供的支持。  
   2. 理解xv6的内存分配器kalloc以及磁盘缓存buffer cache的工作原理，掌握Lock在xv6中内存分配器/磁盘缓存中的作用。  
   3. 掌握锁竞争对程序并行性的影响。  
   4. 学习减少锁竞争的方法。
    

## 2.  实验学时

本实验为4学时。
    

## 3.  实验内容及要求

!!! warning   "请先同步上游远程仓库，并注意切换到lock分支进行试验"
 
    本次实验基于 **lock** 分支，请同学们注意切换。

    **Step 1.** 首先，保存实验二的代码，请参考实验实用工具的[3.3.1 使用命令行完成操作](../../tools/#331)或者[3.3.2 使用VSCode内建的图形化界面完成操作](../../tools/#332-vs-code)这两小节，完成commit操作。或者， **如果你希望直接放弃掉上一次commit后的所有更改** ，那么你也可以使用-f选项强制切换分支，例如`git checkout -f lock`。

    **Step 2.** 切换的方法可以参考实验 -> 实验实用工具 -> [3.1同步上游仓库 ~ 3.3.3 合并冲突更改](../tools.md#31)。


### 3.1 任务一：内存分配器（Memory allocator）

**修改内存分配器（主要修改kernel/kalloc.c）** ，使每个CPU核使用独立的内存链表，而不是现在的共享链表。

XV6的内存分配器只有一个内存链表供多个CPU核使用。在使用`kalloc()`获取内存时，由于添加了内存锁`kmem.lock`，其他CPU如果要切换进行内存申请必须等待当前进程释放内存锁。要消除锁争用的情况，需要重新设计内存链表管理机制以避免单个锁和单个链表。实验基本任务是让每个CPU拥有自己的内存链表，每个链表都有自己的锁。其中最具挑战的就是，当一个CPU内存链表不足时，还可以从其他链表 **窃取** 内存块，这样，就不会让所有的CPU争抢一个空闲区域（窃取可能会引发锁争用，但这也是不可避免的情况）。

!!! note   "怎么查看XV6开启了多少个CPU核？"
    在输入make qemu时，默认会启动3个CPU核，具体可以查看Makefile文件：

    ```makefile
    264 ifndef CPUS
    265 CPUS := 3
    266 endif
    ```

    如果要开启单个CPU，使用命令 make CPUS=1 qemu 重新编译即可。在C代码中，可以使用kernel/param.h中的NCPU宏来查看CPU的数目（maximum number of CPUs）。

#### 3.1.1 关于测评程序kalloctest

测评程序`kalloctest`（见user/kalloctest.c）有两个test，分别是test1和test2。

- (1) 在test1中fork出多个进程，它们并发运行于不同CPU核心，用 `sbrk()` 进行频繁获取空闲内存（通过调用`kalloc()`实现），然后又释放（通过调用`kfree()`实现）。在获取内存的时候不同CPU核心可能会争抢锁。

<!--
!!! note   "`sbrk()`函数功能说明"
    sbrk()是xv6提供的系统调用接口，其内核文件位置在kernel/sysproc.c。作用：为进程分配或回收内存。它有一个参数，代表要分配的字节数。它返回的是新分配内存的地址。
    
    这个系统调用是通过`growproc`来实现的。如果给的参数是正数，`growproc`使用`uvmalloc`来分配内存；如果给的参数是负数，使用`uvmdealloc`来释放内存。
    
    `uvmalloc`首先使用`kalloc`来分配物理内存，然后再用`mappages`把PTE加到用户的页表里。`uvmdealloc`调用`uvmunmap`实现其功能，`uvmunmap`首先用`walk`来找到对应的PTE，然后使用`kfree`来释放相应的物理内存。
-->

- (2) 在test2中测试所有空闲内存的获取和释放，由于test2未使用fork，只在单进程中测试。
   
测评程序kalloctest最终会打印出`fetch-and-add`的数目，也就是 **没有成功获取** 到kmem锁的次数，这种粗略测量锁争用的方式（实验前，即未修改前）如下：

```console
$ kalloctest
start test1
test1 results:
--- lock kmem/bcache stats
lock: kmem: #fetch-and-add 46375 #acquire() 433016
lock: bcache: #fetch-and-add 0 #acquire() 1242
--- top 5 contended locks:
lock: kmem: #fetch-and-add 46375 #acquire() 433016
lock: proc: #fetch-and-add 21429 #acquire() 170067
lock: virtio_disk: #fetch-and-add 13306 #acquire() 114
lock: proc: #fetch-and-add 6023 #acquire() 170143
lock: proc: #fetch-and-add 3661 #acquire() 170147
tot= 46375
test1 FAIL
start test2
total free number of pages: 32499 (out of 32768)
.....
test2 OK
```

其中，"lock: kmem"、"lock: bcache"、"lock: proc" 等指示了锁的类型，acquire()记录了每种锁 **被请求** 的次数，fetch-and-add就是 **进程没有成功获取到此锁** 的次数。例如， *lock: kmem: #fetch-and-add 46375 #acquire() 433016* 这句话的意思是，kmem.lock请求了433016次，仅成功了433016 - 46375 = 386641次，这就是发生了锁争用了（lock contention）。

这些计数来自于在kalloctest调用的ntas()，它通过执行statistics()函数，打开"statistics"设备文件，获取内核打印的那些针对kmem和bcache锁( **本验的重点** )及5个竞争最激烈的锁的计数（见kernel/spinlock.c中的`statslock()`）。关于#acquire和#fetch-and-add的计数算法见[测评程序kalloctest对锁的检测](../part4/#4-kalloctest)。

如果存在锁争用，fetch-and-add的数量将会很高，因为在acquire()获得锁之前要进行许多循环迭代，"statistics"设备文件会返回kmem和bcache锁的#test-and-set的总和（也就是`tot`的值，即没有获取到此锁的次数）。

#### 3.1.2 具体要求
  
1)   请使用`initlock()`初始化锁，并要求锁名字以`kmem`开头；  
2)   运行`kalloctest`查看实现的代码是否减少了锁争用（`tot`没有获取到此锁的次数小于10则为通过）；  
3)   运行 `usertests sbrkmuch` 以测试修改代码后系统是否仍可以分配所有的内存；  
4)   运行`usertests`，确保其能能够全部通过；  
5)   kalloctest和usertests的输出如下图（锁争用的次数大大减少），具体的数据会有所差别：

    
<div align="center"> <img src="../part1.assets/kalloc_proc.jpg" width = 70%/> </div>

### 3.2 任务二：磁盘缓存（Buffer cache）
    
在访问文件数据的时候，操作系统会将文件的数据放置在磁盘缓存中。磁盘缓存是不同进程之间的共享资源，因此需要通过锁确保使用的正确性。如果有多进程密集地使用文件系统，他们可能会竞争磁盘缓存的`bcache.lock`锁。

目前，xv6采用单个锁来管理磁盘缓存。假设有三个进程大量读写磁盘，而由于磁盘缓存只有一个`bcache.lock`锁，这就导致三个进程的竞争异常激烈。因此，我们需要 **修改磁盘缓存块列表的管理机制（主要修改kernel/bio.c）** ，使得可用多个锁管理缓存块，从而减少缓存块管理的锁争用。

<!--
!!! tip   "提示"
    在任务一中我们可以为每个CPU分配属于自己的内存链表，但在任务二中不能为每个CPU分配属于自己的磁盘缓存区。主要原因是多个进程间（或者说多个CPU间）访问磁盘同一块区域时会访问同一块缓存数据块（也就是缓存数据块是共享的），此外，一个磁盘缓存本身比较大，为每个CPU核都分配一个磁盘缓存显然会造成空间浪费。
-->

#### 3.2.1 关于测评程序bcachetest

测评程序bcachetest（见user/bcachetest.c）通过创建多个进程重复去读取不同的文件，从而造成bcache.lock锁争用。

测试方式如下：

实验前（即未修改前），测评程序bcachetest的输出如下：

<div align="center"> <img src="../part1.assets/image-20211019205157241.png" width = 80%/> </div>

可以看到，bcache的`fetch-and-add`值非常大，说明磁盘缓存锁的争用非常严重。

修改buffer cache的设计后，所有锁的fetch-and-add的数目应当接近于0。

#### 3.2.2 具体要求

1)   理想状态下，`bcachetest`中数据块缓存相关的所有锁`fetch-and-add`的总和应该为0，但本实验中总和`tot`不超过500即可；  
2)   请修改`bget()`和`brelse()`，使得缓存区并发的查询和释放不容易发生锁争用，比如，不是所有流程都得等bcache.lock；  
3)   同样要求`usertests`中的用例全部通过，最后的输出如下（具体数据有所出入）：  

<div align="center"> <img src="../part1.assets/image-20211019205449169.png" width = 70%/> </div>

### 3.3 测试

当完成上述的两个实验后，在命令行输入 `make grade` 进行最终测试。


<div align="center"> <img src="../part1.assets/grade-16349752282893.png" /> </div>