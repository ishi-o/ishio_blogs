---
title: "Linux: 文件系统"
date: 2024-03-12
categories: [OS, Linux]
tags: [beginner, Linux, file system]
---
<!-- placeholder -->
<!-- more -->
# `Bash Shell`命令

`Bash Shell`是大多数`Linux`发行版的默认`shell`，对于绝大多数命令都可以通过`man`命令查看详细用法和选项

## `root`用户

区别于`Windows`系统的`Administrator`用户和更复杂的权限机制，在`Linux`中，`root`(超级管理员用户)具有最高的权限，可以调度系统的所有东西，结构上十分简单，但使用时更需慎重

默认情况下，`root`用户是锁定的，如果需要暂时使用`root`的权限：

```shell
$ sudo cat /etc/sudoers  # 查看被允许使用sudo命令的用户,只有root权限可查看

$ visudo     # 修改/etc/sudoers文件,只有root权限可修改
$ visudo -c     # 检查/etc/sudoers文件,每次修改后都应运行这条命令

$ sudo [command]   # root权限运行命令,首次使用需输入当前用户密码
sudo options:
  -i      # 暂时切换到root用户
  -l      # 查看当前权限
  
$ su user     # 切换用户,除非是root用户,否则需输入对方用户的密码,

$ exit      # 退出当前用户
```

需要反复强调的是，平常开发几乎用不上`root`权限，短暂需要使用也应使用后尽快`exit`；不像`Windows`系统，`Linux`系统的软件运行时很少要求`root`权限，这让用户增加戒心而非像使用`Windows`一样无脑授权

开发中，应清楚知道自己在干什么，知道为什么一定要用`sudo`，并尽量少用它

## 选择包管理工具

有两种选择：`apt-get`与`apt`，后者被视为前者的加强版，是`apt-cache`和`apt-get`的集合，最好不要尝试其它包管理工具，否则可能扰乱`apt`的数据库，使一些命令变成危险操作

`apt`获取软件资源的仓库位于`/etc/apt/sources.list`中，如果要修改，记得小心、备份

它们都需要管理员权限，常用命令如下：

```shell
$ sudo apt update    # 列出可更新软件名单
  sudo apt install <p1> <p2> # 安装p1,p2,...软件 若存在则尝试更新
  sudo apt upgrade    # 升级软件,应当定期执行该命令以保证系统稳定性
  sudo apt remove <p1> <p2>  # 删除p1,p2,...软件
  sudo apt purge <p1> <p2>  # 删除p1,p2,...软件,并删除相关依赖包和配置文件,有危险
  sudo apt autoremove   # 删除不再被需要的依赖包,如果一直用的是apt包管理工具,则不危险
  sudo apt list [option]  # 列出...
install options:
  --no-upgrade     # 若软件已安装,则不更新
  --only-upgrade    # 若软件未安装,则不安装;若已安装,则尝试更新
```

## 文件系统

区别于`Windows`系统的盘符分区，`Linux`采取的是让根目录`/`作为所有文件或目录的祖宗路径，将所有文件纳入单个虚拟目录结构中，文件路径中没有指示其所在驱动器的信息

但硬盘驱动器不会只有一个，一般装载着系统文件的驱动器为根驱动器，在其它驱动器的文件会通过挂载点与根驱动器联系起来

路径分为绝对路径和相对路径，形如`./path`和`../path`路径前带点的路径即相对路径，分别表示相对当前目录下和相对于上级目录下，可以用多个`../`来跳转到更上级目录

关于文件的常用命令如下：

```shell
$ touch <file>    # 若文件不存在,则创建它

$ cat <file>    # 一次性展示文件内容
cat options:
  -A      # 查看文件所有字符
  -n      # 查看时附上行号
  
$ less      # 是more的升级版,用于分页查看文件
#[less is more]

$ tail/head <file>   # 查看文件的末尾/头部若干行,默认10行

$ file <file>    # 查看文件类型,事实上也可查看目录,但现在发行版都有彩色显示
```

关于目录的常用命令如下：

```shell
$ cd <dir>     # 跳到指定路径下
#[change directory]

$ ls <file>/<dir>   # 查看[文件是否存在]/[目录下非隐藏文件]
#[list]
ls options:
  -a      # 查看目录下所有文件
  -l      # 查看目录下非隐藏文件的详细信息
  -R      # 递归查看,即会查看目录及其所有子目录
  -al      # 查看目录下所有文件的详细信息
  
$ la      # 相当于ls -a
$ ll      # 相当于ls -al

$ mkdir <dir>    # 创建空目录
#[make directory]

$ rmdir <dir>    # 删除目录,不常用
#[remove directory]
```

有些命令对文件与目录都有效：

```shell
$ cp <file> <file>/<dir>
#[copy]
# 将前者复制到后者中,若为文件路径则以后者命名,若为目录路径则以前者命名
# 因为一个目录下文件不能重名,所以后者路径不能是同名文件和当前目录
$ rm <file>     # 删除文件
#[remove]
cp&rm options:
  -i      # 覆盖/删除时发出询问,最好都加上这条选项,保证安全
  -r      # 递归复制/删除,可用于用于复制目录
  -f      # 强制执行,带有一定危险
$ sudo rm -rf <dir>   # 十分危险的操作
       # 尽管现在的发行版在要删除根目录时会发出警告,也不要轻易尝试

$ mv <file>/<dir> <dst>  # 移动文件/目录至另一路径,常用于文件或目录改名、移动文件
#[move]
mv options:
  -i      # 覆盖文件时发出询问
  -f      # 强制执行
```

