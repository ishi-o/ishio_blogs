---
title: "WSL2: Windows下极其轻量的Linux虚拟机"
date: 2025-05-19
categories: [tools, virtual machine, wsl]
tags: [beginner, linux, wsl]
---
<!-- placeholder -->
<!-- more -->
## `WSL2`

### 基本介绍

- `WSL`全称`Windows Subsystem for Linux`，是`2016`年发布的适用于`Windows`的`Linux`子系统
- 通常在一个电脑上，通过装虚拟机软件会比装两个系统更方便，例如安装`VMWare`或`VirtualBox`，它们能在更广泛的系统上安装其它子系统，例如在`MacOS`里安装`Windows`子系统(通过提供一个镜像文件)，而`WSL`只适用于在`Windows`里安装`WSL`支持的`Windows`子系统，且不提供`GUI`
- 但`WSL`有它独有的优势，它的速度更快、配置更简单，且启动极快
- 实现原理：
  - `wsl`本身不带`Linux`内核，仅仅是作为表示层/翻译层，将`Linux`命令翻译成`Windows`兼容的内核命令

    性能极高(因为只进行翻译)，但仅兼容`ELF`可执行文件，不支持`Docker`等
  - `wsl2`具有完整内核，是通过`Hyper-V`实现的轻量级虚拟机，同样与`Windows`深度集成，在`Linux`内部使用虚拟硬盘性能较高，但访问`Windows`文件系统(`/mnt/`)时性能较低

    兼容所有的`Linux`软件
  - `VMWare`等重量级虚拟机软件：完全模拟计算机硬件，需要完整启动`BIOS`、完整的上下文切换和硬件模拟，因此性能低，但在网络配置上自由度高

### `WSL`配置

- 安装`wsl`：一键式安装

  ```shell
  # 管理员powershell(会将wsl2作为默认版本)
  # 安装后需要重启电脑
  wsl --install
  # 重启后会安装默认的Ubuntu20发行版并要求创建新的用户名和密码
  ```

- 安装`wsl`：手动配置

  ```shell
  # 管理员powershell
  # 启用wsl
  dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
  # 启用虚拟机平台
  dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
  ```

  然后安装`WSL2 Linux kernel update package for x64 machines`

  最后在`powershell`执行启用`wsl2`

  ```shell
  wsl --set-default-version 2
  ```

- 安装`Linux`发行版：在`Microsoft Store`中搜索即可

  或通过命令行安装：

  ```shell
  # 查看Microsoft Store里的所有发行版
  wsl --list --online   # wsl -l -o
  # 安装对应发行版(实例名称为默认的发行版名称)
  wsl --install -d <Distro-name>
  ```

### `WSL`常用命令

- `wsl -l -v`/`wsl --list --verbose`：查看已安装的虚拟机实例的状态
- `wsl --set-default <Distro-name>`：设置指定的虚拟机实例为默认虚拟机
- `wsl`：启动并进入默认虚拟机实例
- `wsl --shutdown`：关闭所有虚拟机
- `wsl --unregister <Distro-name>`：删除虚拟机实例
- `wsl -d <Distro-name>`：启动并进入指定虚拟机实例
- `wsl <command> [-d <Distro-name>]`：在虚拟机实例中执行指定的命令(执行结束后退出)
- `wsl --export <Distro-name> <Filename>`：导出指定发行版为指定的文件
- `wsl --import <Distro-name> <InstallDir> <Filename>`：导入指定的归档文件为指定名称的虚拟机实例，其中`InstallDir`为该虚拟机所挂载的目录
