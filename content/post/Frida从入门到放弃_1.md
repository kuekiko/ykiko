---
title: "Frida从入门到放弃_1"
date: 2019-01-22T20:39:26+08:00
draft: false
tags: [
    "android安全","HOOK","Frida"
]
categories : ["HOOK","Android安全","工具环境"]
description : "Frida安装简单介绍"
---

### 0x00 Frida

Frida 官网：https://www.frida.re/

github: https://github.com/frida/frida

Dynamic instrumentation toolkit for developers, reverse-engineers, and security
researchers.

- Scriptable
- Portable
- Free
- Battle-tested

### 0x01 安装

##### 二进制安装 (推荐)

pip install frida-tools 就一个命令搞定



> Failed to load the Frida native extension: DLL load failed: 找不到指定的模块 
>
> 报了这个错 查了大半天 原来我用的版本是基于python3.7编译的。我现在用的3.6.。。。。
>

绑定：二选一就行

```bash
pip install frida       # Python bindings
npm install frida       # Node.js bindings
```

##### 手动编译

依赖：

`pip3 install colorama prompt-toolkit pygments`

- Linux

`make`

- MacOS and iOS

```bash
export MAC_CERTID=frida-cert
export IOS_CERTID=frida-cert
make
```

- Windows 

```bash
frida.sln #VS2017
```

### 0x02 Android环境

设备：小米mix2 运行Android9.0 MIUI10开发版已解锁root

frida-server: 用的arm64版本

[server文件下载](https://github.com/frida/frida/releases)

下载好对应的 frida-server  然后adb push 进去

`adb push frida-server /data/local/tmp`

然后`chomd 755 frida-server`修改权限

运行`./frida-server`

![frida-server](https://my-md-1253484710.file.myqcloud.com/20190112133452.png)

这几个命令：

```powershell
adb root
adb push frida-server /data/local/tmp
adb shell "chmod 755 /data/local/tmp/frida-server"
adb shell "/data/local/tmp/frida-server &"
```

查看架构：

![aarch64](https://my-md-1253484710.file.myqcloud.com/20190616145105.png)



### 0x03 简单测试

命令行运行`frida-ps -U`

![frida-ps](https://my-md-1253484710.file.myqcloud.com/20190112133604.png)

有显示就是连接成功

接下来对浏览器进行简单测试

![](https://my-md-1253484710.file.myqcloud.com/20190616150349.png)



`frida-trace -U -i open com.android.browser`

![](https://my-md-1253484710.file.myqcloud.com/20190616150506.png)

随便点一下浏览器

![](https://my-md-1253484710.file.myqcloud.com/20190616150623.png)

测试Diva

![](https://my-md-1253484710.file.myqcloud.com/20190616151039.jpg)

![](https://my-md-1253484710.file.myqcloud.com/20190616151307.png)

`frida-trace -U -i "open*" jakhar.aseem.diva`

![](https://my-md-1253484710.file.myqcloud.com/20190616151432.png)

进行Hook login.class checkout函数

逆向过程略

![](https://my-md-1253484710.file.myqcloud.com/20190616151826.png)

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

![](https://my-md-1253484710.file.myqcloud.com/20190616153553.png)

### 0x04 总结

--------------

算是简单的入门了frida。

frida还有很多厉害的功能。多读读官方文档，收货会更多。







