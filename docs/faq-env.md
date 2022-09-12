
# 实验环境问题

## 1. xv6如何在QEMU>=6.0.0上启动？

具体解决方案详见piazza ：https://piazza.com/class/l7fs47nofoc4pm/post/8

![qemu](faq.assets/qemu.png)

Patch文件：[pmp.patch](code/pmp.patch)

该问题的解决方案来自20级某位大佬的分享，非常感谢这位大佬的贡献~~

## 2. 远程实验平台图形化无法调试？

VSCode远程调试时，提示如下错误：

![gdb1](faq.assets/gdb1.jpg)

具体解决方案详见piazza ：https://piazza.com/class/l7fs47nofoc4pm/post/20

打开xv6工作目录下的.gdbint文件，将第三行“target remote 127.0.0.1:***”用“#”注释掉。

![gdb2](faq.assets/gdb2.jpg)


## 3. 远程平台make qemu报错：Is another process using the image [fs.img]?

如图：

![qemu2](faq.assets/qemu2.png)

用ps或top命令查看一下是不是已经开启了qemu？如果qemu已经在运行，请先结束该进程。

注意：该问题出现的原因，主要是没有正常退出QEMU，请务必记得QEMU的退出方法。

!!! tip "QEMU退出方法"
    先按“Ctrl+a”组合键，接着全部松开，再按下“x”键

具体解决方案详见piazza ：https://piazza.com/class/l7fs47nofoc4pm/post/23 


## 4. 使用高版本gcc (≥12)出现报错

具体解决方案详见piazza ：https://piazza.com/class/l7fs47nofoc4pm/post/22 

![gcc](faq.assets/gcc.png)



