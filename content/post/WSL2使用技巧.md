---
title: "WSL2使用技巧"
created_Time: 2024-01-11 03:28:00 +0000 UTC
lastmod: 2024-01-11 03:52:00 +0000 UTC
author: "ukiko"
last_edit_author: "ukiko"
tags: [WSL2]
date: 2023-12-01
draft: false
categories: [杂项]
description: "WSL2常用的一些配置、使用技巧"
---

# WSL2使用技巧

### 0x00 常用配置

**配置.wslconfig（比较常用）**

> **.wslconfig** 用于在 WSL 2 上运行的所有已安装发行版中配置**全局设置**。

存放在目录 C:\Users\[username] 下





**配置wsl.conf**

> • **wsl.conf** 用于为在 WSL 1 或 WSL 2 上运行的每个 Linux 发行版按各个发行版配置**本地设置**。





[microsoft/WSL2-Linux-Kernel: The source for the Linux kernel used in Windows Subsystem for Linux 2 (WSL2) (github.com)](https://github.com/microsoft/WSL2-Linux-Kernel)



### 0x01 自编译内核

WSL2支持使用自编译的内核运行

1. 下载Kernel → [microsoft/WSL2-Linux-Kernel: The source for the Linux kernel used in Windows Subsystem for Linux 2 (WSL2) (github.com)](https://github.com/microsoft/WSL2-Linux-Kernel)

1. 安装依赖

	`$ sudo apt install build-essential flex bison dwarves libssl-dev libelf-dev`



1. 修改需要的配置

1. 编译安装

### 0x02 使用USBIP连接USB设备



### 0x03 使用VHD解决卡顿问题



### 0x04 在WSL2上编译AOSP



### 0x05 使用WSLg图形界面



### 0x06 使用CAN/CANFD协议



