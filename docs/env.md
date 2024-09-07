# 实验平台搭建以及环境配置

本课程实验提供了 **远程实验平台** 供同学们做实验。当然，同学们也可以尝试自己部署实验环境，体验一下自己动手的乐趣: )

## 1. 远程实验平台

远程实验平台已经将XV6实验环境部署在实验中心的服务器上，我们把所有依赖的配置都已经事先搭建完毕。无论你的电脑性能如何，无论你是在宿舍、实验室还是自习室，只要你还能连上校园网，你就能完成你的实验。具体详见[远程实验环境使用指南](../remote_env/)。

!!! warning
    虽然我们已经做了一些方案保证远程环境的可靠性，但在某些特殊情况下，也不能确保不出故障，为安全起见，建议同学们将代码及时上传到git仓库或者下载到本地保存。

## 2. 自行部署的实验环境

!!! note
    我们在远程实验平台上已经部署好的实验环境，你只需要按照实验要求编写相应的代码就行。但实际上，你无法知道项目里的所有细节，难以掌握整个项目的架构。因此，我们希望同学们能够自己动手搭建实验环境，这样你得到的收获才是最大的: )

!!! tip "用于部署实验平台的实验工具"

    对于本课程的实验平台搭建，我们需要用到几个工具：

    - WSL2

    如果你的电脑安装的是Windows 10或Windows 11，推荐使用WSL2（Windows Subsystem for Linux 2）来搭建Linux环境。WSL2不仅能够在Windows系统中高效运行Linux内核，而且相比于WSL1具有更好的性能和完全的Linux系统调用兼容性。WSL2支持运行主流的Linux发行版（如Ubuntu、Debian等），并能够直接访问Windows的文件系统，可以通过Windows自带的文件资源管理器，像操作Windows的文件那样操作WSL上的文件，并且可以在WSL和Windows之间直接进行文件拖拽。通过WSL2可以避免使用虚拟机，占用的系统资源更少，操作也更加便捷。

    如何安装WSL并使用WSL进行开放？请参阅官方文档[WSL文档](https://learn.microsoft.com/zh-cn/windows/wsl/)

    - VirtualBox虚拟机

    如果你的电脑装的是Windows系统，那么必须要装一个能运行Linux系统的虚拟机。VirtualBox是一款开源虚拟机软件，相对VMWare等其他虚拟机来说，速度稍慢，灵活性较差，但免费！

    - Linux发行版

    Linux发行版是由Linux内核、GNU工具、附加软件和软件包管理器组成的操作系统。所谓“发行”，是由某些机构“发行”了Linux内核以及所有必要的软件及实用程序，使其可以作为一个操作系统使用。主流的Linux发行版有Ubuntu、Debian、Fedora、Centos等，国产Linux发行版有openEuler、Deepin等，为支持国产Linux发行版，我们建议同学们使用华为公司发行的 openEuler 作为实验平台，以此揭开openEuler的神秘面纱: )

    - RISC-V工具链

    包括一系列交叉编译的工具，用于把源码编译成机器码，如gcc、binutils、glibc等。

    - QEMU模拟器

    qemu模拟器用于在电脑（X86）上模拟RISC-V架构的CPU。在实验中，我们将通过qemu模拟器来观察xv6的运行过程。QEMU本质上是通过高级语言来模拟仿真RISC-V处理器，对于每一个CPU核，QEMU都会运行这么一个循环：
    (1)取指：读取4/8字节的RISC-V指令；
    (2)译码：解析RISC-V指令，并找出对应的OP code；
    (3)执行：在软件中执行相应的指令。
    QEMU主循环里也需要有寄存器文件，用于维护寄存器的状态。回想我们在《计算机设计与实践》的CPU实验，是不是有似曾相识的感觉: )

!!! warning "注意事项"
    请确保你已经安装有ubuntu、Centos或Debian等其他Linux系统，下面的安装步骤以Ubuntu系统为例，如果是其他Linux操作系统，其具体安装步骤请参考:https://pdos.csail.mit.edu/6.S081/2020/tools.html

### 2.1 XV6环境配置

#### 2.1.1 安装依赖包

!!! warning "注意事项"
    第一个字符“#”或"\$"不需要输入，只需要输入“#”或"\$"后的字符串

安装XV6需要用到的依赖包

```console
$ sudo apt-get install git build-essential gdb-multiarch qemu-system-misc gcc-riscv64-linux-gnu binutils-riscv64-linux-gnu
```

#### 2.1.2 安装RISC-V GNU编译器工具链

```console
$ git clone --recursive https://github.com/riscv/riscv-gnu-toolchain
```

如果下载太慢，可以直接用已经下载好的包。

> 下载地址： https://mirrors.osa.moe/misc/， 选择riscv-gnu-toolchain.tar.gz。下载完成后，需要将riscv-gnu-toolchain.tar.gz上传至Linux系统中，上传方法详见MobaXterm(SSH工具)这一节中的[文件传输SFTP服务](../tools/#12-sftp)。上传到Linux系统中，需要对RISC-V GNU编译器工具链进行解压，解压命令如下：

> ```console
> $ tar zxvf riscv-gnu-toolchain.tar.gz
> ```

接下来，下载编译工具链所需的依赖包：

```console
$ sudo apt-get install autoconf automake autotools-dev curl libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev
```

安装RISC-V GNU编译器工具链

```console
$ cd riscv-gnu-toolchain
$ ./configure --prefix=/usr/local
$ sudo make
$ cd ..
 
$ rm riscv-gnu-toolchain.tar.gz
rm: remove regular file 'riscv-gnu-toolchain.tar.gz'? y
$ 
```

#### 2.1.3 安装QEMU

下载QEMU

```console
$ wget https://download.qemu.org/qemu-5.1.0.tar.xz
$ tar xf qemu-5.1.0.tar.xz
```

安装

```console
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

```console
$ riscv64-unknown-elf-gcc --version
riscv64-unknown-elf-gcc (GCC) 10.2.0
...

$ qemu-system-riscv64 --version
QEMU emulator version 5.1.0
```

### 2.2 运行XV6

```console
$ git clone https://gitee.com/greenhandzpx/xv6-oslab23-hitsz.git
Cloning into 'xv6-oslab23-hitsz'...
...
# 如果是初次运行git，设置你自己的gitee用户信息
$ git config --global user.email "you@example.com"
$ git config --global user.name "Your Name"
...
$ cd xv6-oslab23-hitsz
$ git checkout util
$ make qemu
# ... lots of output ...
init: starting sh
$
```

当可以看到“init: starting sh”的字样表示xv6已经正常启动，此时在“$”提示符后可输入xv6支持的shell命令。

!!! tip "QEMU退出方法"
    先按“Ctrl+a”组合键，接着全部松开，再按下“x”键

至此，XV6已经能够正常运行了: )
