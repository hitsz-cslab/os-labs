# 实验实现
!!! note "提示"
    这里，我们将对一些实现的细节做一定提醒，但是完整的流程需要自己把握，我们 **不会给出每一步实现步骤** 。因此，最重要的还是阅读[实验原理](../part2)内容，得出自己的解决方案，然后参考"实验实现"这部分的内容。

    值得注意的是，xv6的代码很值得参考，可以看看其他系统调用和功能是怎么实现的。同时，还可以使用 `printf` 或者 `gdb` 等方法查看具体的执行情况。此外，还可以阅读 xv6 指导书，看看背后的设计机理。
    
    如果在实现过程中遇到困难，可以 **先尝试回答任务三** 提出的问题，它们或许可以帮助你更好地理解。

## 1. 任务一：系统调用信息打印

**具体要求：添加一个名为 *trace* 的系统调用。**

### 1.1 流程

在用户部分：

- 参考[这个指南](../../tools/#31)，切换到syscall分支并同步上游库；
- 记得在Makefile中给 `UPROGS` 加 `$U/_trace`。
- 然后 `make qemu` 会发现无法编译 user/trace.c，这是因为我们没在用户态包装好 `trace()` ，因此我们需要按照 ”实验原理“ 中所述，慢慢添加一个系统调用的接口。
  
    - 在 `user/user.h` 加入函数定义；
    - 在 `user/usys.pl` 加入用户系统调用名称（也就是之前所说的entry）；
    - 在 `kernel/syscall.h` 加入系统调用号SYS_trace，以给trace做一个标识，该调用号的取值可自行决定。
    
- 然后启动xv6，在shell输入 `trace 32 grep hello README` ，会发xv6崩溃了，因为这条系统调用没有在内核中实现。

在内核部分：

- 在 `kernel/sysproc.c` 中添加 `sys_trace()`。
- 在 `kernel/syscall.c` 中加入对应的系统调用分发逻辑。
- 开始实现 `sys_trace()` 对应的逻辑，同时该系统调用还需要修改其他数的逻辑。

### 1.2 实现提示

- 进程启动`trace`后，如果fork，子进程也应该开启trace，并且继承进程的`mask`，也就是trace系统调用的入参。（这需要注意修改`kernel/proc.c`中fork()的代码）

- 可以在PCB （`struct proc`）中添加成员 `int mask`，这样我们可记住trace告知进程的mask。PCB定义于`kernel/proc.h`。

- 为了让每个系统调用都可以输出信息，我们应该在 `kernel/syscall.c` 的 `syscall()` 添加相应逻辑。（添加一个系统调用对应名字的数组，好像以大大简化实现噢！）

- 其他的系统调用实现可以参考，详见`kernel/sysproc.c`。

    

## 2. 任务二：添加系统调用sysinfo

**具体要求：添加一个名为 *sysinfo* 的系统调用。**

### 2.1 流程

在用户部分：

- 切至 `syscall` git分支
- 记得在`Makefile`中给 `UPROGS` 加 `$U/_sysinfotest`
- 然后 `make qemu` 会发现无法编译 `user/sysinfotest.c` ，问题和上面一样，一顿操作猛如虎。
  
  ```c
  /* 你需要在user/user.h添加如下定义 */
  struct sysinfo;  // 需要预先声明结构体，参考fstat的参数stat
  int sysinfo(struct sysinfo *);
  ```

在内核部分：

- 请参考之前的流程自行完成设计。
  
### 2.2 实现提示

- `sysinfo` 需要在内核地址空间中填写结构体，然后将其复制到用户地址间。可以参考`kernel/file.c fstat()` 以及 `kernel/sysfile.csys_fstat()` 中通过 `copyout()` 函数对该过程的实现。

- 计算剩余的内存空间的函数代码，最好写在文件 `kernel/kalloc.c` 里。

- 计算空闲进程数量的函数代码，最好写在文件`kernel/proc.c` 里。

- 计算可用文件描述符数量的代码，最好写在文件`kernel/proc.c` 里。

  - 文件描述符的值实际上就是`PCB`成员`ofile`的下标（见`kernel/proc.h`）。

- 在添加上述两个函数后，可以在`kernel/defs.h`中声明，以便在其他文中调用这些函数。

- 查阅《xv6 book》`chapter1`和`chapter2`中相关的内容。

​      

## 3. 任务三：回答问题

!!! question  "回答问题"
    

    (1) 阅读`kernel/syscall.c`，试解释函数 `syscall()` 如何根据系统调用号调用对应的系统调用处理函数（例如`sys_fork`）？`syscall()` 将具体系统调用的返回值存放在哪里？
    
    (2) 阅读`kernel/syscall.c`，哪些函数用于传递系统调用参数？试解释argraw()函数的含义。
    
    (3) 阅读`kernel/proc.c`和`proc.h`，进程控制块存储在哪个数组中？进程控制块中哪个成员指示了进程的状态？一共有哪些状态？
    
    (4) 阅读`kernel/kalloc.c`，哪个结构体中的哪个成员可以指示空闲的内存页？Xv6中的一个页有多少字节？
    
    (5) 阅读`kernel/vm.c`，试解释`copyout()`函数各个参数的含义。


​      