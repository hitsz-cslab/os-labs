# 实验平台搭建以及环境配置

&emsp;&emsp;本课程实验提供了 **远程实验平台** 供同学们做实验。当然，同学们也可以尝试自己部署实验环境，体验一下自己动手的乐趣: )

## 1. 远程实验平台

&emsp;&emsp;远程实验平台已经将XV6实验环境部署在实验中心的服务器上，我们把所有依赖的配置都已经事先搭建完毕。无论你的电脑性能如何，无论你是在宿舍、实验室还是自习室，只要你还能连上校园网，你就能完成你的实验。具体详见[远程实验环境使用指南](../remote_env/)。

!!! warning
    虽然我们已经做了一些方案保证远程环境的可靠性，但在某些特殊情况下，也不能确保不出故障，为安全起见，建议同学们将代码及时上传到git仓库或者下载到本地保存。

## 2. 自行部署的实验环境

!!! note
    我们在远程实验平台上已经部署好的实验环境，你只需要按照实验要求编写相应的代码就行。但实际上，你无法知道项目里的所有细节，难以掌握整个项目的架构。因此，我们希望同学们能够自己动手搭建实验环境，这样你得到的收获才是最大的: )

!!! tip "用于部署实验平台的实验工具"

    对于本课程的实验平台搭建，我们需要用到几个工具：

    - WSL2

    &emsp;&emsp;如果你的电脑安装的是Windows 10或Windows 11，推荐使用WSL2（Windows Subsystem for Linux 2）来搭建Linux环境。WSL2不仅能够在Windows系统中高效运行Linux内核，而且相比于WSL1具有更好的性能和完全的Linux系统调用兼容性。WSL2支持运行主流的Linux发行版（如Ubuntu、Debian等），并能够直接访问Windows的文件系统，可以通过Windows自带的文件资源管理器，像操作Windows的文件那样操作WSL上的文件，并且可以在WSL和Windows之间直接进行文件拖拽。通过WSL2可以避免使用虚拟机，占用的系统资源更少，操作也更加便捷。

    &emsp;&emsp;如何安装WSL并使用WSL进行开放？请参阅官方文档[WSL文档](https://learn.microsoft.com/zh-cn/windows/wsl/)。

    - VirtualBox虚拟机

    &emsp;&emsp;如果你的电脑装的是Windows系统，那么也可以选择能运行Linux系统的虚拟机。VirtualBox是一款开源虚拟机软件，相对VMWare等其他虚拟机来说，速度稍慢，灵活性较差，但免费！

    - Linux发行版

    &emsp;&emsp;Linux发行版是由Linux内核、GNU工具、附加软件和软件包管理器组成的操作系统。所谓“发行”，是由某些机构“发行”了Linux内核以及所有必要的软件及实用程序，使其可以作为一个操作系统使用。主流的Linux发行版有Ubuntu、Debian、Fedora、Centos等，国产Linux发行版有openEuler、Deepin等。

    - RISC-V工具链

    &emsp;&emsp;包括一系列交叉编译的工具，用于把源码编译成机器码，如gcc、binutils、glibc等。

    - QEMU模拟器

    &emsp;&emsp;qemu模拟器用于在电脑（X86）上模拟RISC-V架构的CPU。在实验中，我们将通过qemu模拟器来观察xv6的运行过程。QEMU本质上是通过高级语言来模拟仿真RISC-V处理器，对于每一个CPU核，QEMU都会运行这么一个循环：  
    &emsp;&emsp;(1)取指：读取4/8字节的RISC-V指令；  
    &emsp;&emsp;(2)译码：解析RISC-V指令，并找出对应的OP code；  
    &emsp;&emsp;(3)执行：在软件中执行相应的指令。  
    &emsp;&emsp;QEMU主循环里也需要有寄存器文件，用于维护寄存器的状态。回想我们在《计算机设计与实践》的CPU实验，是不是有似曾相识的感觉: )

!!! warning "注意事项"
    &emsp;&emsp;请确保你的计算机已经安装有ubuntu、Centos或Debian等其他Linux发行版。如果你使用的是Windows系统，推荐利用WSL2来安装并运行Linux环境，或者通过虚拟机来安装Linux。下面的XV6环境配置安装指导以Ubuntu系统为例进行详细说明。若你计划安装其他Linux系统，请参考详细的XV6环境安装指南: https://pdos.csail.mit.edu/6.S081/2020/tools.html。

### 2.1 XV6环境配置

#### 2.1.1 安装依赖包

!!! warning "注意事项"
    第一个字符“#”或"\$"不需要复制输入，只需要输入“#”或"\$"后的命令

&emsp;&emsp;安装XV6需要用到的依赖包

```shell
$ sudo apt-get install git build-essential gdb-multiarch qemu-system-misc gcc-riscv64-linux-gnu binutils-riscv64-linux-gnu
```

#### 2.1.2 安装RISC-V GNU编译器工具链

```shell
$ git clone --recursive https://github.com/riscv/riscv-gnu-toolchain
```

&emsp;&emsp;如果下载太慢，可以直接用已经下载好的包。

> 下载地址： https://mirrors.osa.moe/misc/， 选择riscv-gnu-toolchain.tar.gz。下载完成后，需要将riscv-gnu-toolchain.tar.gz上传至Linux系统中，上传方法详见MobaXterm(SSH工具)这一节中的[文件传输SFTP服务](../tools/#12-sftp)。上传到Linux系统中，需要对RISC-V GNU编译器工具链进行解压，解压命令如下：

