# 实验概述

!!! warning "温（守）馨（住）提（红）示（线）"
    本课程实验已引入代码自动查重系统，请同学们保持[学术诚信](https://integrity.mit.edu/)！

!!! note   "提示"
    本节实验与理论课的 **“导论与操作系统结构”** 和 **“进程与线程：进程”** 这两章课程内容相关，在开始实验前，请复习这两章课程内容。
    

## 1.  实验目的

本节实验的目的是对操作系统的系统调用模块进行修改，尽可能在真正修改操作系统之前，先对操作系统有一定的了解。

1. 了解xv6系统调用的工作原理。
2. 熟悉xv6通过系统调用给用户程序提供服务的机制。
3. 熟悉进程上下文切换的过程。
4. 了解进程的完整生命周期。

## 2.  实验学时

本实验为4学时。

## 3.  实验内容及要求

实验要求可以参考`MIT XV6 lab2`和`lab4`提供的部分官方说明：[[Lab: System calls(mit.edu)]](https://pdos.csail.mit.edu/6.828/2020/labs/syscallhtml) [[Lab: Trap(mit.edu)]](https://pdos.csail.mit.edu/6.S081/2020/labs/traps.html)， **但请以指导书为准，否则可能无法通过测试！** 

### 3.1 切换分支

!!! warning   "请先同步上游远程仓库，并注意切换到syscall分支进行试验"
    本次实验基于syscall分支，请同学们注意切换。

    **Step 1.** 首先，保存实验一的代码，请参考实验实用工具的[3.3.1 使用命令行完成操作](../../tools/#331)或者[3.3.2 使用VSCode内建的图形化界面完成操作](../../tools/#332-vs-code)这两小节，完成commit操作。或者， **如果你希望直接放弃掉上一次commit后的所有更改** ，那么你也可以使用-f选项强制切换分支，例如`git checkout -f syscall`。

    **Step 2.** 切换的方法可以参考实验 -> 实验实用工具 -> [3.1同步上游仓库 ~ 3.3.3 合并冲突更改](../tools.md#31)。

本次实验需要为xv6实现一些必要的系统调用和功能，在完成这些之后，你就可以正常使用`exittest`和`waittest`测试程序。具体来说，本实验有三部分：


### 3.2 任务一：进程信息收集

在该任务中，你需要在理解xv6的`exit`系统调用的基础上，实现 **进程在退出时打印自己的父进程和子进程的信息** 这一功能。

**具体要求：在exit系统调用当中寻找合适的输出时间点，在相应的函数内进行父子进程信息的打印。** 

#### 3.2.1 exit系统调用的功能

` void exit(int status)`

1. **参数`status`** ：退出状态，0表示正常退出，-1（大部分）表示异常退出。  
2. **返回值** ：无，函数调用exit之后该进程在内核态进行exit相关的资源回收之后，对应进程终止，不会返回。  
3. **功能** ：回收进程资源，回收完毕之后终止进程。  
（1）exit系统调用在处理当前进程的资源时，流程为：关闭所有打开文件 -> 将当前进程的所有子进程交给初始进程initproc -> 更改当前进程状态 -> 手动进入调度器（等待回收）。  
（2）需要完成的：当前进程的父进程的信息输出格式： `proc %d exit, parent pid %d, name %s, state %s`；当前进程的子进程的信息输出格式：`proc %d exit, child %d, pid %d, name %s, state %s`  

#### 3.2.2 运行结果

实验提供了一个exittest（见 `user/exittest.c`）用户级应用程序，该程序首先通过fork系统调用创建3个子进程，并通过sleep保证父进程在退出时子进程还没有退出，然后父进程先退出，3个子进程再退出。

!!! note   "提示"
    大家不要修改 `user/exittest.c`应用程序，它只是用于测试[3.2.1 exit系统调用的功能](#321)。

在你实现完上述功能之后，运行用户程序 exittest运行正确的情况下，你可以看到以下输出：

```shell
[cs@localhost xv6-oslab23-hitsz]$ make qemu

/* 一大波输出 …… */

xv6 kernel is booting

hart 2 starting
hart 1 starting
init: starting sh
$

/* 例子1，手动输入:trace 32 grep hello README */

$ trace 32 grep hello README
3: sys_read(3) -> 1023
3: sys_read(3) -> 966
3: sys_read(3) -> 70
3: sys_read(3) -> 0
$

/* 例子2，手动输入:trace 2147483647 grep hello README */

$ trace 2147483647 grep hello README
4: sys_trace(2147483647) -> 0
4: sys_exec(12240) -> 3
4: sys_open(12240) -> 3
4: sys_read(3) -> 1023
4: sys_read(3) -> 966
4: sys_read(3) -> 70
4: sys_read(3) -> 0
4: sys_close(3) -> 0
$

/* 例子3，手动输入:grep hello README */

$ grep hello README
$


/* 例子4，手动输入:trace 2 usertests forkforkfork */

$ trace 2 usertests forkforkfork
usertests starting
test forkforkfork: 407: syscall fork -> 408
408: sys_fork(-1) -> 409
409: sys_fork(-1) -> 410
410: sys_fork(-1) -> 411
409: sys_fork(-1) -> 412
410: sys_fork(-1) -> 413
409: sys_fork(-1) -> 414
411: sys_fork(-1) -> 415
...
$   
```

我们先不着急动手，先看看结果长什么样。

1. 在 **第一个例子** 中，`trace 32 grep hello README`，其中，trace表示我们希望执行用户态应用程序trace（见user/trace.c），后面则是trace应用程序附带的入参：

    - `32`是"1 << SYS_read"，表示只追踪系统调用read；  
    - `grep`是trace应用程序中通过"exec"启动的另一个程序（见 user/grep.    c）；  
    - `hello README`则是grep程序的入参；  
    - 该命令的作用是使用grep程序查找README文件中匹配"hello"的行，并将其所使用到的read系统调用的信息打印出来，打印的格式为：`PID: sys_read(read系统调用的arg0) -> read系统调用的return_value`。

- 在 **第二个例子** 中，trace也是启动了`grep`程序，同时追踪所有的系统调用其中`2147583647`是`31`位bit全置一的十进制整型。可以看出，打的第一条信息就是系统调用trace，其第一个参数即命令行中输入2147583647。

- 在 **第三个例子** 中，启动了`grep`程序，但是没有使用trace，所以什么trace都不会出现。

- 在 **第四个例子** 中，trace启动了`usertests`程序中`forkforkfork`（见 user/usertests.c），追踪系统调用了fork，每次fork后代都会打印对的进程id。
    - 该例中的fork实际上并没有参数，方便起见，你可以直接打印用于传该参数的寄存器的值，它可能是任意值。
    
    - forkforkfork 会一直不停的fork子进程，直到进程数超过`NPROC`，其定义见kernel/param.h。
    
    - usertests是实验提供的用于测试xv6的系统调用，详见user/usertests.c。
    
      ```c
      /* user/usertest.c */
      //Tests xv6 system calls.  usertests without arguments runs them all
      // and usertests <name> runs <name> test. The test runner creates for
      // each test a process and based on the exit status of the process,
      // the test runner reports "OK" or "FAILED".  Some tests result in
      // kernel printing usertrap messages, which can be ignored if test
      // prints "OK".
      ```
    
      

### 3.3 任务二：添加系统调用sysinfo

在该任务中，你需要加入一条新的系统调用，叫做`sysinfo`。该系统调用将收集xv6运行的一些信息。
sysinfo只需要一个参数，这个参数是结构体 `sysinfo`的指针， **这个结构体在kernel/sysinfo.h** 可以找到。xv6内核的工作就是把这个结构体填上应有的数值。下面介绍结构体每个成员的含义

```c
 1  struct sysinfo {
 2    uint64 freemem;   // amount of free memory (bytes)
 3    uint64 nproc;     // number of process
 4    uint64 freefd;    // number of free file descriptor
 5  };
```

- `freemem`：当前剩余的内存 **字节** 数
- `nproc`： **状态为UNUSED** 的进程个数
- `freefd`：当前进程可用文件描述符的数量，即 **尚未使用** 的文件描述符数量

实验提供了一个`sysinfotest`用户级应用程序（见`user/sysinfotest.c`），依次测试剩余的内存字节数、UNUSED的进程个数、未被使用的文件描述符数量。

完成任务后，你可以在xv6中运行`sysinfotest`程序，通过测试会显示如下内容：

![image-20211005130943125](part1.assets/image-20211005130943125.png)

### 3.4 测试

当完成上述的两个任务后，与Lab1一样，你也需要在在xv6-oslab23-hitsz目录下，新建time.txt文件，在该文件中写入你做完这个实验所花费的时间（估算一下就行，单位是小时），只需要写一个整数即可。

最后，在命令行输入 `make grade` 进行测试。如果通过测试，会显示如下内容：

![image-20211005131516101](part1.assets/image-20211005131516101.png)



