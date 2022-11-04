---
title: "Frida从入门到放弃_2"
date: 2019-02-02T20:39:26+08:00
draft: false
tags: [
    "android安全","HOOK","Frida"
]
categories : ["HOOK","Android安全","工具环境"]
description : ""
---

### 0x00 前言

复习一下Android安装frida命令

```bash
adb root
adb push frida-server /data/local/tmp
adb shell "chmod 755 /data/local/tmp/frida-server"
adb shell "/data/local/tmp/frida-server &"
```

常用Frida 命令

```bash
frida-ps -U      #列出USB设备运行ing的进程
frida-ps -Ua     #列出运行中的应用
frida-ps -Uai    #列出已安装的应用列表
frida-ps -D xxx  #连接指定的设备
frida-ps -R

#在Safari中跟踪recv*和send* API
frida-trace -i "recv*" -i "send" Safari 
#在Safari中跟踪ObjC方法调用
frida-trace -m "-[NSView drawRect:]" Safari
#在iPhone上启动SnapChat，跟踪加密API调用
frida-trace -U -f com.app.testing -I "libcommonCrypto*"
#当程序启动时，frida跟踪所有open function
frida-trace -U -i open com.app.testing
```

下面通过自己创建一个Android APP来学习Frida在Android上的一般用法。

### 0x01 简单用例

Frida常用的两种启动方式：

1. python 绑定启动

```python
import frida
import sys,os

## package name
pkg_name = "com.example.xxx"
js_hook_file = "xxx.js"

def on_message(message,data):
    if message['type'] == 'send':
        print("[*] {0}".format(message['payload']))
    else:
        print(message)
## 插入js代码 or 加载js文件
jscode = ""

script = ""
process = frida.get_usb_device().attach(pkg_name)

with open(js_hook_file,"r") as f:
    script = process.create_script(f.read())

script.on('message', on_message)

script.load()
sys.stdin.read()
```

```js
console.log("Script loaded successfully ");
Java.perform(function x() {

    //hook代码
});
```

2. 直接命令加载脚本启动

```bash
frida -U -l xxx.js com.example.xxx
```

### 0x02 App源码

java层

```java

```

native层

```c

```



### 0x03 Java层hook

HOOK代码

```java

```





### 0x04 Native层hook

native层代码

```c

```



### 0x05 总结

源代码地址：

### 0x06 参考

- https://www.freebuf.com/articles/system/190565.html
- https://blog.csdn.net/jiangwei0910410003/article/details/80372118
- https://github.com/dweinstein/awesome-frida

