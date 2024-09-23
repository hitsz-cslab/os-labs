# 操作系统实验指导书 - 2024秋 | 哈工大（深圳）

!!! warning "声明"
    本课程资料仅限于哈尔滨工业大学（深圳）《操作系统》2024年课程实验使用，如有其他用途，请联系夏文老师：xiawen@hit.edu.cn。

!!! tip "注意事项"
    请同学们遵守学术诚信（[What is Academic Integrity](https://integrity.mit.edu/))。
    你可能会在同一个时间段内需要完成多项课程任务，或者准备考试，或者其他……但是，无论你处于何种程度的压力之下，我们都希望你能做到管理好时间，以诚实正直的态度来完成课程实验。


## 0. 导读

&emsp;&emsp;本实验文档为哈尔滨工业大学（深圳）《操作系统》课程实验指导材料。页面顶端为实验的指导书，页面左侧为指导书的各个小节目录，右侧为小节内的索引，页面右上角可详细搜索。

&emsp;&emsp;本学期《操作系统》课程共设置6次实验课，涵盖5个实验项目，总计24个学时。前四个实验项目均以开源操作系统xv6为基础，但已根据哈尔滨工业大学（深圳）的课程要求进行了针对性的修改和扩展。每个实验项目均独立设置，各占4个学时。第五个实验分配8个学时，由我校学生自主设计的文件系统实验案例，该案例在2022年全国大学生计算机系统能力大赛操作系统设计大赛OS功能挑战赛道中获得二等奖。

&emsp;&emsp;<font color=red> **由于我们已经对XV6的实验内容及要求进行了修改，请大家从下面的[“3. 获取XV6实验框架代码”](#3-xv6)仓库下载代码，务必在这套代码上完成实验，使用其他xv6仓库代码的同学极有可能通不过实验考核！！！** </font>  在实验开始之前，请同学们务必先按顺序阅读指导书，掌握实验平台的搭建/使用方法，掌握Linux系统下的一些基本命令/操作，熟练使用GDB调试工具。对于有兴趣有余力的同学，挑战并完成[2020版本的xv6实验](https://pdos.csail.mit.edu/6.828/2020/index.html)的所有任务，若能完成会有意想不到的惊喜！

&emsp;&emsp;为了帮助同学们理解xv6的概貌、代码结构以及调试方法，我校OS课题组、各级学长助教们合力为大家录制的[XV6讲解视频和调试视频](#4)，推荐大家按需观看。

&emsp;&emsp;如果同学们对本课程实验内容、实验安排、实验指导书或者代码框架等各方面有宝贵的意见或建议，请在[piazza在线交流平台](https://piazza.com/harbin_institute_of_technology_shenzhen/fall2024/comp3001/home)（access code：comp3001）提出来或私信老师或助教。


## 1. 课程信息

- 课程名称：操作系统（实验）
- 学期：2024秋季
- 理论课教师：夏文、陈斌、陈芳林、陈俊杰、刘川意、邹翔宇、卢瑶（按任课班级顺序）
- 实验课教师：仇洁婷、苏婷

## 2. 实验相关链接

-  **实验代码** 下载地址：https://gitee.com/ftutorials/xv6-oslab24-hitsz.git
-  **实验工具** 下载地址（校内网）：https://mirrors.osa.moe/misc/
-  **实验提交** 地址（校内网）：http://10.249.12.98:8000/#/login
-  **远程实验环境** （校内网），详见[远程实验环境使用指南](remote_env/)
-  **piazza在线交流平台（access code：comp3001）** 欢迎同学们在piazza上提问，提问的内容第一时间会由助教或老师回答，当然也欢迎同学们来回答：https://piazza.com/harbin_institute_of_technology_shenzhen/fall2024/comp3001/
  
## 3. 获取XV6实验框架代码

&emsp;&emsp;在下载实验代码前，请先搭建好xv6实验平台，搭建方法参考[实验平台搭建以及环境配置](env)。
  
&emsp;&emsp;如果你已经进入实验平台，请在命令行终端中执行：

!!! note "注意事项"
    第一个字符`$`不需要输入，只需要输入`$`后的字符串。

```shell
$ git clone https://gitee.com/ftutorials/xv6-oslab24-hitsz.git
```

&emsp;&emsp;获取框架代码，将克隆xv6-oslab24-hitsz到当前目录。如遇问题请联系老师或助教。

## 4. 视频链接

### 4.1 xv6讲解视频链接
    
1. 【HITSZ操作系统课程组讲解XV6（一）启动过程】 			 
    
    - 视频链接: https://www.bilibili.com/video/BV1mK411S7N9?share_source=copy_web&vd_source=225a99017e082147ac525beeddd6e3e2
    - 课程PPT: [点这里下载](https://gitee.com/hitsz-cslab/os-labs/tree/master/references/xv6原理简析1-进程管理.pdf)
  
2. 【HITSZ操作系统课程组讲解XV6（二）进程管理】 
    
    - 视频链接: https://www.bilibili.com/video/BV1ge4y1J7Je?share_source=copy_web&vd_source=225a99017e082147ac525beeddd6e3e2
    - 课程PPT: [点这里下载](https://gitee.com/hitsz-cslab/os-labs/tree/master/references/xv6原理简析2-内存管理.pdf)
  
3. 【HITSZ操作系统课程组讲解XV6（三）内存管理】 
    
    - 视频链接: https://www.bilibili.com/video/BV1Te4y1i77z?share_source=copy_web&vd_source=225a99017e082147ac525beeddd6e3e2
    - 课程PPT: [点这里下载](https://gitee.com/hitsz-cslab/os-labs/tree/master/references/xv6原理简析3-启动过程.pdf)
    
&emsp;&emsp;也可以观看B站上的[MIT 6.S081/Fall 2020课程视频](https://www.bilibili.com/video/BV19k4y1C7kA?p=1)，通过学习这门课程可以让你对xv6操作系统有一个全面的认识。

### 4.2 xv6调试演示视频链接

1. 【使用gdb调试xv6启动流程（1）】 https://www.bilibili.com/video/BV1zAt6e3EfU/?share_source=copy_web&vd_source=a822dcda3537564ccdd0bb45aa0afe33
   
- 【使用gdb调试xv6启动流程（2）】 https://www.bilibili.com/video/BV1MrtUe7ELC/?share_source=copy_web&vd_source=a822dcda3537564ccdd0bb45aa0afe33
- 【1. VSCode调试xv6内核代码】 https://www.bilibili.com/video/BV1ZB4y1E7X5?share_source=copy_web&vd_source=a822dcda3537564ccdd0bb45aa0afe33

- 【2. VSCode调试xv6用户代码】 https://www.bilibili.com/video/BV1i14y1Y7ZZ?share_source=copy_web&vd_source=a822dcda3537564ccdd0bb45aa0afe33

- 【3. VSCode调试系统调用过程（包含pagetable和汇编）】 https://www.bilibili.com/video/BV12P411J7xq?share_source=copy_webvd_source=a822dcda3537564ccdd0bb45aa0afe33

- 【4. VSCode调试系统调用——从内核到用户，再从用户返回内核】 https://www.bilibili.com/video/BV1ug411m7ir?share_source=copy_web&vd_source=a822dcda3537564ccdd0bb45aa0afe33vd_source=a822dcda3537564ccdd0bb45aa0afe33


## 致谢

&emsp;&emsp;在实验教学过程中，我们有幸邀请到了一些本科生助教和研究生助教，这些同学共同完成了实验指导书的编写、实验代码的修改、调试和实验环境的搭建。感谢这些同学对实验做出的贡献。

&emsp;&emsp;特别感谢对本课程实验贡献的助教同学：

- 17级：胡博涵、施杨
- 18级：李程浩、宫浩辰、潘延麒、黎庚祉、刘定邦
- 19级：赵鹏宇、陈一邹、叶自立、张艺枫
- 20级：曾培鑫、陈佳豪、满洋、杨大荣
- 21级：陈睿玮、石全、王华杰

&emsp;&emsp;未来更多的可能，期待由现在的你们来书写！








