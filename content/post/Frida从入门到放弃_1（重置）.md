---
title: "Frida从入门到放弃_1（重置）"
created_Time: 2023-05-05 16:34:00 +0000 UTC
lastmod: 2023-08-17 17:09:00 +0000 UTC
author: "ukiko"
last_edit_author: "ukiko"
draft: false
categories: [移动安全,工具]
description: "frida 1 "
tags: [Android,Hook,原理,基础]
date: 2019-01-22T00:00:00.000+08:00
---

# Frida从入门到放弃_1（重置）

### 0x00 Frida

Frida 官网：https://www.frida.re/

github: https://github.com/frida/frida

Dynamic instrumentation toolkit for developers, reverse-engineers, and security researchers.

- Scriptable

- Portable

- Free

- Battle-tested

### 0x01 安装

### 二进制安装 (推荐)

pip install frida-tools 就一个命令搞定

> Failed to load the Frida native extension: DLL load failed: 找不到指定的模块

	报了这个错 查了大半天 原来我用的版本是基于python3.7编译的。我现在用的3.6.。。。。



绑定：二选一就行

```shell
pip install frida       # Python bindings
npm install frida       # Node.js bindings
```

### 手动编译

依赖：

`pip3 install colorama prompt-toolkit pygments`

- Linux

`make`

- MacOS and iOS

```shell
export MAC_CERTID=frida-cert
export IOS_CERTID=frida-cert
make
```

- Windows

```shell
frida.sln #VS2017
```

### 0x02 Android环境

设备：小米mix2 运行Android9.0 MIUI10开发版已解锁root

frida-server: 用的arm64版本

[server文件下载](https://github.com/frida/frida/releases)

下载好对应的 frida-server 然后adb push 进去

`adb push frida-server /data/local/tmp`

然后`chomd 755 frida-server`修改权限

运行`./frida-server`

![img](https://my-md-1253484710.file.myqcloud.com/20190112133452.png)

frida-server

这几个命令：

```powershell
adb root
adb push frida-server /data/local/tmp
adb shell "chmod 755 /data/local/tmp/frida-server"
adb shell "/data/local/tmp/frida-server &"
```

查看架构：

![img](https://my-md-1253484710.file.myqcloud.com/20190616145105.png)

aarch64

### 0x03 简单测试

命令行运行`frida-ps -U`

![img](https://my-md-1253484710.file.myqcloud.com/20190112133604.png)

frida-ps

有显示就是连接成功

接下来对浏览器进行简单测试

![img](https://my-md-1253484710.file.myqcloud.com/20190616150349.png)

`frida-trace -U -i open com.android.browser`

![img](https://my-md-1253484710.file.myqcloud.com/20190616150506.png)

随便点一下浏览器

![img](https://my-md-1253484710.file.myqcloud.com/20190616150623.png)

测试Diva

![img](https://my-md-1253484710.file.myqcloud.com/20190616151039.jpg)

![img](https://my-md-1253484710.file.myqcloud.com/20190616151307.png)

`frida-trace -U -i "open*" jakhar.aseem.diva`

![img](https://my-md-1253484710.file.myqcloud.com/20190616151432.png)

进行Hook login.class checkout函数

逆向过程略

![img](https://my-md-1253484710.file.myqcloud.com/20190616151826.png)

HOOK脚本：

```javascript
Java.perfrom(function(){
    console.log("######")
    var logActivity = Java.use("jakhar.aseem.diva.LogActivity");
    logActivity.checkout.implementation = function(){
        console.log("Hook")
    }
})
```

命令行载入脚本运行

`frida -U jakhar.aseem.diva -l diva1.js --no-pause`

进入logging关卡 点击check out，成功hook到checkout函数。

![img](https://my-md-1253484710.file.myqcloud.com/20190616153553.png)

### 0x04 总结

---

算是简单的入门了frida。

frida还有很多厉害的功能。多读读官方文档，收货会更多。

