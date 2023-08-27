---
title: "QUME的安装使用(补充)"
created_Time: 2023-05-05 16:35:00 +0000 UTC
lastmod: 2023-08-17 17:14:00 +0000 UTC
author: "ukiko"
last_edit_author: "ukiko"
date: 2018-08-09T00:00:00.000+08:00
draft: false
categories: [工具]
description: "QEMU的安装使用"
tags: [qemu]
---

# QUME的安装使用(补充)

# QEMU的安装使用

### 安装

WIndows：https://qemu.weilnetz.de/w64/ 下载exe安装就行

MACOS:`brew install qemu` or `sudo port install qemu`

LINUX：

- **Arch:** `pacman -S qemu`

- **Debian/Ubuntu:** `apt-get install qemu`

- **Fedora:** `dnf install @virtualization`

- **Gentoo:** `emerge --ask app-emulation/qemu`

- **RHEL/CentOS:** `yum install qemu-kvm`

- **SUSE:** `zypper install qemu`

源码安装：https://download.qemu.org/

wget

```plain text
wget https://download.qemu.org/qemu-3.0.0-rc1.tar.xz
tar xvJf qemu-3.0.0-rc1.tar.xz
cd qemu-3.0.0-rc1
./configure
make
```

git

```plain text
git clone git://git.qemu.org/qemu.git
cd qemu
git submodule init
git submodule update --recursive
./configure
make
```

最新的开发发生在主分支上。稳定的树位于名为“稳定x”的分支中。YY分支,X。YY是发布版本。

### 树莓派内核制作（在windows上)

下载树莓派系统：http://downloads.raspberrypi.org/raspbian/images/

下载qume 的树莓派内核： [https://github.com/dhruvvyas90/qemu-rpi-kernel](https://github.com/dhruvvyas90/qemu-rpi-kernel) 改名为kernel-qemu放在和系统镜像同目录下

放在了raspbia目录下

`qemu-system-arm.exe -kernel kernel-qemu -cpu arm1176 -m 512 -M versatilepb -dtbversatile-pb.dtb -no-reboot -append "root=/dev/sda2 panic=1rootfstype=ext4 rw" -net nic -net user,hostfwd=tcp::5022-:22 -hda 2018-06-27-raspbian-stretch.img`

注意自己下载的镜像版本

### Linux上

待补充。。。

