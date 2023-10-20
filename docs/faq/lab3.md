
# Lab3常见问题

本节总结了一些目前遇到的问题供大家参考：

## 1. 问题：panic init exiting
    
内存分配出问题，导致第一个用户进程崩溃

(1)可能是kinit修改错误，导致同一个空闲物理页重复出现在freelist；

(2)也可能是没能理解kalloc的含义，返回值错误或者链表操作错误。

    

## 2. 问题：panic freeing free block"

很可能是bget函数锁的使用不正确，一定需要保证bget函数的原子性(指导书有提及)，否则同一磁盘块可能在缓存重复出现，造成二次释放。

## 3. 问题：运行时出现如下图的错误

![](lab3.assets/kerneltrap.png)

以上的意思是： **`sepc`地址对应的那条指令，由于对`stval`这个地址进行了不合法的`scause`(读/写/执行）操作，导致内核panic**

以下是这几个寄存器意义的解释

### scause

如下图所示，上图中scause为0xd即13，对应的就是`Load page fault`，即发生了不合法的读操作；
![](lab3.assets/scause.png)

### sepc

- 如果`sepc`为0x8000xxxx，则说明是内核态的指令地址，需要从`kernel/kernel.asm`反汇编代码中去查看对应的指令，如下图所示：
![](lab3.assets/kernel_asm.png)
- 如果`sepc`是类似0x000xxxx这种格式，那说明是用户态的指令地址，需要从`user`目录下对应的用户程序的反汇编代码(`xxx.asm`)中去查找

### stval

指对哪个地址发生了不合法的操作，上图示例中是对0x50这个地址发生了不合法的读操作

