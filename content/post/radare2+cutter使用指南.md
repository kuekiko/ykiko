---
title: "radare2+cutter使用指南"
date: 2019-01-02T20:39:26+08:00
draft: false
tags: ["radare2","RE"]
categories : ["RE","工具环境"]
description : "radare2+cutter使用指南"
---


### 0x00 介绍

[radare2](https://github.com/radare/radare2) 一个很实用的二进制分析和调试工具

[cutter](https://github.com/radareorg/cutter) 是r2的GUI版。

### 0x01 安装

支持的平台有如下：

> Windows (since XP), GNU/Linux, OS X, [Net|Free|Open]BSD,
> Android, iOS, OSX, QNX, Solaris, Haiku, FirefoxOS.

Linux平台下直接

```bash
git clone https://github.com/radare/radare2
cd radare2
sys/install.sh //(or sys/user.sh)
```

Windows下可以下载二进制安装包安装。官网[下载](https://www.radare.org/r/) 

Windows用户推荐使用Windows下的linux（wsl）来使用， win下更新慢。还是linux下用得舒服（方便，快捷）。

### 0x03 工具介绍

r2常用的包含有一下组件：

- rax2 用于数值转换
- rasm2  反汇编和汇编
- rabin2   查看文件格式
- radiff2 对文件进行 diff
- ragg2/ragg2­cc  开发shellcode工具
- rahash2  各种密码算法， hash算法
- radare2 整合了所有工具

使用帮助直接`-h`

- rax2

![](http://my-md-1253484710.coscd.myqcloud.com/20181123153746.png)

- rasm2  

![](http://my-md-1253484710.coscd.myqcloud.com/20181123153901.png)

![1542959275318](E:\笔记\Typora\学习日记\assets\1542959275318.png)

- rabin2  

![](http://my-md-1253484710.coscd.myqcloud.com/20181123154355.png)

eg: (`-I`)

![](http://my-md-1253484710.coscd.myqcloud.com/20181123154422.png)

- radiff2 

![](http://my-md-1253484710.coscd.myqcloud.com/20181123154448.png)

- ragg2/ragg2­cc  

![](http://my-md-1253484710.coscd.myqcloud.com/20181123154514.png)

- rahash2 

![](http://my-md-1253484710.coscd.myqcloud.com/20181123154604.png)



- radare2 (最常用) 可缩写为r2

![](http://my-md-1253484710.coscd.myqcloud.com/20181123155156.png)

### 0x04 r2 实战学习

challenge来源于：http://reversing.kr

先查看一下文件信息：

![](http://my-md-1253484710.coscd.myqcloud.com/20181123155334.png)

GUI?:

![](http://my-md-1253484710.coscd.myqcloud.com/20181123155444.png)

用r2载入，自动分析`aaa`命令：

![](http://my-md-1253484710.coscd.myqcloud.com/20181123155649.png)

`vv` 命令查看界面：

![](http://my-md-1253484710.coscd.myqcloud.com/20181123160028.png)

注意0x00401080 调用了GetDlgItemTextA

![](http://my-md-1253484710.coscd.myqcloud.com/20181123160457.png)

s 调到main函数，查看main的汇编代码：

![](http://my-md-1253484710.coscd.myqcloud.com/20181123160744.png)

![](http://my-md-1253484710.coscd.myqcloud.com/20181123160818.png)

`pdc`查看伪代码：

![](http://my-md-1253484710.coscd.myqcloud.com/20181123161145.png)

大写的`VV`命令查看图形界面 使用hijk来进行界面移动。

![](http://my-md-1253484710.coscd.myqcloud.com/20181123161242.png)

看到调用地址0x401020，s跳过去 ；发现没解析 可使用af来解析。 

![](http://my-md-1253484710.coscd.myqcloud.com/20181123162026.png)

看到GetDlgTemTextA调用：

函数调用

![](http://my-md-1253484710.coscd.myqcloud.com/20181123162157.png)



![](http://my-md-1253484710.coscd.myqcloud.com/20181123162302.png)



差不多逻辑就是一直比对字符串，从第二位开始比最后第一位

得到`Ea5yR3versing`

命令记不住或者想知道有些什么命令可以用就可以加个？号查询

### 0x05 Cutter的使用



- 多图待补



[Radare2 Book](https://legacy.gitbook.com/book/radare/radare2book/details)

