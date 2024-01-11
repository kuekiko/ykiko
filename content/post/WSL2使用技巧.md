---
title: "WSL2使用技巧"
created_Time: 2024-01-11 03:28:00 +0000 UTC
lastmod: 2024-01-11 06:08:00 +0000 UTC
author: "ukiko"
last_edit_author: "ukiko"
categories: [杂项]
description: "WSL2常用的一些配置、使用技巧"
tags: [WSL2]
date: 2023-12-01
draft: false
---

# WSL2使用技巧

### 0x00 常用配置

**配置.wslconfig（比较常用）**

> **.wslconfig** 用于在 WSL 2 上运行的所有已安装发行版中配置**全局设置**。

存放在目录 C:\Users\[username] 下

示例文件

```bash
# Settings apply across all Linux distros running on WSL 2
[wsl2]

# Limits VM memory to use no more than 4 GB, this can be set as whole numbers using GB or MB
memory=4GB 

# Sets the VM to use two virtual processors
processors=2

# Specify a custom Linux kernel to use with your installed distros. The default kernel used can be found at https://github.com/microsoft/WSL2-Linux-Kernel
kernel=C:\\temp\\myCustomKernel

# Sets additional kernel parameters, in this case enabling older Linux base images such as Centos 6
kernelCommandLine = vsyscall=emulate

# Sets amount of swap storage space to 8GB, default is 25% of available RAM
swap=8GB

# Sets swapfile path location, default is %USERPROFILE%\AppData\Local\Temp\swap.vhdx
swapfile=C:\\temp\\wsl-swap.vhdx

# Disable page reporting so WSL retains all allocated memory claimed from Windows and releases none back when free
pageReporting=false

# Turn on default connection to bind WSL 2 localhost to Windows localhost
localhostforwarding=true

# Disables nested virtualization
nestedVirtualization=false

# Turns on output console showing contents of dmesg when opening a WSL 2 distro for debugging
debugConsole=true

# Enable experimental features
[experimental]
sparseVhd=true
```



**配置wsl.conf**

> • **wsl.conf** 用于为在 WSL 1 或 WSL 2 上运行的每个 Linux 发行版按各个发行版配置**本地设置**。

放在发行版的/etc/下



### 0x01 自编译内核

WSL2支持使用自编译的内核运行

1. 下载Kernel → [microsoft/WSL2-Linux-Kernel: The source for the Linux kernel used in Windows Subsystem for Linux 2 (WSL2) (github.com)](https://github.com/microsoft/WSL2-Linux-Kernel)

1. 安装依赖

	`$ sudo apt install build-essential flex bison dwarves libssl-dev libelf-dev`



1. 修改需要的配置

1. 编译安装

### 0x02 使用USBIP连接USB设备

- [dorssel/usbipd-win: Windows software for sharing locally connected USB devices to other machines, including Hyper-V guests and WSL 2. (github.com)](https://github.com/dorssel/usbipd-win)

### 0x03 使用VHD解决卡顿问题



### 0x04 在WSL2上编译AOSP



### 0x05 使用WSLg图形界面



### 0x06 使用CAN/CANFD协议



### 0x07 WSL常用命令

```bash
~ > wsl --version                                                                                 01/11/2024 01:45:41 PM
WSL 版本： 2.0.9.0
内核版本： 5.15.133.1-1
WSLg 版本： 1.0.59
MSRDC 版本： 1.2.4677
Direct3D 版本： 1.611.1-81528511
DXCore 版本： 10.0.25131.1002-220531-1700.rs-onecore-base2-hyp
Windows 版本： 10.0.19045.2965
```

- 使用root作为默认用户

	wsl —



- 升级WSL2/降级为WSL

	- 



- WSL2备份/恢复

	导出：`wsl -export <Distro> <FileName>` [选项] → `wsl -export Ubuntu D:/bak/wslUbuntu.tar`
	
		选项为 -vhd  可以将指定应将分发版导出为 .vhdx 文件
	
	

	导入：`wsl -import <Distro> <InstallLocation> <FileName>` → `wsl -import Ubuntu D:/wsl2/ubuntu D:/bak/wslUbuntu.tar`



### 0x08 WSL2上使用CUDA



