---
title: "Linux: 高效地监控操作系统"
date: 2025-01-11
categories: [OS, Linux, Administrator cmds]
tags: [Linux, command]
---
<!-- placeholder -->
<!-- more -->
# `Bash Shell`命令

## 进程

### 获取进程信息

跟进进程(`process`)是很重要的任务，包括查看、终止、执行者和执行时间等等...

`ps`命令就是用来获得进程信息的：

```shell
# 不带选项情况下,获得当前终端、在当前瞬间、且进程属主为当前用户的进程的:
# PID  TTY(所处终端) TIME(占用的CPU时间)  CMD(命令名)
$ ps
ps options:
 -A/-e  # 获取所有进程
 -a   # 获取除无终端进程、控制进程外的所有进程
 -f   # 获取详细信息,以Unix风格输出
 -u   # 获取详细信息,以BSD风格输出
 
```

### 监测进程信息

`ps`用于获取瞬时的进程信息，而`top`用于监测一个时段的信息：

```shell
top  # 实时获取进程信息,每过一段时间会更新
```

### 结束进程

用`kill`来杀死进程：

```shell
kill PID # 杀死PID指向的进程

pkill CMD # 杀死命令名为CMD的进程,支持正则表达式
```

## 磁盘空间
