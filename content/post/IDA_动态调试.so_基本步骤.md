---
title: "IDA_动态调试.so_基本步骤"
created_Time: 2023-05-05 16:34:00 +0000 UTC
lastmod: 2023-08-17 17:09:00 +0000 UTC
author: "ukiko"
last_edit_author: "ukiko"
date: 2018-08-31T00:00:00.000+08:00
draft: false
categories: [二进制安全,移动安全]
description: "使用IDA调试Android so的方法"
tags: [Android,基础,原理,IDA]
---

# IDA_动态调试.so_基本步骤

## IDA 动态调试.so 基本步骤

- 待补图

### 0x00 IDA快捷键

- Shirt+F12 字符串窗口

- F5大法好 反汇编

- Ctrl+S 查看so对应段的信息（非调试），快速定位so文件的内存地址（Debug）

- G 快速跳转到对应地址。s

- 调试-F7单步进入调试、F8单步、F9运行

### 0x01 方法一

1. 获取运行Android_server。

	android_server文件放在IDA安装目录下的注意版本的不同。

	之后只需 `push android_server /data/local/tmp/`。

	之后`adb shell`，`su` ，`cd /data/local/tmp/`。

	可能还得`chmod 755 android_server` 才有权限运行。



1. 建立通信、attach进程。

	`adb forward tcp:23946 tcp:23946`命令。

	在IDA的Debugger选项中attach进程。



1. 加载so、找函数下断点

	双开IDA ，Ctrl+S找到so文件的基地址，另外一个IDA找到函数的相对地址。相加得到绝对地址。



### 0x02 方法二

无法加载so文件需要在加载之前断点。反调试之类

1. Debug方式启动app。需要应用可调试开启

	`adb shell am start -D -n 包名/.MainActivity`



1. 方法一的1，2两步 勾选选项。

1. jdb attach程序

	`jdb -connect com.sun.jdi.SocketAttach:hostname=127.0.0.1,port=8700`



1. 开始调试 同上

