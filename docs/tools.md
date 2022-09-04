# 实验实用工具

对于初入openEuler/Linux的同学，可能对于Linux的命令行（openEuler20.09暂未支持图形化界面）感觉到一脸茫然。为此，我们给同学们推荐几款实用工具，以便帮助同学们能快速上手Linux开发工作。

## 1. SSH 

!!! question "question 1"
	如何实现Linux和Windows这两个操作系统的数据通信、文件共享？

在实验中，我们推荐大家使用ssh（Secure Shell）远程登录协议。

### 1.1 MobaXterm（SSH工具）

常用的SSH工具有Putty、XShell、SecureCRT、MobaXterm等。在实验中，我们推荐同学们使用MobaXterm。

MobaXterm 是一款远程控制工具，支持创建SSH、Telnet、Rsh、Xdmc、RDP、VNC、FTP、 SFTP、串口(Serial COM)、本地Shell、Mosh、Aws、WSL(微软子系统)等超多连接功能。MobaXterm 提供了人性化的操作界面，支持与主流的操作系统连接进行控制和管理操作，功能十分强大。

MobaXterm 下载链接： http://10.249.14.14:8000/index.php/s/7vlEVZPKaMTIpi0

打开MobaXterm，单击打开工具Session（位于左上角）选择New sessions打开新建窗口，选择SSH，输入远程服务器的IP地址、登陆用户名及端口（默认是22，远程实验平台端口号是6666），点击OK。

![0-12.png](tools.assets/0-12.png)

SSH配置信息根据实验环境来配置：

- 远程实验平台。IP地址： `10.249.12.98` ，端口号： `6666` ，用户名和初始密码都是 `你的学号` 。

![image-20201017220316972.png](tools.assets/image-20201017220316972.png)


