# 常见问题

## 1. 如何新开一个命令行

按下`F5`（或`Fn+F5`）后，文件系统运行为 **前台模式** 。想要输入命令，如执行`ls`，则需要新开一个命令行。可以通过`ctrl + shift +｀`呼出一个新的命令行，然后在命令行操作。

![image-20211024170839872](part3.assets/image-20211024170839872.png)

另外，如下图所示，可以按这个 **拆分键** ，就可以同时显示两个终端了。

![image-20211024170743344](part3.assets/image-20211024170743344.png)

## 2. 挂载点没连接上

提示"fuse: bad mount point ... Transports end point is not connected" 如下图所示：

![image-20211024171103128](part3.assets/image-20211024171103128.png)

这种情况就是不正常的卸载文件系统，同学们直接`ctrl + c`或者在VSCode直接关掉程序，导致上次的挂载点仍被占用。我们需要完成 **文件系统的卸载** ，释放上次占用的挂载点：

```sh
fusermount -u ./tests/mnt
```

## 3. 挂载点不空

提示"mountpoint is not empty" 如下图所示：

![image-20211024171348041](part3.assets/image-20211024171348041.png)

这种情况就是在测试的时候，不小心给`mnt`文件夹下创建了一个文件，导致`mnt`目录不是空的，把`mnt`下的 **文件删除** 即可，保证`mnt`目录在挂载时是空的 。

## 4.挂载点不存在

提示"fuse: bad mount point ... No such file or directory" 如下图所示：

![](part5.assets/没有挂载点.png)

这种情况是，文件系统挂载的挂载点，如`./tests/mnt`不存在。需要同学们提前创建好一个空的文件夹`./tests/mnt`。

## 5.挂载点忙碌

同学们在执行任务一的测评脚本时，如果遇到`fusermount: failed to umount xxx : Device or resource busy`的问题，请使用`git pull`命令更新实验包的任务一测评程序即可。

## 6.S_IFxxx标红

![](part5.assets/red.png)

这个问题包括任务一和任务二，上图以`simplefs`为例展示。

这个标红不会影响同学们项目的编译（`F5`或`Fn + F5`），可以选择直接无视。如果你项目编译失败，不是因为这个问题。

当然如果看着不习惯，想取消掉标红，可以在这个文件的 **第一行** （注意是第一行），添加以下这个宏定义。

```c
#define _XOPEN_SOURCE 700
```

[一个帖子](https://cloud.tencent.com/developer/ask/sof/107977855?from=16139)给出的参考解释：

![](part5.assets/red_why.png)
