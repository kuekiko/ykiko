---
title: "Android-ARM进阶"
date: 2018-10-10T16:40:20+08:00
draft: false
keywords: ["ARM"]
description: "ARM"
tags: ["ARM"]
categories: ["ARM","Android安全"]
author: "Vorblock"
---

学习一些关于ARM的汇编结构特点，以及分析。理解一些结构最好的方法就是多去尝试动手做。。

### NDK-Build的使用

可以参考[官方文档](https://developer.android.com/ndk/guides/ndk-build?hl=zh-cn)。

1. 创建一个Android项目

2. cd 项目目录

3. <ndk>/ndk-build   。也可以将NDK-build加入环境变量。

4. 创建jni文件夹，添加 Android.mk和 Application.mk两个文件。（参考官方文档）

   ```mk
   //Android.mk
   LOCAL_PATH := $(call my-dir)
   include $(CLEAR_VARS)
   
   # 要生成的.so库名称
   LOCAL_MODULE := hello
   # c++文件
   LOCAL_SRC_FILES := hello.cpp
   include $(BUILD_SHARED_LIBRARY)
   ```

   ```mk
   //Application.mk
   APP_PLATFORM := android-17
   # APP_ABI := all
   APP_ABI :=armeabi-v7a arm64-v8a
   ```

   添加hello.cpp：

   ```cpp
   #include<cstdio>
   int i,j;
   int num[] = {1,2,3,4,5};
   int main()
   {
       /* code */
       printf("hello,world!\n");
       for(i=0;i<5;i++){
           printf("num value is %d\n",num[i]);
       }
       return 0;
   }
   ```

   

5. `ndk-build`

   ![](http://my-md-1253484710.coscd.myqcloud.com/20180814152220.png)

6. push 到Android设备运行

   这里ARM32位出现里非法引用（Illegal instruction）。。之后再试试

   改成ARM64之后又出现内存区段错误“Segmentation fault”  有毒呀。。可能是哪里设置有问题。。

### arm-linux-gcc交叉编译器编译 

arm-linux-gcc也能编译出ARM可执行文件。`sudo apt install g++-arm-linux-gnueabihf `

或者下载[二进制](https://developer.arm.com/open-source/gnu-toolchain/gnu-rm/downloads)文件安装。

`arm-linux-gnueabihf-g++  -static  helloworld.cpp `

push进Android之后运行成功

![](http://my-md-1253484710.coscd.myqcloud.com/20180814164353.png)



使用这个方法和用NDK-build编译的有差异。

### for循环	

待添加

### if-else

待添加

### while

### switch

### 优化

###C++

### JNI API分析