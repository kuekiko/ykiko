---
title: "ARM-PWN从入门到放弃0x00-环境准备"
date: 2019-08-03T12:26:24+08:00
draft: false
categories:
- ARM
- PWN
tags:
- ARM
- PWN
description: ARM PWN从入门到放弃0x00 环境准备
---

### 0x00 前言

学PWN也有一段时间了，x86/x86_64 下算是入了个门，平时接触ARM比较多，正好以ARM架构下再更加深入的学习PWN。

会用的工具、环境：

- 在Ubuntu 18.04/WSL2
- Docker for wsl2
- 树莓派3B+
- qemu
- unicorn
- IDA
- Radare2
- GDB+gef+pwndbg+peda
- pwntools

### 0x01 环境安装

这里选用的有两种环境

1. ARM设备 树莓派

   之前618买了树莓派，派上用场了。

   把PI的官方系统换成了Ubuntu 18.04。

   安装了GCC 、gdb、gef（经测试只有gef能用），其他也没什么需要装的。

   关闭地址随机化

   ```bash
   sudo cat /proc/sys/kernel/randomize_va_space # 状态查看
   2 # 开启中
   sudo echo 0 > /proc/sys/kernel/randomize_va_space 
   bash: /proc/sys/kernel/randomize_va_space: Permission denied
   su
   echo 0 > /proc/sys/kernel/randomize_va_space
   cat /proc/sys/kernel/randomize_va_space
   0 #这就OK
   ```

2. QEMU 模拟 (WSL2)

   安装

   ```bash
   #防止又出什么乱七八糟的错误和需求，尽量都装上
   apt install gdb gdb-multiarch ##
   apt install qemu ##
   apt install gcc-arm-linux-gnueabi gcc-aarch64-linux-gnu ##gcc arm 依赖库安装
   ```

### 0x02 启动和调试

   1. 树莓派

      直接运行

      进行socat绑定端口 就OK

      ```bash
      socat tcp-listen:6666,fork exec:./binfile
      ```

   2. QEMU

      - 32bit

      ```bash
      # 本地GDB调试
      qemu-arm -g 1234 -L /usr/arm-linux-gnueabi ./binfile
      gdb-multiarch
      target remote localhost:1234
      # 绑定运行到指定端口 远程调试
      socat tcp-l:10002,fork exec:"qemu-arm  -L /usr/arm-linux-gnueabi ./binfile",reuseaddr &
      # + -g 方便调试  
      socat tcp-l:10002,fork exec:"qemu-arm -g 1234 -L /usr/arm-linux-gnueabi ./binfile",reuseaddr &
      ```

      - 64bit

      ```bash
      # 本地调试 -g 等待GDB调试
      qemu-aarch64 -g 1111 -L /usr/aarch64-linux-gnu ./file
      socat tcp-l:10002,fork exec:"qemu-aarch64  -L /usr/aarch64-linux-gnu ./binfile",reuseaddr &
      socat tcp-l:10002,fork exec:"qemu-aarch64 -g 1234 -L /usr/aarch64-linux-gnu ./binfile",reuseaddr &
      ```

### 0x03 调试过程

   在树莓派直接gdb调试就行。省略。。。

   ![img](https://my-md-1253484710.file.myqcloud.com/20190804221031.png)

   使用qemu+gdb-multiarch+插件进行调试步骤

   `qemu-aarch64 -g 10002 -L /usr/aarch64-linux-gnu ./baby_arm`

   ```bash
   gdb-multiarch ./baby_arm -q
   target remote:10002
   ```

   ![img](https://my-md-1253484710.file.myqcloud.com/20190804225625.png)

### 0x04 题目搭建

基于[ctf_xinetd](https://github.com/Eadom/ctf_xinetd)项目自己改了一个

地址：[CTF_ARM_xinetd](https://github.com/Vorblock/CTF_arm_xinetd)

### 0x05 参考链接

- https://xz.aliyun.com/t/3154
- https://github.com/Eadom/ctf_xinetd
- https://github.com/bkerler/exploit_me