> ```shell
> $ tar zxvf riscv-gnu-toolchain.tar.gz
> ```

&emsp;&emsp;接下来，下载编译工具链所需的依赖包：

```shell
$ sudo apt-get install autoconf automake autotools-dev curl libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev
```

&emsp;&emsp;安装RISC-V GNU编译器工具链

```shell
$ cd riscv-gnu-toolchain
$ ./configure --prefix=/usr/local
$ sudo make
$ cd ..
 
$ rm riscv-gnu-toolchain.tar.gz
rm: remove regular file 'riscv-gnu-toolchain.tar.gz'? y
$ 
```

#### 2.1.3 安装QEMU

&emsp;&emsp;下载QEMU

```shell
$ wget https://download.qemu.org/qemu-5.1.0.tar.xz
$ tar xf qemu-5.1.0.tar.xz
```

安装

```shell
$ cd qemu-5.1.0
$ ./configure --disable-kvm --disable-werror --prefix=/usr/local --target-list="riscv64-softmmu"
$ make
$ make install
$ cd ..
 
$ rm qemu-5.1.0.tar.xz
rm: remove regular file 'qemu-5.1.0.tar.xz'? y
$ 
```

检查安装是否成功

```shell
$ riscv64-unknown-elf-gcc --version
riscv64-unknown-elf-gcc (GCC) 10.2.0
...

$ qemu-system-riscv64 --version
QEMU emulator version 5.1.0
```

### 2.2 运行XV6

```shell
$ git clone https://gitee.com/ftutorials/xv6-oslab24-hitsz.git
Cloning into 'xv6-oslab24-hitsz'...
...
# 如果是初次运行git，设置你自己的gitee用户信息
$ git config --global user.email "you@example.com"
$ git config --global user.name "Your Name"
...
$ cd xv6-oslab24-hitsz
$ git checkout util
$ make qemu
# ... lots of output ...
init: starting sh
$
```

&emsp;&emsp;当可以看到“`init: starting sh`”的字样表示xv6已经正常启动，此时在“`$`”提示符后可输入xv6支持的shell命令。

!!! tip "请记住：QEMU退出方法"
    先按“`Ctrl+a`”组合键，接着松开所有按键。再按下“`x`”键。

    &emsp;&emsp;（1）以下是命令解释：
    “`Ctrl+a`”组合键，用于激活QEMU的全局命令模式。松开所有按键。然后，按下x键，这样将直接终止QEMU进程，返回到你启动QEMU的Shell或命令行界面。

    &emsp;&emsp;（2）如果在退出QEMU时遇到任何问题（如QEMU进程无法终止），你可能需要使用kill命令来强制结束QEMU进程。
    
    &emsp;&emsp;如果是在远程实验平台，你需要重新开启一个Linux Terminal命令行终端界面，输入如下命令，把与自己有关的进程全kill：

    ```shell
    $ killall -u id
    ```

    &emsp;&emsp;`killall -u id`中的`id`应该替换为`你的学号`。

    &emsp;&emsp;如果是在你自己安装的Linux环境，请你自行搜索如何安全地结束进程。


&emsp;&emsp;至此，XV6已经能够正常运行了: )



## 3. Docker

Docker 是一个开源的平台，旨在简化应用程序的开发、测试和部署。它使用容器化技术，将应用程序及其依赖环境打包到一个标准化的单元（称为容器）中，从而确保应用程序可以在任何地方一致运行，无论是开发环境、测试环境还是生产环境。

### Docker安装

参考官网[安装指南](https://docs.docker.com/get-started/get-docker/)

### Docker的关键概念

- 镜像 (Image): 镜像是Docker容器的只读模板。它包含应用程序所需的所有文件、依赖库、配置等。开发者可以基于现有的镜像创建新的镜像。
- 容器 (Container): 容器是从镜像创建的运行实例。每个容器都是独立的，彼此之间隔离，可以拥有自己的文件系统、进程、网络等。容器轻量且启动迅速。
- Dockerfile: Dockerfile 是一个包含一系列指令的文本文件，定义了如何构建一个镜像。通过Dockerfile，开发者可以描述应用程序及其依赖项的构建步骤。
- Docker Hub: 这是一个在线仓库，用于存储和共享Docker镜像。开发者可以从Docker Hub下载公共镜像，也可以上传自己创建的镜像。

### Docker的优点

- 跨平台兼容: 容器中的应用程序不依赖于主机的操作系统环境，可以跨平台运行。
- 高效资源利用: 容器比虚拟机更轻量，启动速度更快，能够高效利用系统资源。
- 易于管理和部署: Docker可以将应用程序及其依赖打包在一起，简化了部署过程，减少了环境配置问题。

### 通过Docker启动xv6

我们在 xv6-oslab24-hitsz 目录下配好了 Dockerfile，只需要在 Linux 命令行中进入该目录：

1. 使用 `su` 命令进入 root 用户
2. 使用 `make build_docker` 构建 docker 镜像（仅需构建一次）。
3. 使用 `make docker` 启动容器进入 bash 终端。
4. 使用 `make qemu` 命令启动 xv6。

![docker-run-qemu](./env.assets/docker-run-qemu.png)
