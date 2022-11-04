---
title: "Raspberry Pi 3B+ 0安装使用"
date: 2019-06-23T01:14:22+08:00
draft: false
categories:
- ARM
- Raspberry_Pi
tags:
- Raspberry_Pi
- ARM
- 树莓派
description: Raspberry Pi 3B+ 0安装使用
---

## Raspberry Pi 3B+ 0安装使用

### 0x00 购买 

终于下手买了。打算用来学习ARM、以及一些硬件的知识。

- 3B+ 裸机
- 32G SD card
- 散热片
- 保护壳

![](https://my-md-1253484710.file.myqcloud.com/20190623005746.png)

### 0x01 安装系统

首先下载系统镜像。

官网有挺多系统可以选择，这里选择了安装[Raspbian](https://www.raspberrypi.org/downloads/) desktop最新版

之后打算装Lite版，手上没有多余的显示器。而且桌面版占用很高。

![](https://my-md-1253484710.file.myqcloud.com/20190622221745.png)

迅雷，3分钟搞定。

其次开始向SD卡中写镜像。

买的32G闪迪的高速卡，现在32G都白菜价了，想想几年前16G的死贵。

官方教程用的是[Etcher](https://etcher.io/) ，也可以用[Win32DiskImager](https://sourceforge.net/projects/win32diskimager/)。这里省事还是用Etcher。

步骤

1. 下载[etcher.io](https://etcher.io/)安装包安装Etcher](https://etcher.io/) 
2. 运行Etcher,选择镜像和sd卡

![](https://my-md-1253484710.file.myqcloud.com/20190622222523.png)

3. Flash一键搞定。

![](https://my-md-1253484710.file.myqcloud.com/20190622223043.png)

### 0x02 配置

系统安装完，开始进行配置。

![](https://my-md-1253484710.file.myqcloud.com/20190623003205.png)

先连上显示器看看

![](https://my-md-1253484710.file.myqcloud.com/20190623003758.png)

然而平时显示器还是要连笔记本，而且这分辨率好糊。所以还是配ssh和VNC连接使用吧。

- 修改源

```
# 编辑 `/etc/apt/sources.list` 文件，删除原文件所有内容，用以下内容取代：
deb http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ stretch main non-free contrib
deb-src http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ stretch main non-free contrib

# 编辑 `/etc/apt/sources.list.d/raspi.list` 文件，删除原文件所有内容，用以下内容取代：
deb http://mirrors.tuna.tsinghua.edu.cn/raspberrypi/ stretch main
```

`sudo apt update`

- 拓展SD卡

`sudo raspi-config`  -> Advanced Opt ->  A1 Expand Filesystem

![](https://my-md-1253484710.file.myqcloud.com/20190623140154.png)

- ssh 配置 

最新版系统直接想要的接口打开就OK

![](https://my-md-1253484710.file.myqcloud.com/20190623004049.png)

允许root登陆，修改`/etc/ssh/sshd.conf` 下的`PermitRootLogin yes`
`StrictModes yes` 就ok。

![](https://my-md-1253484710.file.myqcloud.com/20190623004313.png)

- VNC

win10下载[vnc客户端](https://www.realvnc.com)

RaspberryPI 命令开启server:`vncserver`

连接成功：

![](https://my-md-1253484710.file.myqcloud.com/20190623004603.png)

连接出现分辨率问题

设置分辨率：命令`sudo raspi-config`->Advanced Opt ->Resolution选择分辨率。重启就完事。

### 0x04 硬件检查

1. 系统镜像版本号

![](https://my-md-1253484710.file.myqcloud.com/20190623005614.png)

2. 板子型号：

![](https://my-md-1253484710.file.myqcloud.com/20190623005511.png)

3. **系统固件版本号**

![](https://my-md-1253484710.file.myqcloud.com/20190623005714.png)



看看硬件：

1. usb

![](https://my-md-1253484710.file.myqcloud.com/20190623010117.png)

2. cpu

![](https://my-md-1253484710.file.myqcloud.com/20190623010156.png)

说好的v8?

![](https://my-md-1253484710.file.myqcloud.com/20190623010258.png)

3. 网卡

![](https://my-md-1253484710.file.myqcloud.com/20190623010503.png)

![](https://my-md-1253484710.file.myqcloud.com/20190623010528.png)

### 0x05 总结

闲了很久，现在终于动手搞自己想搞的东西，花了一个晚上，搞完这些简单的安装配置，挺费时费力的，不过自己开心就好。最好是自己能够坚持下去，做更多有趣的事。

![12](https://as2.bitinn.net/uploads/5d/cjvmpwzcf000bx38hgr1oua5d.1080p.jpg)