
# Lab1常见问题

## 1. sleep

### 1.1 执行sleep报错

检查一下是否修改Makefile

## 2. pingpong

### 2.1 检验pingpong时出现如下报错

![pingpong1](lab1.assets/pingpong1.png)

解决方案：

![image-20220926180507976](lab1.assets/image-20220926180507976.png)

### 2.2 pingpong出现usertrap()报错

1. 数组越界、内存溢出（scause=13）
![pingpong2](lab1.assets/pingpong2.png)
解决方案：
![image-20220926180632009](lab1.assets/image-20220926180632009.png)
2. 用户程序退出时使用return（scause=2）
![pingpong3](lab1.assets/pingpong3.png)
解决方案：用户程序退出时使用 `exit()` 函数退出，不要使用 `return` 返回。

## 3. primes

### 3.1 primes输出发生奇怪的错误

不知道为什么第二个输出带着奇怪的$符号

![primes](lab1.assets/primes.png)

解决方案：

![image-20220926180747910](lab1.assets/image-20220926180747910.png)

### 3.2 prime 内核中输出正确，但是测试时不通过？

![primes2](lab1.assets/primes2.png)

解决方案：

![image-20220926180931786](lab1.assets/image-20220926180931786.png)
