---
title: "OLLVM 混淆之一"
date: 2018-09-10T20:39:26+08:00
draft: false
tags: ["android安全","OLLVM"]
categories : ["Android安全","LLVM"]
description : ""
---




### OLLVM

OLLVM(Obfuscator-LLVM)是瑞士西北应用科技大学安全实验室于2010年6月份发起的一个针对LLVM代码混淆项目， 用于增加逆向难度，保护代码的安全。最新版本为[4.0](https://github.com/obfuscator-llvm/obfuscator/tree/llvm-4.0)。OLLVM适用LLVM支持的所有语言（C, C++, Objective-C, Ada 和 Fortran）和目标平台（x86, x86-64, PowerPC, PowerPC-64, ARM, Thumb, SPARC, Alpha, CellSPU, MIPS, MSP430, SystemZ, 和 XCore）。

- [LLVM](http://llvm.org/)是lowlevel virtual machine的简称，是一个编译器框架。详细介绍可以看[WIKI-LLVM](https://zh.wikipedia.org/wiki/LLVM)

![](http://my-md-1253484710.coscd.myqcloud.com/20180823112535.png)

经典的三段式设计，前端使用不同的编译工具对代码进行分析转换成LLVM的中间表示IR（intermediate representation）。中间部分优化器只对IR进行操作，通过一系列的Pass对IR做优化。后端主要是讲优化好的IR解释成对应的机器码。

对IR的处理过程下图：

![IR Pass](http://my-md-1253484710.coscd.myqcloud.com/20180823113111.png)

OLLVM的混淆操作在IR层，通过编写Pass来混淆IR，以致后端生成的目标代码也被混淆了。

### OLLVM-Android环境搭建

前提环境：

- NDK环境
- LLVM

下载源码(包括了LLVM和Clang)-编译OLLVM步骤如下：

> ```bash
> $ git clone -b llvm-4.0 https://github.com/obfuscator-llvm/obfuscator.git
> $ mkdir build
> $ cd build
> $ cmake -DCMAKE_BUILD_TYPE=Release ../obfuscator/
> //（cmake -G "MinGW Makefiles" -DCMAKE_BUILD_TYPE=Release ../obfuscator/）(windows)
> $ make -j7   //这个数字自己看自己CPU填 太小可能非常编译慢 
> ```

可以参照[官方wiki](https://github.com/obfuscator-llvm/obfuscator/wiki)来操作。编译完成后，二进制文件放在build/bin目录下。

配置整合NDK：

1. 打开NDK目录ndk-bundle下的toolchains，新建obfuscator-llvm-4，将llvm文件夹里的所有文件复制到新建的目录下。

2. 将`\build\bin`目录下的`clang.exe`、`clang++.exe`和`clang-format.exe`复制到`\toolchains\llvm\prebuilt\windows-x86_64\bin`目录下，直接替换掉其中的文件。（Windows下）

3. （linux下)将llvm目录下的prebuilt目录和文件 config.mk、setup.mk和setup-common.mk拷贝到创建的obfuscator-llvm目录下->然后替换obfuscator-llvm/prebuilt/linux-x86下的bin和lib为我们编译好的bin和lib

4. 然后将下面文件复制一份，改名称如下，比如arm-linux-androideabi-clang3.4复制一行改名为arm-linux-androideabi-obfuscator3.4

   arm-linux-androideabi-clang3.4-> arm-linux-androideabi-obfuscator3.4

   mipsel-linux-android-clang3.4-> mipsel-linux-android-obfuscator3.4

   x86-clang3.4-> x86-obfuscator3.4

5. 分别修改以上三个文件的 setup.mk 中的 LLVM_NAME ，即将其指定到开始建立的obfuscator-llvm-3.4目录，也就是把把`LLVM_NAME := llvm-$(LLVM_VERSION)改成LLVM_NAME := obfuscator-llvm-$(LLVM_VERSION)`

6. 如果是配置64位的ndk配置,还要额外修改$NDK_PATH/build/core/setup-toolchain.mk文件，在NDK_64BIT_TOOLCHAIN_LIST := 加入 obfuscator 对应的NDK_TOOLCHAIN_VERSION NDK_64BIT_TOOLCHAIN_LIST := obfuscator3.4 clang3.6 clang3.5 clang3.4 4.9'

### 开始使用OLLVM



### 参考

- http://www.freebuf.com/articles/terminal/130142.html
- https://geneblue.github.io/2016/10/09/%E5%88%A9%E7%94%A8OLLVM%E6%B7%B7%E6%B7%86Android%20Native%E4%BB%A3%E7%A0%81%E7%AF%87%E4%B8%80/
- https://www.jmpoep.com/thread-4016-1-1.html
- BCTFhttp://ele7enxxh.com/Bctf-2016-LostFlower-Writeup.html