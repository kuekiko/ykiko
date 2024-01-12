---
title: "Jackalope使用和基本原理"
created_Time: 2024-01-11 08:19:00 +0000 UTC
lastmod: 2024-01-12 02:23:00 +0000 UTC
author: "ukiko"
last_edit_author: "ukiko"
categories: [Fuzzing,基础原理,工具]
description: ""
tags: [Fuzzing]
date: 2023-10-31T11:00:00.000+08:00
draft: false
---

# Jackalope使用和基本原理

## 0x00 前言



## 0x01 编译运行

### Windows

这里是Windows10环境 + Visual Studio 2022

```bash
git clone https://github.com/googleprojectzero/Jackalope.git
cd Jackalope
git clone --recurse-submodules https://github.com/googleprojectzero/TinyInst.git
mkdir build
cd build
cmake -G "Visual Studio 16 2022" -A x64 .. ## 报错则需要使用x64 Native Tools Command or vcvars64.bat / vcvars32.bat
cmake --build . --config Release
```

使用Visual Studio 2022会报错，需要进行下列操作

1. patch代码

	找到D:\xxx\Jackalope\TinyInst\third_party\mbuild\mbuild\build_env.py文件

	在if os.environ['VisualStudioVersion']  in ['15.0','16.0']:处加上17.0：if os.environ['VisualStudioVersion']  in ['15.0','16.0','17.0']:



1. 打开x64 Native Tools Command Prompt for VS 2022 命令行窗口

1. `cd xxx\Jackalope\build
cmake -G "Visual Studio 17 2022" -A x64 ..
cmake --build . --config Release`

测试运行

编译完的程序在xx\Jackalope\build\Release 下面，有一个fuzzer.exe和test.exe

创建一个seed进行测试  新建文件夹in，在里面新建1.txt 内容为111

运行以下命令开始测试，几分钟后出现了几个崩溃。

```powershell
fuzzer.exe -in in -out out -t 1000 -delivery shmem -instrument_module test.exe -target_module test.exe -target_method fuzz -nargs 1 -iterations 10000 -persist -loop -cmp_coverage -- test.exe -m @@

```

### Linux

使用gcc报错

```shell
CMake Error at CMakeLists.txt:127 (message):
  You need to use Clang to compile TinyInst on Linux
```

使用clang 

```bash
git clone https://github.com/googleprojectzero/Jackalope.git
cd Jackalope
git clone --recurse-submodules git@github.com:googleprojectzero/TinyInst.git
mkdir build && cd build
CC=clang CXX=clang++ cmake ..
cmake --build . --config Release
```

clang14可能报错

```shell
/usr/bin/ld: libtinyinst.a(debugger.cpp.o): in function `Debugger::CreateSharedMemory(unsigned long)':
debugger.cpp:(.text+0x1f64): undefined reference to `shm_open'
/usr/bin/ld: libtinyinst.a(debugger.cpp.o): in function `Debugger::ClearSharedMemory()':
debugger.cpp:(.text+0x238f): undefined reference to `shm_unlink'
/usr/bin/ld: libtinyinst.a(debugger.cpp.o): in function `Debugger::Init(int, char**)':
debugger.cpp:(.text+0x7bb6): undefined reference to `pthread_create'
clang-14: error: linker command failed with exit code 1 (use -v to see invocation)
make[2]: *** [TinyInst/CMakeFiles/litecov.dir/build.make:86: TinyInst/litecov] Error 1
make[1]: *** [CMakeFiles/Makefile2:256: TinyInst/CMakeFiles/litecov.dir/all] Error 2
make: *** [Makefile:84: all] Error 2
```

解决办法：修改TinyInst\CMakeLists.txt文件，将末尾几行改成

```makefile
project("sslhook")

add_executable(sslhook
  sslhook-main.cpp
  sslhook.h
  sslhook.cpp
)
find_package(Threads REQUIRED)
target_link_libraries(sslhook tinyinst rt Threads::Threads)

project("litecov")

add_executable(litecov
  tinyinst-coverage.cpp
)
find_package(Threads REQUIRED)
target_link_libraries(litecov tinyinst rt Threads::Threads)
```



### Android

```bash
mkdir build-android
cd build-android
cmake -DCMAKE_TOOLCHAIN_FILE=/path/to/android/ndk/build/cmake/android.toolchain.cmake -DANDROID_NDK=/path/to/android/ndk -DANDROID_ABI=arm64-v8a -DANDROID_PLATFORM=<platform> ..
cmake --build . --config Release
```

## 0x02 基础命令







## 0x03 源码解读

