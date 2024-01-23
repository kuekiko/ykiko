---
title: "WSL2使用技巧"
created_Time: 2024-01-11 03:28:00 +0000 UTC
lastmod: 2024-01-23 03:49:00 +0000 UTC
author: "ukiko"
last_edit_author: "ukiko"
date: 2023-12-01T00:00:00.000+08:00
draft: false
categories: [杂项]
description: "WSL2常用的一些配置、使用技巧"
tags: [WSL2]
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

一些文档

- [How to manage WSL disk space | Microsoft Learn](https://learn.microsoft.com/en-us/windows/wsl/disk-space)

1. 创建VHD

	打开Windows磁盘管理工具。点击【操作】→ 【创建VHD】 → 选择自己需要的选项

	![img](https://prod-files-secure.s3.us-west-2.amazonaws.com/9c7f1f5e-be9f-42de-8e14-4ff6785eb454/12e0497d-3615-471d-ad62-483168b09775/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240123%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240123T035336Z&X-Amz-Expires=3600&X-Amz-Signature=f0df0b8f14faf2b671294e0b8e04ca335219eb6cab87883387be3f1681a38a12&X-Amz-SignedHeaders=host&x-id=GetObject)



1. 格式化磁盘为ext4

	- 在Windows使用

	- 在WSL2中使用`mkfs.ext4`
	
		`sudo mkfs.ext4 /mnt/d/xxx/xxx.vhd`
	
	



1. 挂载磁盘

	- 直接挂载
	
		`mkdir workspace`
	
		`sudo mount -t auto /mnt/d/workspace.vhd workspace`
	
		`sudo chmod 755 -R workspace`
	
	

	- 使用wsl命令挂载
	
		wsl --mount --vhd xxx
	
	



### 0x04 在WSL2上编译AOSP



### 0x05 使用WSLg图形界面



### 0x06 使用CAN/CANFD协议

1. 下载内核：到https://github.com/microsoft/WSL2-Linux-Kernel下载需要的内核版本

1. 安装编译内核需要的依赖

	`sudo apt install build-essential flex bison dwarves libssl-dev libelf-dev ncurses-dev`



1. 配置内核

	cp Microsoft/config-wsl .config

	`make menuconfig` → Networking support--->CAN BUS subsystem support 开启相关模块 

	对于SLCAN支持等依然需要自己手动修改.config文件开启

	也可以自己手动修改.config文件  为m或者y

	```plain text
	# 编译时添加到config-wsl文件中的配置指令
	CONFIG_CC_CAN_LINK=y
	CONFIG_CC_CAN_LINK_STATIC=y
	
	# CONFIG_HAMRADIO is not set
	CONFIG_CAN=m
	CONFIG_CAN_RAW=m
	CONFIG_CAN_BCM=m
	CONFIG_CAN_GW=m
	CONFIG_CAN_J1939=m
	CONFIG_CAN_ISOTP=m
	
	#
	# CAN Device Drivers
	#
	CONFIG_CAN_VCAN=m
	CONFIG_CAN_VXCAN=m
	CONFIG_CAN_SLCAN=m
	CONFIG_CAN_DEV=m
	CONFIG_CAN_CALC_BITTIMING=y
	CONFIG_CAN_KVASER_PCIEFD=m
	CONFIG_CAN_C_CAN=m
	# CONFIG_CAN_C_CAN_PLATFORM is not set
	# CONFIG_CAN_C_CAN_PCI is not set
	CONFIG_CAN_CC770=m
	# CONFIG_CAN_CC770_ISA is not set
	# CONFIG_CAN_CC770_PLATFORM is not set
	CONFIG_CAN_IFI_CANFD=m
	CONFIG_CAN_M_CAN=m
	# CONFIG_CAN_M_CAN_PCI is not set
	# CONFIG_CAN_M_CAN_PLATFORM is not set
	CONFIG_CAN_PEAK_PCIEFD=m
	CONFIG_CAN_SJA1000=m
	# CONFIG_CAN_EMS_PCI is not set
	# CONFIG_CAN_EMS_PCMCIA is not set
	# CONFIG_CAN_F81601 is not set
	# CONFIG_CAN_KVASER_PCI is not set
	# CONFIG_CAN_PEAK_PCI is not set
	# CONFIG_CAN_PEAK_PCMCIA is not set
	# CONFIG_CAN_PLX_PCI is not set
	# CONFIG_CAN_SJA1000_ISA is not set
	# CONFIG_CAN_SJA1000_PLATFORM is not set
	CONFIG_CAN_SOFTING=m
	CONFIG_CAN_SOFTING_CS=m
	
	#
	# CAN USB interfaces
	#
	CONFIG_CAN_8DEV_USB=m
	CONFIG_CAN_EMS_USB=m
	CONFIG_CAN_ESD_USB2=m
	CONFIG_CAN_ETAS_ES58X=m
	CONFIG_CAN_GS_USB=m
	CONFIG_CAN_KVASER_USB=m
	CONFIG_CAN_MCBA_USB=m
	CONFIG_CAN_PEAK_USB=m
	CONFIG_CAN_UCAN=m
	# end of CAN USB interfaces
	
	CONFIG_CAN_DEBUG_DEVICES=y
	# end of CAN Device Drivers
	```



1. 编译安装内核

	`make -j8`

	`sudo make modules_install`  

	`sudo make install`

	`cp arch/x86/boot/bzImage /mnt/c/Users/<User>/`

	在`/mnt/c/Users/<User>/`下创建.wslconfig文件写入以下内容：

	```plain text
	[wsl2]
	kernel=C:\\Users\\[username]\\bzImage
	```



1. 测试安装

	重启wsl2  → `wsl --shutdown [Ubuntu] ` []中是虚拟机系统 可能是kali之类的

	```elm
	# 安装can utils 工具
	sudo apt install can-utils
	
	# 使能并创建vcan0
	sudo modprobe can
	sudo modprobe can_raw
	sudo modprobe vcan 
	sudo ip link add dev vcan0 type vcan
	sudo ip link set vcan0 up
	
	# 检查vcan0是否存在，应该能看到vcan0已经存在
	ifconfig
	
	# 在一个窗口向vcan0发送随机数据
	cangen vcan0
	
	# 在另一个窗口查看vcan0是否接收到数据
	candump vcan0
	
	###能看到数据即安装正常
	```



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



