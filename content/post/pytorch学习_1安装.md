---
title: "pytorch学习_1安装"
created_Time: 2023-05-05 16:35:00 +0000 UTC
lastmod: 2023-08-17 17:10:00 +0000 UTC
author: "ukiko"
last_edit_author: "ukiko"
categories: [AI]
description: "pytorch基础"
tags: [AI,pytorch]
date: 2018-12-08T00:00:00.000+08:00
draft: false
---

# pytorch学习_1安装

### 前言

本来想着用**tensorflow**的 然而GPU版总是报各种各样的BUG

所以打算入坑一下学pytorch

配置：win10+i76700HQ+GTX1060+16G

软件版本：CUDA10+python3.6+pytorch 1 源码编译

尝试1：官方安装方法不支持 CUDA 10 太坑，社区有编译通过的，所以只有自己编译试试

报各种异常，但是没停，那就等等

![img](http://my-md-1253484710.coscd.myqcloud.com/20181204002444.png)

![img](http://my-md-1253484710.coscd.myqcloud.com/20181204153247.png)

CPU被占满，巨卡。

一觉起来之后：安装失败

尝试2：等着完全支持CUDA10之后在用GPU跑吧。

妥协：用阿里云的学生服务器装了CPU的版本：顺便把TensorFlow 也给装了。。

![img](http://my-md-1253484710.coscd.myqcloud.com/20181204153129.png)

然而 在一个星期之后 pytorch1.0出来了 支持了CUDA10 nice

```plain text
pip3 install http://download.pytorch.org/whl/cu100/torch-1.0.0-cp36-cp36m-win_amd64.whl
pip3 install torchvision
```

![img](http://my-md-1253484710.coscd.myqcloud.com/20181208233904.png)

期间没有遇到任何问题 真舒畅。。。