- 自行部署的实验环境。如果你用的是你自己的实验环境，可以使用ip addr命令来查询IP地址，详见[ip addr 显示或设置网络状态](../Linux/#72-ip-addr)。端口号如果在没有修改的情况下，默认是22。用户名和密码是Linux系统中的用户名和密码。


点击OK后，输入密码（输入过程是没有显示的，输入完成后敲回车键）

![mobaxterm-4.png](tools.assets/mobaxterm-4.png)


即可进入Linux系统的命令行窗口（shell）。

### 1.2 文件传输SFTP服务

SSH远程登录完毕，在左下角没有一个竖着的Sftp，双击它，就出现了远程服务器的文件目录

![mobaxterm-6](tools.assets/mobaxterm-6.png)

接下来在可以点击红色框框的按钮“Upload to current folder”或者“Download selected files”，在windows系统和虚拟机/远程实验平台之间上传或下载文件。



## 2 VScode

!!! question "question 2"
	如何查阅、编辑、运行和调试代码？

在实验中，强烈推荐同学们使用VScode代码编辑器，而不是在物理机上编辑代码再拷贝过去手动运行，这将节省大量的时间。

通过VSCode的强大扩展库，我们可以在Windows系统上使用VScode一站式创建，编辑，构建，运行和调试远程主机上的工程文件，就像使用Code::Blocks操作本地工程一样。

### 2.1 配置openEuler以支持VScode访问

!!! warning ""
	如果你用的系统不是openEuler（例如远程实验平台或者其他Linux系统），那你可以忽略这一小节。

openEuler系统默认不支持Vscode远程登录，需要在openEuler虚拟机中做如下修改：

**step1：** 首先需要在openEuler虚拟机中切回root用户，如果不是root用户，输入exit即可返回root用户。


```console
[cs@localhost root]$ exit
exit
[root@localhost ~]#
```

**step2：** 在/etc/ssh/sshd_config文件中做如下修改：

```bash
AllowAgentForwarding yes
AllowTcpForwarding yes
GatewayPorts yes
```

用vi命令打开sshd_config：

vi是所有Unix及Linux系统下标准的编辑器，它就相当于windows系统中的记事本一样，它是我们使用Linux系统不能缺少的工具。vim可以当作vi的升级版本，学会它，你将在Linux的世界里畅行无阻：）

```console
# vi /etc/ssh/sshd_config
```

输入"/Allow"，按下回车，就会发现定位到了#AllowAgentForwarding yes这一行，我们把前面的#删去即可。

如何删除呢？此时按下i会进入插入模式，下面显示一个--INSERT--，然后正常删除即可。接着修改AllowTcpForwarding yes和GatewayPorts yes。

接下来如何保存呢？按下esc键，即可退出插入模式，然后，输入":wq"即可保存并退出。（输入":q!"是不保存并退出）

**step3：**  重启ssh服务

设置好ssh的代理转发后，现在还没有生效，需要重启一下ssh服务，命令行如下：

```console
# systemctl restart sshd.service
```

### 2.2 安装VScode

在 https://code.visualstudio.com/ 下载并在物理机（Windows系统）上安装VSCode软件。

打开后在左侧边栏的扩展商店中搜索并安装以下扩展

> 1. Chinese (Simplified) Language Pack for Visual Studio Code
> 2. Remote-SSH
> 3. C/C++

安装完成后重启VSCode。

### 2.3 使用VSCode登录远程环境

!!! 提示
    使用前，需要安装Remote-SSH插件。

在左侧边栏的远程资源管理器工具栏中点击+号，在顶部输入 `ssh 你的用户名@IP地址 -p 端口号` 命令并一路回车。

此时在工具栏中可以看到新添加的远程主机。在远程电脑上点击连接，输入密码后将会打开新窗口连接到远程主机中。

在连接到远程主机之后，需要在再次搜索并安装上述扩展，将其安装在远程主机上。C/C++扩展在安装后可能需要一段时间来下载依赖文件。

![image-20201016162754878](remote_env.assets/image-20201016162754878.png)

回车，提示Host Added!

在此处点击连接：

![image-20201016162859685](remote_env.assets/image-20201016162859685.png)

输入密码后，稍等片刻，等待窗口成为如下时，即连接成功：

![image-20201016163031421](remote_env.assets/image-20201016163031421.png)


此时，可以像本地一样使用远程机器进行工作了。

VScode的详细用法查看“使用VSCode阅读源代码教程.pdf”。

## 3 git

!!! question "question 3"
	你是否遇到过这样的情况，一个项目由于不停地修改，每次都是复制粘贴副本，产出的文件副本就越来越多。而随着版本的增多，你还记得这些版本各自都修改了什么吗：？

为了能够更方便我们管理不同版本的文件/代码，强烈建议同学们使用git进行版本控制。

### 3.1 同步上游仓库
如果本课程实验修改了实验上游仓库（如果课程实验有修改，第一时间将在群里推送通知，请关注群消息！），而你需要将本课程实验的修改同步到自己的仓库，请按以下步骤操作：

#### 3.1.1 检查上游仓库设置
为了同步上游仓库，我们需要先设置上游仓库。检查上游仓库是否已经设置好，可以使用`git remote -v`指令查看。  
如果上游仓库未设置完成，那么指令的执行效果大概是这样的：

```console
lgz_admin@OSLabExecNode0:~/xv6-labs-2020$ git remote -v
origin  git@gitee.com:willson0v0/xv6-labs-2020.git (fetch)
origin  git@gitee.com:willson0v0/xv6-labs-2020.git (push)
```

可以看到，其中只有一个远程仓库，即`origin`；如果上游仓库已经设置完成，那么指令的执行效果大概是这样的：

```console
lgz_admin@OSLabExecNode0:~/xv6-labs-2020$ git remote -v
origin  git@gitee.com:willson0v0/xv6-labs-2020.git (fetch)
origin  git@gitee.com:willson0v0/xv6-labs-2020.git (push)
upstream        git@gitee.com:hitsz-lab/xv6-labs-2020.git (fetch)
upstream        git@gitee.com:hitsz-lab/xv6-labs-2020.git (push)
```

其中，远程仓库`origin`的地址（在此处显示为`git@gitee.com:willson0v0/xv6-labs-2020.git`）应该是是你自己的远程仓库地址，而远程仓库`upstream`的地址应该显示为本课程实验提供的远程仓库地址，即`git@gitee.com:hitsz-lab/xv6-labs-2020.git`或者`https://gitee.com/hitsz-lab/xv6-labs-2020.git`。  
如果你的远程仓库已经设置完成，那么跳到[3.1.3](#313)继续。否则，请按照[3.1.2](#312)节完成远程仓库设置。

#### 3.1.2 完成远程仓库设置

如果你没有设置上游仓库（即`upstream`），并且 **origin仓库的链接被设置为本课程实验的远程仓库而非自己的远程仓库** ，那么请先参照[3.2](#32)节，将代码发布至自己的远程仓库。
如果你的`origin`仓库正确设置为自己的远程仓库，那么请输入以下指令，设置上游仓库为本课程实验提供的仓库：

```bash
$ git remote add upstream git@gitee.com:hitsz-lab/xv6-labs-2020.git
```

此时，如果在输入`git remote -v`，应该能显示出[3.1.1](#311)中设置完成一样的效果。

#### 3.1.3 同步上游仓库更改

在这里，我们以将上游仓库的`syscall`分支同步到本地的`syscall`分支为例。键入以下指令，以获取上游更改、切换到本地`syscall`分支、完成与上游仓库`syscall`分支的合并，并最后上传到自己的远程仓库：

```bash
$ git fetch --all
$ git checkout syscall
$ git merge upstream/syscall --no-edit
$ git push origin
```

!!! warning "检查工作区更改"
	切换分支之前，我们需要保存工作区的更改。如果切换时提示`error: Your local changes to the following files would be overwritten by checkout`，参考[这篇Q&A](#353-checkout)，检查工作区并commit你的更改。

!!! warning "初次Checkout？"
	如果你此前从未Checkout过该分支，Checkout时可能会向你报告如下错误：
	```console
	lgz_admin@OSLabExecNode0:~/git_demo/xv6-labs-2020$ git checkout syscall
	error: pathspec 'syscall' did not match any file(s) known to git
	hint: 'syscall' matched more than one remote tracking branch.
	hint: We found 2 remotes with a reference that matched. So we fell back
	hint: on trying to resolve the argument as a path, but failed there too!
	hint:
	hint: If you meant to check out a remote tracking branch on, e.g. 'origin',
	hint: you can do so by fully qualifying the name with the --track option:
	hint:
	hint:     git checkout --track origin/<name>
	hint:
	hint: If you'd like to always have checkouts of an ambiguous <name> prefer
	hint: one remote, e.g. the 'origin' remote, consider setting
	hint: checkout.defaultRemote=origin in your config.
	```
	这是由于，两个远程仓库（你自己的和上游的）都存在`syscall`分支，git不知道跟踪哪一个导致的。这个时候，只需要执行以下命令：
	```bash
	git checkout --track origin/syscall
	```
	即可。

如果在`git merge upstream/syscall`一步提示存在冲突，说明你和本课程实验修改过同一个文件，git无法自动合并。冲突的输出应该类似于下面这样：

```console
lgz_admin@OSLabExecNode0:~/git_demo/xv6-labs-2020$ git merge upstream/syscall
CONFLICT (add/add): Merge conflict in time.txt
Auto-merging time.txt
Automatic merge failed; fix conflicts and then commit the result.
```

如果发生冲突，请参照[3.3.3](#333)节完成冲突合并操作。

### 3.2 将代码发布至自己的远程仓库

如果你的代码是直接从本课程实验仓库clone而来，而非先fork出自己的远程仓库、再进行clone，那么这篇教程会指引你设置自己的远程仓库、并将代码发布至自己的远程仓库处。

!!! info "这么做有什么好处？" 
	如果你在不同的电脑上工作，比如同时在实验室的电脑、自己的电脑和远程实验环境上操作，那么远程仓库可以帮助你在不同设备之间同步自己的工作进度。

#### 3.2.1 设立自己的远程仓库（以gitee为例）

登录gitee，右上角可见一个加号，点击它下拉菜单中的“新建仓库”。

![new_repo](tools.assets/new_repo.png)

点击进入，并为自己的仓库设置一个你喜欢的名称。你可以选择公开仓库或者私有仓库。完成后点击“创建”，即可创建一个空的远程仓库。

![new_repo2](tools.assets/new_repo2.png)

!!! warning "请不要初始化仓库！"
	你的仓库在本地，因此请不要初始化仓库、设置模板或选择分支模型。总的而言，我们的目标是创建一个空的远程仓库，并且把我们本地的仓库塞进去。

仓库创建完成后，会来到这个界面。将这个界面的SSH地址复制下来：

![new_repo3](tools.assets/new_repo3.png)

#### 3.2.2 清理本地工作树

在命令行界面，输入`git status`。如果工作树干净，其会显示如下内容：

```console
lgz_admin@OSLabExecNode0:~/git_demo/xv6-labs-2020$ git status
On branch syscall
Your branch is up to date with 'origin/syscall'.

nothing to commit, working tree clean
```

那么，你可以跳过这一步，前往[3.2.3](#323)继续。如果本地有尚未commit的更改，则命令会显示出你更改过的内容，提示其尚未被跟踪：

```console
lgz_admin@OSLabExecNode0:~/git_demo/xv6-labs-2020$ git status
On branch syscall
Your branch is up to date with 'origin/syscall'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   time.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

如果本地有尚未commit的更改，请参照[3.3.1.1](#3311)中的操作（命令行操作）或者[3.3.2](#332)中的操作（VS code图形化界面操作），完成commit。你没有推送到上游仓库（本课程实验提供的远程仓库）的权限，所以不必push。

#### 3.2.3 修改远程仓库地址并推送

在命令行中，输入以下命令：

```bash
$ git remote set-url origin git@gitee.com:willson0v0/my_xv6_repo.git
```
将其中`git@gitee.com:willson0v0/my_xv6_repo.git`替换成[3.2.1](#321)中复制下来的远程仓库地址。可以输入`git remote -v`检视远程连接有没有更改成功。完成后，切换到你想要推送的分支，并上传到远程。这里以`syscall`分支为例：

```bash
$ git push -u origin syscall
```

这会将`syscall`分支设置成跟踪远程`origin`的`syscall`分支，并推送。现在回到浏览器并刷新仓库页面，你应该能看到上传已经完成。

### 3.3 常用git操作

你可以选择使用命令行完成`commit`/`push`/`pull`/`merge`等操作，也可以通过VS Code内建的图形化界面完成这些操作。

git工作过程

![img](tools.assets/git-command.jpg)

说明：

- workspace：工作区
- staging area：暂存区/缓存区
- local respository：版本库或本地仓库
- remote repository：远程仓库

以下是git的说明：

https://git-scm.com/book/zh/v2

!!! info "什么是Commit？"
	你可以将`Commit`视作是一个个小小的里程碑，它标示了你的代码进度；版本控制工具（比如git）的版本就是基于这一个个小里程碑实现的：你可以修改完代码后创建新的里程碑；也可以将放弃出现了问题的代码修改，将代码回溯到此前的某个里程碑处。	
	每一个Commit都有一个独特的hash值，用以和其他Commit区分；Commit还会标示是谁、在何时进行了这个Commit，这对协作开发有很大的帮助。

!!! info "什么是Push/Pull？"
	通过Commit系统，我们可以完成一定的版本控制功能。但是，为了完成代码协作与同步，我们还需要把代码放在一个各个写作者或者自己的各个工作环境都能访问到的地方——一般而言，也就是远程仓库。Push操作就是将本地的更改（也就是Commit们）推送到远程仓库的修改；而Pull则是将远程仓库的更改同步到本地。

!!! info "什么是Merge？"
	我们知道，Git作为版本控制工具可以允许我们在同一个代码仓库内进行协作。如果两个开发者同时进行了修改，那么就有可能出现冲突；又或者我们要将代码的两个分支合并。这个时候，我们就需要进行Merge操作。如果无法完成自动冲突合并，可能会需要手动完成冲突合并操作。

#### 3.3.1 使用命令行完成操作

##### 3.3.1.1 Commit

```bash
$ git add 你要保存更改的文件们
# 或者，git add --all 以添加所有更改。
$ git commit -m "本次Commit你做了什么"
```
建议每次Commit都使用有效的Commit信息，以便于自己追踪代码进度。

##### 3.3.1.2 push
push可以将自己的进度同步到远程仓库。
```bash
$ git push
```

##### 3.3.1.3 pull
pull可以将远程仓库的进度同步到本地。
```bash
$ git pull
```

##### 3.3.1.4 merge
merge一般用于分支之间的合并，也可以将上游的远程分支和本地分支进行合并。
```bash
$ git checkout 你想合并的本地分支
$ git merge 远程仓库名称/分支名称
```

!!! waning "仓库权限"
	你可以从公共仓库拉取，但你只能推送到你拥有权限的仓库。为了验证你的身份，你需要设置自己的ssh密钥对。请参照[3.4节](#34-ssh)完成设置。

#### 3.3.2 使用VS Code内建的图形化界面完成操作

VS Code的版本控制界面在左侧从上往下数第二个按钮处；下方会显示当前所处的分支，点击可以切换当前所处分支；右侧的圆形双箭头则代表与远程环境同步。右侧Changes显示工作区中被修改的文件，而Staged则显示将被加入到下次Commit中的修改。如图，这个界面显示我们有两个被修改的文件：

![vs_git](tools.assets/vs_git.png)

鼠标移至Changes或下面所列文件处，点击右侧的加号，可以将文件加入到Staged区中：
![stage](tools.assets/stage.png)
![stage2](tools.assets/stage2.png)

此时，这些文件已经准备好被Commit。在上方框内输入Commit信息，并点击 √ 按钮以提交：

![commit](tools.assets/commit.png)

在下方的同步按钮处，可以发现有0个提交需要拉取、1个提交需要推送。点击这个按钮以同步，或者点击左上方那个巨大的蓝色的Sync Changes（同步更改）：

![push](tools.assets/push.png)

这样，你的代码就保存到云端了。

!!! waning "仓库权限"
	你可以从公共仓库拉取，但你只能推送到你拥有权限的仓库。为了验证你的身份，你需要设置自己的ssh密钥对。请参照[3.4节](#34-ssh)完成设置。

#### 3.3.3 合并冲突更改

如果有两个Commit的内容出现冲突，merge的时候就有可能需要合并冲突更改。建议在VSCode中进行冲突修改。例如此处，我在对本地`syscall`做修改之后，将其与远程`upstream/origin`做修改，并触发了冲突：
![merge_cf](tools.assets/merge_cf.png)
从左侧红框中“Merge Changes”栏中可以看到出现冲突的文件。点开文件，右侧会出现冲突视图。上方青色部分指的是当前（本地）的修改，而下方蓝色部分则是远程(`upstream/origin`)的修改。根据实际情况，可以从上方几个按钮中选择合适的那个。这次，我想要保存我自己的修改、而丢弃远程的修改，故选择`Accept Current Change`。随后，和Commit前一样，将刚才的修改Stage，并点击上方 √ 号提交Commit，并推送。这样，我们就完成了冲突更改的合并。

### 3.4 ssh密钥设置

通过ssh密钥，我们可以简单的完成从本地到远程仓库的认证。跟随以下步骤，以完成ssh密钥创建，并部署到远程仓库：

#### 3.4.1 生成ssh密钥对

在终端中，输入以下指令以生成ssh密钥对:
```bash
ssh-keygen -t ed25519 -C "name_of_ssh_keypair"  
```

!!! info "密钥对的名字"
	密钥对的名字在教程中常用邮箱地址代替，这是一个某种意义上的传统。此外，为了兼容性考虑，不建议在密钥对名字中使用除了ASCII之外的任何字符。

输入后，根据终端的提示，按下三次回车即可完成ssh密钥生成，或者你也可以自定义这些需要输入的内容：

```console
lgz_admin@OSLabExecNode0:~/.ssh$ ssh-keygen -t ed25519 -C "willson0v0@willson0v0.com"
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/lgz_admin/.ssh/id_ed25519):	# 按下回车以使用默认路径（~/.ssh/id_ed25519）
Enter passphrase (empty for no passphrase):								# 按下回车以使用空密码，或者自定义密码再回车。通常可以不设密码。
Enter same passphrase again:											# 按下回车确认空密码，或者重新输入上一行中自定义的密码。
Your identification has been saved in /home/lgz_admin/.ssh/id_ed25519
Your public key has been saved in /home/lgz_admin/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:1oDtnF8c1FXpDc+6U1Gogf8Swur0N/Dh6uFJFIlVJ9o willson0v0@willson0v0.com
The key's randomart image is:
+--[ED25519 256]--+
|          .o+..o=|
|       o o.=.o+..|
|      . +.+.Eo.+o|
|       o +oo+. o+|
|        S.o.oo ..|
|       .oo..o o .|
|       o .++ o o |
|        .o.o= o  |
|         .=o . . |
+----[SHA256]-----+
```

现在，你的密钥对已经生成完成。如果遵循默认设置，那么`~/.ssh/id_ed25519`是你的私钥文件，`~/.ssh/id_ed25519.pub`则是你的公钥文件。通过`cat`可以展示其中的内容：

```console
lgz_admin@OSLabExecNode0:~/.ssh$ cat ~/.ssh/id_ed25519	# 这是展示用的密钥对。请不要使用该密钥对，也不要讲自己的私钥展示给任何人。
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
QyNTUxOQAAACB07KkIxYhV60DnPqKY1E0AFoRgqnKRUy1qoh5M9EYJeAAAAKDKf/BCyn/w
QgAAAAtzc2gtZWQyNTUxOQAAACB07KkIxYhV60DnPqKY1E0AFoRgqnKRUy1qoh5M9EYJeA
AAAEBixqKUUTawYfl+mnEcqtxWT5up8xfbyaN438Iqjs3Gu3TsqQjFiFXrQOc+opjUTQAW
hGCqcpFTLWqiHkz0Rgl4AAAAGXdpbGxzb24wdjBAd2lsbHNvbjB2MC5jb20BAgME
-----END OPENSSH PRIVATE KEY-----
lgz_admin@OSLabExecNode0:~/.ssh$ cat ~/.ssh/id_ed25519.pub
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHTsqQjFiFXrQOc+opjUTQAWhGCqcpFTLWqiHkz0Rgl4 willson0v0@willson0v0.com
```

!!! warning "存储位置"
	ssh agent会自动装载~/.ssh/id_rsa或者~/.ssh/id_ed25519等密钥文件。如果你更改了密钥文件的位置，可能需要ssh-agent额外添加密钥到身份。

!!! warning "保护好你的私钥！"
	私钥（`~/.ssh/id_ed25519`）顾名思义，是只有你才知道的密钥。请绝对不要将私钥泄露，因为任何持有这个私钥的人都可以伪装成是你。

!!! warning "关于密码"
	可以为ssh密钥设置密码，以保证密钥的安全。不过，通常情况下这并没有必要，而且这会让各种ssh变得很麻烦：每次push或者pull都要输一次密码。因此，我们不建议你为ssh密钥设置密码。

#### 3.4.2 将密钥部署到Gitee

如果你要访问的远程仓库位于Gitee，那么你需要将你的 **ssh公钥** 部署到Gitee。首先，打开[Gitee页面](https://gitee.com/)，登录，并点击左上角个人头像-设置：

![gitee_config](tools.assets/gitee_config.png)

随后，点击左侧的ssh公钥，并将此前生成的 **ssh公钥** 内容拷贝进公钥区。你可以通过`cat ~/.ssh/id_ed25519.pub`从终端获得这个文件的内容。随后，你可以编辑这个公钥的标题。最后，点击最下方橙色的“确定”，即可完成公钥的添加。

![gitee_addssh](tools.assets/gitee_addssh.png)

我们可以通过以下指令，尝试连接到Gitee，以验证公钥是否添加成功：

```console
lgz_admin@OSLabExecNode0:~$ ssh -T git@gitee.com
Hi 用户名! You've successfully authenticated, but GITEE.COM does not provide shell access.
```

#### 3.4.3 将密钥部署到Github

如果你要访问的远程仓库位于Github，那么你需要将你的 **ssh公钥** 部署到Gitee。首先，打开[Github页面](https://gitee.com/)，登录，并点击左上角个人头像-Settings：

![github_setting](tools.assets/github_setting.png)

随后，点击左侧SSH and GPG keys，并点击右上角的New SSH key：

![github_addssh](tools.assets/github_addssh.png)

最后，输入SSH key的标题，将此前生成的 **ssh公钥** 内容拷贝进公钥区，并点击下方绿色`Add SSH key`即可。

![github_addssh2](tools.assets/github_addssh2.png)

我们可以通过以下指令，尝试连接到Github，以验证公钥是否添加成功：

```console
lgz_admin@OSLabExecNode0:~$ ssh -T git@github.com
Hi 用户名! You've successfully authenticated, but GitHub does not provide shell access.
```

### 3.5 常见问题

#### 3.5.1 无法推送怎么办？

如果你尝试推送到远程仓库，或者从远程仓库拉取，但是git却显示出这样的错误：

```console
git@gitee.com: Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights 
and the repository exists
```

这表明你没有权限推送到这个远程仓库。一般而言，这是由于你没有设置好自己的ssh密钥造成的。请参照[3.4节](#34-ssh)，设置自己的ssh密钥。

#### 3.5.2 我怎么知道我修改了哪些文件？

通常而言，实验要求同学们提交 **所有** 被修改过的文件。通过将当前工作路径与上游仓库（课程实验提供的仓库）对应分支的内容相比较，可以方便地知道自己曾经修改过哪些文件，并打包提交。这里，我们将当前分支与上游仓库的`syscall`分支相比较：

```console
lgz_admin@OSLabExecNode0:~/git_demo/xv6-labs-2020$ git fetch --all
Fetching origin
Fetching upstream
lgz_admin@OSLabExecNode0:~/git_demo/xv6-labs-2020$ git diff --stat upstream/syscall *
 kernel/kalloc.c  | 2 ++
 kernel/syscall.c | 1 +
 kernel/syscall.h | 1 +
 kernel/sysinfo.h | 1 +
 4 files changed, 5 insertions(+)
```

要比较其他分支，将第二条指令中的`syscall`改为对应分支名即可。

!!! warning "请先在本地切换到对应分支！"
	在比较的时候，你需要将本地分支与上游对应分支进行比较。举例而言，你不会想比较本地的`util`分支和远程的`syscall`分支，因为这两个不是一个实验的。请先使用`git checkout 分支名`以切换到对应分支，并在使用`git diff --stat upstream/分支名 *`时指定 **同一个** 上游分支。

!!! warning "未设置上游仓库？"
	如果git报告“`fatal: ambiguous argument 'upstream/分支名': unknown revision or path not in the working tree.`”，说明你尚未设置上游仓库。请先按本指南中的[3.1-同步上游仓库](#31)一节完成上游仓库的设置与同步，再进行比较。

#### 3.5.3 为什么我不能切换分支（checkout）？

通常而言，不能切换分支的主要原因是你当前有尚未保存的修改，此时checkout的话会触发以下错误：
```console
lgz_admin@OSLabExecNode0: ~/git_demo/xv6-labs-2020$ git checkout util                                                                                                ─╯
error: Your local changes to the following files would be overwritten by checkout:
        Makefile
Please commit your changes or stash them before you switch branches.
Aborting
```

通过`git status`指令，我们可以检查当前git的状态。如果你的输出如下所示：

```console
lgz_admin@OSLabExecNode0: ~/git_demo/xv6-labs-2020$ git status
On branch syscall
Your branch is up to date with 'origin/syscall'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   Makefile

no changes added to commit (use "git add" and/or "git commit -a")
```

这说明当前你的工作区有尚未保存的更改。请参考[3.3.1.1](#3311-commit)或者[3.3.2](#332-vs-code)节，完成commit操作。
或者，如果你希望直接放弃掉上一次commit后的 **所有更改** ，那么你也可以使用`-f`选项强制切换分支，例如`git checkout -f syscall`。

## 4 gcc

!!! question "question 4"
	对于习惯使用Windows现有的IDE来编译程序的我们，面对Linux的命令行，应该如何编译自己写好的代码：？

通常情况下，Linux用户更喜欢直接编写相应的GCC命令来编译程序。大部分Linux发行版中都会默认安装GCC编译器。

### 4.1 GCC常用编译选项

命令格式：gcc [选项] [文件名]

-E: 仅执行编译预处理

-S：将C代码转换为汇编代码

-c：仅执行编译操作，不进行链接操作

-o: 指定生成的输出文件

### 4.2 GCC编译的四个阶段

1、将hello.c预处理输出hello.i文件

```bash
gcc -E hello.c -o hello.i
```

2、将预处理输出文件hello.i汇编成hello.s文件。

```bash
gcc -S hello.i -o hello.s
```

3、将汇编输出文件hello.s编译输出hello.o文件

```bash
gcc -c hello.s -o hello.o
```

4、将编译输出文件hello.o链接成最终可执行文件hello。

```bash
gcc hello.o -o hello
```

或直接输入下面代码一步到位

```bash
gcc hello.c -o hello
```

也可以有多个模块、多个文件一起编译，请自行百度。

## 5 Makefile

!!! question "question 5"
	什么是makefile？作为一个专业的程序员，特别是在Unix（Linux）环境下开发的程序员，如果不会使用GNU make来构建和管理自己的工程，应该不能算是一个合格的专业程序员。会不会写Makefile，从一个侧面说明了一个人是否具备完成大型工程的能力。因为，Makefile关系到了整个工程的编译、连接等规则。为工程编写Makefile的好处是能够用一个make命令来完成“自动化编译"，极大地提高了软件开发的效率。make是一个命令工具，它用于解释makefile中的指令。

编写规则：

```makefile
<target> : <prerequisites> 
[tab]  <commands>
```

为了大家能更为清晰地了解Makefile的基本概念，我们来写一个简单的Makefile。

直接使用gcc编译hello.c

```bash
$ gcc -o hello hello.c
```

那么，简易的Makefile可以写成如下的样子，之后执行make即可产生hello可执行文件

```makefile
all: hello.c
	gcc -o hello hello.c
```

建议同学们自己尝试写一个简单的Makefile文件，这里为同学们提供两个网址供大家学习参考：

[跟我一起写Makefile:MakeFile](https://wiki.ubuntu.org.cn/%E8%B7%9F%E6%88%91%E4%B8%80%E8%B5%B7%E5%86%99Makefile:MakeFile%E4%BB%8B%E7%BB%8D)

[Reading-Makefiles](http://www.gnu.org/software/make/manual/make.html#Reading-Makefiles) 
