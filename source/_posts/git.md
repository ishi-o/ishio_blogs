---
title: "Git: 使用git进行版本管理"
date: 2024-06-12
categories: [Tools & Utilities, manager, version manager]
tags: [quick start, git]
---
<!-- placeholder -->
<!-- more -->
# 使用`Git`进行版本管理

## 介绍

`Git`是一个分布式版本管理工具，它诞生的故事很有意思->[Git的历史](<https://zhuanlan.zhihu.com/p/114067067#:~:text=[译>] Git 的历史%3A 软件版本控制的统治之路 1 Git 的历史%3A 软件版本控制的统治之路,Android 采用 ... 8 Microsoft 改变态度 ... 更多项目)
版本控制器：当我们需要修改代码但又担心无法回退时，我们需要创建副本。当修改次数多、修改规模大时，我们很难记住这些副本实现了什么功能或是修复了哪些`bug`，也不知道改动了哪里，而这只是一人修改的情况，如果是一个需要互相沟通的团队呢？因此，版本控制器应运而生，它方便我们**查看历史版本、快速回退、创建版本分支**等。总而言之，一个合格的程序员应学会熟练使用`Git`这个高效强大且开源的版本控制工具
虽然`git`在对非代码文件时无法查看不同版本间的差别，但依旧是十分好用的版本管理工具

## 基本使用

安装好`Git`之后，需要进行身份认证(但其实只需要提供信息，假的也可以)，在终端或虚拟终端中输入：

```shell
git config --global user.name "Username"
git config --global user.email "email@123.com"
```

如此就可以开始使用`Git`了！

### 创建仓库

打开项目文件夹，右键点击`Open Git Bash here`，便可以打开终端页面，或是打开本地终端`cd`到项目文件夹，输入：

```shell
git init    # 新建空仓库
git init [dir]  # 指定目录作为仓库
git clone [href]  # 克隆别人的仓库,href即资源位置
```

新建后，该文件夹内就会出现`.git`隐藏文件夹，可以进行提交等操作
`href`在`github`中，你可以在一个项目的根目录下找到一个绿色的`Code`按钮，里面就有`url`

### 提交代码

输入以下命令来提交部分或全部文件到**暂存区**：

```shell
git add file
git add .     # 提交文件夹下所有文件
git rm --cached file  # 删除暂存区指定文件
```

在暂存区内，可以撤回或添加文件，确认后，输入以下命令将它们提交到仓库：

```shell
git commit -m "explanation" # -m后为注释,表示这次修改改动的东西
```

一般来说，每次更新或修复一个功能，就需要提交一次，因此，上述命令是很常用的

### 查看日志

每次提交，就会新增一个节点，通过如下命令查看它们：

```shell
git log
```

这条命令可以查看所有历史节点详细信息如注释、修改时间、修改内容、修改人、修改行数、`commit id`等

可以加入其它参数查看更细致的内容，例如：

```shell
git log --oneline   # 每个节点只用一行表示
git log --stat   # 查看具体改动过的文件名和一些简略的信息
git log --author=[name] # 只查看名为'name'作者的提交
git log --since=[time] # 只查看'time'之后的提交
```

如果窗口大小足够，它会全部印出，否则会进入`vim`查看

### 版本回退

每个节点都有自己的一个`commit id`，即版本序列号，输入如下命令查看改动、回溯：

```shell
git diff [id]    # 查看历史版本与当前工作区的不同
git reset --hard [id]  # 版本回退
git checkout [id]   # 版本回退的另一种方式
```

如果是在`VSC`或`PyCharm`等环境下，可以在时间线里看到更清晰的改动而无需使用命令

### 版本分支

一个项目，或许有社区/专业版，或许有中文/英文版，或许有和谐/反和谐版，这种情况下，可以用到版本分支功能。分支间不具有强关联，可以有部分区别和不同的更新时间，输入以下命令查看、创建、删除版本分支：

```shell
git branch      # 查看所有分支,星号所指即当前所在分支
git branch [branch name]   # 创建名为'branch name'的分支
git checkout -b [branch name]  # 创建名为'branch name'的分支,并自动切换到该分支
git branch -d [branch name]  # 删除名为'branch name'的分支
```

一般来说，`git`自动创建的`master`分支用于发布版本，通常会创建一个`develop`分支进行新功能的撰写和`debug`，在开发和测试完成后，将`develop`分支和主分支进行**合并**：

```shell
git checkout master  # 切换到master分支
git switch [branch name] # 同样是切换分支,效果一样
git merge [branch name] # 将指定分支合并到当前分支下
```
