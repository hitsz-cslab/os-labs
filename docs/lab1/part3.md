# 实验步骤

下面，我们就正式进入到各实验的细节中，并给出实验实现的一些提示。大家需要依照实验指导书的[实验概述](../part1/)中提到的要求进行实现。同时，还需要回答指导书给出的[问题](../part3/#33)。

### 1. 部署实验环境

实验环境主要分为三部分：xv6运行环境、xv6源码、xv6的编译与运行。

#### 1.1  xv6运行环境

xv6运行环境详见[实验平台及环境配置](../../env/)。

#### 1.2  xv6源码

请clone最新代码到本地进行实验：   

```
$ git clone https://gitee.com/greenhandzpx/xv6-oslab23-hitsz.git
```

每个实验项目都在不同的分支上完成，请 **注意切换分支** ，例如，实验一需切换到util分支后进行开发。

```shell
$ git branch -a
$ git checkout util
```

<div align="center"> <img src="../part3.assets/image-20210913105916234.png" /> </div>

 xv6的代码结构：

<div align="center"> <img src="../part3.assets/image-20201017231446472.png" /> </div>

#### 1.3  编译并运行xv6

Step1 在代码总目录xv6-oslab23-hitsz下输入`make qemu`， 编译并运行xv6;

Step2 当可以看到“init: starting sh”的字样表示xv6已经正常启动，此时在“$”提示符后可输入xv6支持的shell命令。

<div align="center"> <img src="../part3.assets/image-20210913110108193.png" /> </div>

!!! warning "qemu退出方法"
    先按`ctrl+a`组合键，接着完全松开, 再按`x`



### 2.  准备工作

本次实验需要编写[实验内容](../part1/#3)中介绍的3个Unix实用程序。初次接触操作系统实验的你可能会感到不知所措，因此不妨先体验一下这些程序的运行效果。实际上，Linux中具备本次实验要实现的一些程序，例如sleep、find、xargs。你可以先尝试在Linux中使用这些命令，充分体会功能后再开始编程。当然，Linux中命令的功能较为复杂，我们仅要求实现简化版。

实验开始之前，我们 **强烈建议** 你先完成以下工作：

!!! tip ""
    1. 熟悉常见命令的使用，如 `echo`、`xargs`、`find`。
    - 了解目录的使用。了解 `.`、`..`、`/`分别表示什么，熟悉常见的目录操作命令，如`mkdir`、`cd`。
    - 了解重定向的使用，重定向即命令中的`<`和`>`，用于修改右侧命令的标准输入/输出。例如`echo Hello world > file_a`会将字符串`Hello world`输出至文件`file_a`，而不是打印在终端。
    - 了解管道的使用。管道即命令中的`|`，用于将左侧命令的标准输出传递给右侧命令的标准输入。
    - 了解常见系统调用的使用。如 `fork`、`exit`、`wait`、`open`、`close`、`read`/`write`、`pipe`、`dup`。



### 3.  编写用户程序

#### 3.1  代码示例

sleep程序已经实现，你需要理解其代码并成功将其运行：

Step1. 阅读user/sleep.c文件，理解代码和注释；

<div align="center"> <img src="../part3.assets/image-20210913110416368.png" /> </div>

Step2. 由于sleep.c为新增的用户程序文件，请在Makefile文件中找到UPROGS，在UPROGS上增加一行`$U/_sleep`：

<div align="center"> <img src="../part3.assets/image-20220926104856325.png" /> </div>

Step3. 编译xv6并运行sleep。

<div align="center"> <img src="../part3.assets/image-20201018125249828.png"  width = 40%/> </div>

Step4.回答3.3中的[相关问题](#33)。

#### 3.2  实验提示

##### 3.2.1   pingpong

a)   使用`pipe()`创建管道，详见[实验原理](../part2/)；

b)   使用`fork()`创建子进程，注意根据返回值，判断父子进程；

c)    利用`read()`, `write()`函数对管道进行读写。

d)   请在`user/pingpong.c`中实现。

e)   修改`Makefile`，将程序添加到`UPROGS`。

##### 3.2.2  find

a)    可查看`user/ls.c`以了解如何读取目录；

b)    可参照`user/ls.c`的逻辑实现；

c)    使用递归允许`find`进入到子目录；

d)    不要递归进入`.`和`..`；

e)    测试时需要创建新的文件和文件夹，可使用`make clean`清理文件系统，并使用`make qemu`再编译运行。


**提示：**

- 关于以上2个Unix实用程序的实现亦可参考MIT官方的实验指导完成实验[Lab: Xv6 and Unix utilities](https://pdos.csail.mit.edu/6.828/2020/labs/util.html)

- 系统调用接口的示例可查阅《xv6 book》chapter 1的内容， **尤其是1.2、1.3节的内容** 。

#### 3.3 回答问题

一、阅读sleep.c，回答下列问题

1) 当用户在xv6的shell中，输入了命令`sleep hello world\n`，请问argc的值是多少，argv数组大小是多少。  
   
2) 请描述main函数参数argv中的指针指向了哪些字符串，他们的含义是什么。
   
3) 哪些代码调用了系统调用为程序sleep提供了服务？
   

二、了解管道模型，回答下列问题

1) 简要说明你是怎么创建管道的，又是怎么使用管道传输数据的。  
   
2) fork之后，我们怎么用管道在父子进程传输数据？ 
   
3) 试解释，为什么要提前关闭管道中不使用的一端？（提示：结合管道的阻塞机制）

### 4.  xv6启动流程分析

当我们输入`make qemu`后，就能启动qemu，从控制台参数加载资源，将xv6整体加载到物理内存中。xv6启动涉及kernel目录下的entry.S、start.c与main.c等少数几个文件，entry、start两个函数运行在机器态（类似于BIOS），main运行在内核态（特权级），后面的用户程序运行在用户态。

启动流程步骤如下:

（1）QEMU将pc设为0x8000_0000，开始运行xv6的首个函数：_entry (kernel/entry.S)。

（2）entry函数设置当前CPU的栈寄存器(sp)，拥有栈后就可以调用函数了，进入start。

（3）start函数进行了如下操作：

- 初始化机器态特权寄存器
- 启用时钟中断
- 把当前CPU的id放入tp寄存器

（4）start函数完成了机器态初始化，随后就进入了内核态的首个函数：main。

（5）main函数初始化了各种外设的驱动、内存模块、进程模块、文件系统。

（6）在main函数调用的userinit()里面，xv6在内核加载了第一个用户程序initcode，只有不到100字节。这个程序运行时只占用一个内存页（数据来自initcode），这个页兼具代码段、数据段、栈的功能。

（7）从main函数进入scheduler函数（调度器）之后，scheduler调度到初始进程initcode，它只做一件事：运行exec(“/init”)，然后进入第一个进程。这样，xv6已经可以正常运行用户程序了，只要用户进程存在就会被CPU调度运行。

init程序被放在磁盘上，它就是所有进程的祖先（0号进程）。

- 从磁盘加载应用程序的过程是exec系统调用完成的
- 它通过fork+exec的方式启动控制台程序sh
- 它不断地调用wait(0)来回收所有僵尸进程
- 它被启动后会输出一行“init: starting sh”
- sh程序是控制台进程。它被启动后就会输出'$'

有兴趣的同学可以用GDB在`_entry`函数处打断点，调试跟踪xv6的启动（详见[VSCode图形化调试指南](../../remote_env_gdb)）

<div align="center"> <img src="../part3.assets/image-20230914120057118.png" width = 60%/> </div>

