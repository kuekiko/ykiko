---
title: "Android应用安全防护和逆向分析-基础篇④"
date: 2018-07-22T14:15:38+08:00
lastmod: 2018-07-22T14:15:38+08:00
draft: false
keywords: ["Android",".so","ELF"]
description: "so文件格式解析"
tags: [
    "android安全",
    "读书笔记"
]
categories: ["读书笔记","Android安全"]
author: "Vorblock"
---

## 一、 基础篇④

### 第四章 so文件格式解析

1. ELF文件格式

   so文件->elf文件，文件格式看图（引用自@非虫）：

   ![](http://my-md-1253484710.coscd.myqcloud.com/elf_feichong.png)

2. 解析工具

   - readelf  常用命令 
     - readelf -h xxx.so 查头部信息
     - readelf -S xxx.so 查节（Section）信息
     - readelf -l xxx.so 查段（Program）信息
     - readelf -a xxx.so 查全部信息

3. 解析ELF文件

   动手解析一个elf文件  。。。

   太水 这里的内容 

   直接去看源码实现用java解析elf文件信息https://github.com/fourbrother/parse_androidso

   ELF 相关内容还是单独详细分析 单独写一篇吧

   ELF书籍《[Linux二进制分析](https://item.jd.com/12240585.html)》

   ### 总结

   加固脱壳必须掌握的知识点。

   

   