## 文件权限

输入`ls -al`，可以看到文件和目录的一长串信息，其中头部内容就是该条文件的权限，它含有10个字符：

第一个字符代表文件类型，常见的有`l、d、-`三种，分别对应软链接文件、目录、普通文件

其余九个字符每三个为一组，分别对应属主权限(`u,user`)、属组权限(`g,group`)、其它用户权限(`o,other`)

三个字符`r、w、x`，分别对应可读、可写、可执行

关于权限的常用命令如下：

```shell
$ chmod <paras> <file>  # 改变文件模式
#[change mode]
chmod paras:
  [u/g/o/none][+-=][rwx] # 增/减/等于指定权限,默认改变三者的权限
  [mmm]      # 一串三位、八进制数,每一位m表示对应用户的权限
  # m=nnn     # 每一位m等于一串三位、二进制数,每一位n表示是否拥有对应rwx权限
```

`Linux`系统不同组会有不同权限，我们创建的第一个用户属于`sudo`组，有些发行版会为该用户创建一个独立的组，该用户创建的文件属于这个组，而通过`sudo`命令创建的文件又属于`root`组...

这种复杂灵活的机制让`Linux`在共享资源时保持有序、安全，关于组的常用命令如下：

```shell
$ sudo groupadd    # 创建组,和addgroup差不多
groupadd options:
  -g [id]     # 指定GID

$ groupmod     # 修改组属性
groupmod options:
  -n      # 改名
  
$ sudo groupdel    # 删除组,和delgroup差不多

$ cat /etc/group   # 查看所有组的信息,它们储存在/etc/group中
       # 每行信息为:组名,密码,GID,[组员]
       
$ chgrp <newGrp> <file/dir> # 改变文件/目录所属组
#[change group]
```

用户分为系统用户和普通用户，系统用户是某些服务运行时所用的账户，系统一般会预留500或1000个`UID`给它们(即普通用户`UID`会从500或1000开始数)

从未改变默认值的默认创建下，新用户将被分配到`GID=100`的公共组、不会为其设置过期日期

一个用户只能有一个主要组，可以有多个附属组，登录时属于主要组，可通过命令切换到其它所属组

关于用户的常用命令如下：

```shell
$ cat /etc/passwd   # 查看所有用户的信息,它们储存在/etc/passwd中

$ id      # 查看用户信息
$ groups     # 查看用户所在所有组

$ sudo useradd    # 创建用户,和adduser是差不多
useradd options:
  -g <grp>     # 指定主要组
  -r      # 创建系统用户
  -u [id]     # 指定UID
  -e      # 指定过期时间
  -G [grps]     # 指定用户附属组
  
$ sudo userdel    # 删除用户信息,不删除home目录下文件,和deluser差不多
userdel options:
  -r      # 会删除home目录下文件,使用时需谨慎

$ usermod     # 改变用户信息,一些选项与adduser的选项相同
usermod options:
  -l      # 改名
  -L/-U      # 锁定/解锁用户
  -p      # 更改密码
  -g      # 更改用户主要组,重新登录后生效
  -G [grps]     # 更改用户附属组,是更改而非添加,具有一定危险

$ newgrp     # 切换到其它所属组

$ chage      # 更改用户过期时间
```

无论是在`/etc/group`还是在`/etc/passwd`，出于安全因素，里面的密码字段都是`'x'`，真正的密码储存在`/etc/shadow`内，只有`root`用户有权限查看与修改，更改密码的命令如下：

```shell
$ passwd <user>    # 修改用户密码
passwd options:
  -d      # 删除用户密码

$ gppasswd <grp>   # 修改组密码,组外用户通过密码可暂时拥有其权限,不常用
gppasswd options:
  -a <user>     # 使grp成为user的一个附属组,常用
```

## 链接

如果多个项目需要使用相同文件，为了节省空间、或是方便管理与维护，`Linux`提供了链接功能

链接分为软链接与硬链接：

- 软链接(也称符号链接)：类似快捷方式，在软链接文件中存放的是源文件的路径，故占用空间小
- 硬链接：将创建一个虚拟文件，其本质上与源文件是同一个文件，共用同一个`iNode`，指向同一份数据块，即占用同一份空间

无论是软链接还是硬链接，通过链接文件都可以修改源文件的内容，区别是：

- 删除掉软链接的源文件，则所有指向该源文件的软链接均失效；但如果创建了另外的、同路径的文件，则这些软链接又会指向这个新文件
- 只有删除掉全部的硬链接，它们的源文件才会从磁盘中释放掉

需要注意的是，硬链接不能对目录使用，也不能在不同文件系统间使用；而由于软链接存储的是路径，所以通常不应使用相对路径，否则容易失效

关于链接的常用命令如下：

```shell
$ ln <file1> <file2>  # 给file1,file2创建硬链接;file1为源文件,必须存在
ln options:
  -s      # 给file1,file2创建软链接;file1为源文件,必须存在
  
$ pwd      # 显示所在目录的路径,可用于创建软链接时快捷找到路径
#[print work directory]
pwd options:
  -P      # 如果所在目录为软链接目录,则显示当前目录指向的源目录的路径
  
$ unlink <file>    # 删除链接,事实上多使用rm -i而非unlink
```

如要删除链接目录，需要十分小心，假设我们有源目录`/tmp`和链接目录`/link->/tmp`：

```shell
rm ~/link/    # 删除/tmp,即源目录的所有文件,链接目录本身会保留
rm ~/link     # 这是正确方式,删除链接目录而非其中的文件
```
