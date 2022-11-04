---
title: "Android应用安全防护和逆向分析 基础篇③"
date: 2018-07-16T13:51:50+08:00
lastmod: 2018-07-16T13:51:50+08:00
draft: false
keywords: ["命令"]
description: "Android中开发与逆向常用命令总结"
tags: [
    "android安全",
    "读书笔记"
]
categories: ["读书笔记","Android安全"]
author: "Vorblock"


---

## 一、 基础篇③

### 第三章 Android中开发与逆向常用命令总结

#### 1. 基础命令

##### 1.1 cat命令

​	查看文件内容  结合grep进行过滤

##### 1.2 echo/touch命令

​	写文件   配个定向符使用

#### 2. 非shell命令

##### 2.1 adb shell dumpsys sctivity top

​	说明：查看当前应用的activity信息

​	用法：运行需要查看的应用

![](http://my-md-1253484710.coscd.myqcloud.com/android-note-3-1.png)

​	如果直接运行	` adb shell dmpsys`会把当前系统中的所有应用运行的四大组件都打印出来 内容非常多 使用信息重定向来进行选择：可借助Windows的`start`命令

![](http://my-md-1253484710.coscd.myqcloud.com/android-note-3-2.png)

##### 2.2 adb shell dumpsys package

​	说明：查看指定包名应用的详细信息 （相当于AndroidManifest.xml的内容）

​	用法：adb shell dumpsys package [pkgname]

![](http://my-md-1253484710.coscd.myqcloud.com/android-note-3-3.png)

##### 2.3 adb shell dumpsys meminfo

​	说明：查看指定进程名或者进程id的内存信息

​	用法：adb shell dumpsys meminfo [pname/pid]

![](http://my-md-1253484710.coscd.myqcloud.com/adnroid-note-3-4.png)

​	和后面的top命令结合使用 可以分析应用的性能消耗情况

##### 2.4 adb shell dump dbnfo

​	说明：查看指定包名应用的数据库存储信息（包括存储的SQL语句）

​	用法：adb shell dump dbnfo [packagename]

![](http://my-md-1253484710.coscd.myqcloud.com/20180716123331.png)

##### 2.5 adb install 

​	说明：安装应用包apk文件

​	用法：adb install [apk文件]

​	升级安装 使用apk install -r [apk文件]

##### 2.6 adb uninstall 

​	卸载

##### 2.7 adb pull 

​	从设备复制到本地

​	adb pull 设备目录 本地目录

##### 2.8 adb push 

​	从本地复制到设备

​	同上

##### 2.9 adb shell screencap

​	截屏

​	adb shell scteencap -p 截图文件目录

​	快速截取手机屏幕

``` bat
adb shell screencap -p /sdcard/tmp.png
adb pull /sdcard/tmp.png D:\
start D:\tmp.png
```

##### 2.10 adb shell screenrecord

​	录屏

​	adb shell screenrecord 路径

##### 2.11 adb shell input text

​	输入文本内容

​	adb shell input text [需要输入文本框的内容]

​	eg: 让输入内容的文本框回去焦点

​	` adb shell input text 'hello world'`

​	这个命令可以模拟物理键盘、虚拟键盘、滑动、滚动等事件。

##### 2.12 adb forward 

​	端口转发命令

​	adb forward  [远程协议：端口号]· [设备协议：端口号 ]

​	eg: adb forward tcp:23946 tcp:23946  IDA 调试 

​	      adb  forwrd tcp:8700 jwdp:1786

##### 2.13 adb jdwp

​	查看设备中可以被调试的应用的进程号

​	adb jdwp 

##### 2.14 adb logcat 

​	查看当前的日志信息

​	adb logcat -s tag    eg：` adb logcat -s fb`

​	adb logcat | findstr pname/pip/keyword

​	adb logcat | findstr 包

​	日志信息过滤



#### 3. shell 命令

这儿shell命令是指先运行adb shell 再执行命令 与非shell命令互通

##### 3.1 run-as

​	在非root设备中查看指定debug模式的包名应用沙盒数据

​	run-as [package name]

#####  3.2 ps 

​	查看设备进程信息或者指定进程的线程信息

​	ps | grep 过滤内容

​	ps -t [pid] 查看pid 对应的线程信息

​	

##### 3.3 pm clear

​	清空指定包名的应用数据

​	pm clear [packagename]

##### 3.4 pm install 

​	安装设备中的apk 同adb install

##### 3.5 pm uninstall

​	卸载

##### 3.6 am start

​	启动一个应用

​	am start -n [packname]/[packname].[activity name]

​	am start -D -n    (以debug方式启动)

##### 3.7 am startservice

​	启动一个服务

​	am startservice -n [packagename]/[package name].[service name]

##### 3.8 am broadcast 

​	发送一个广播

​	am broadcast -a [广播动作]

##### 3.9 netcfg

​	查看设备的ip地址

##### 3.10 netstat

​	查看设备的端口号信息

##### 3.11 app_process

​	运行java代码

​	app_process [运行代码目录]· [运行主类]

​	eg:

​	`export CLASSPATH = /data/demo.jar`

​	`exec /system/bin/app_process /data/cn.wdasdkl.Main`

##### 3.12 dalvikvm

​	运行dex文件

​	dalvikvm -cp [dex文件] · [运行主类]

​	差不多同上的用处

##### 3.13 top

​	查看当前应用CPU的消耗信息。

​	top [-n/-m/-d/-s/-t]

​	-m 最多显示多少个进程

​	-n 刷新次数

​	-d 刷新时间间隔

​	-s 按那一列排序

​	-t 显示线程信息而不是进程

##### 3.14 getprop

​	查看系统属性

​	getprop [属性值名称]

​	eg：`getprop ro.debuggable` 

​	查看设备的信息

#### 4 操作apk 命令

##### 4.1 用aapt 命令操作apk命令

​	查看apk中的信息以及编辑apk程序包

​	aapt dump xmltree [apk包] · [需要查看的资源文件]

​	eg：`aapt dump xmltree demp.apk AndroidManifest.xml`

##### 4.2 用dexdump 操作dex 命令

​	查看dex的详细信息

​	dexdump [dex文件路径]

#### 5 进程命令

	##### 5.1 查看当前进程的内存加载情况

​	cat  proc/[pid]/maps   查看当前进程的内存映射信息，比如加载了那些so文件，dex文件等等。

![](http://my-md-1253484710.coscd.myqcloud.com/20180716134420.png)



##### 5.2 查看进程的状态信息

​	cat /proc/[pid]/status

![](http://my-md-1253484710.coscd.myqcloud.com/20180716134609.png)

##### 5.3 查看当前应用使用的端口号信息

​	cat /proc /[pid] / net /     tcp/ tcp6 /udp /udp6

![](http://my-md-1253484710.coscd.myqcloud.com/20180716134826.png)



### 总结

​	这章就是一些会用到的命令，后面的学习必不可少的知识点。
