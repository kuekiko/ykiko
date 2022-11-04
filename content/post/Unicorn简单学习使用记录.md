---
title: "Unicorn简单学习使用记录"
date: 2020-04-17T19:01:19+08:00
draft: false
categories:
- Unicorn
tags:
- Unicorn
- fuzzing
description: Unicorn简单学习使用记录，小工具
---

### 0x00 介绍

Unicorn是一个轻量级, 多平台, 多架构的CPU模拟器框架，基于qemu开发，它可以代替CPU模拟代码的执行，常用于恶意代码分析，Fuzz等。

### 0x01 安装

1. 官网有编译好不同平台的[二进制包](http://www.unicorn-engine.org/download/)，直接安装就行。但是看了一下时间2017年的，qemu已经升级好几个版本了。api也比较老旧。

2. 去github看了一下是否有在更新，发现一直有在更新，不过好像也没更新啥，只是修复一些bug，下载git源码，自己编译。目前最新1.0.2rc

   ``` bash
   git clone https://github.com/unicorn-engine/unicorn.git
   cd unicorn
   # UNICORN_ARCHS="arm aarch64 x86" ./make.sh # 设置要编译的架构 可选 默认6种全编译 (Arm, Arm64, M68K, Mips, Sparc, & X86)
   ./make.sh
   sudo ./make.sh install
   
   # python 绑定
   cd bindings/python
   sudo python3 setup.py install
   ```

   还有更多交叉编译选项参考 [文档](https://github.com/unicorn-engine/unicorn/blob/master/docs/COMPILE-NIX.md)

### 0x02 简单使用

C :

```c
#include <stdio.h>
#include <unicorn/unicorn.h>

// 要模拟的指令
#define X86_CODE32 "\x41\x4a" // INC ecx; DEC edx

// 起始地址
#define ADDRESS 0x1000000

int main(int argc, char const *argv[]){
  // 设置engine
  uc_engine *uc;
  uc_err err;

  //设置寄存器
  int r_ecx = 0x1234;
  int r_edx = 0x5678;
  printf("Emulate i386 code\n");

  // x86 32bit 初始化
  err = uc_open(UC_ARCH_X86, UC_MODE_32, &uc);
  if (err != UC_ERR_OK){
    printf("Failed on uc_open() with error returned: %u\n", err);
    return -1;
  }
  // 给模拟器申请 2MB 内存
  uc_mem_map(uc, ADDRESS, 2 * 1024 * 1024, UC_PROT_ALL);

  // 将要模拟的指令写入内存
  if (uc_mem_write(uc, ADDRESS, X86_CODE32, sizeof(X86_CODE32) - 1)){
    printf("Failed to write emulation code to memory, quit!\n");
    return -1;
  }
  // 初始化寄存器
  uc_reg_write(uc, UC_X86_REG_ECX, &r_ecx);
  uc_reg_write(uc, UC_X86_REG_EDX, &r_edx);
  printf(">>> ECX = 0x%x\n", r_ecx);
  printf(">>> EDX = 0x%x\n", r_edx);

  // 模拟代码
  err = uc_emu_start(uc, ADDRESS, ADDRESS + sizeof(X86_CODE32) - 1, 0, 0);
  if (err){
    printf("Failed on uc_emu_start() with error returned %u: %s\n",
           err, uc_strerror(err));
  }
  // 打印寄存器值
  printf("Emulation done. Below is the CPU context\n");

  uc_reg_read(uc, UC_X86_REG_ECX, &r_ecx);
  uc_reg_read(uc, UC_X86_REG_EDX, &r_edx);
  printf(">>> ECX = 0x%x\n", r_ecx);
  printf(">>> EDX = 0x%x\n", r_edx);

  uc_close(uc);

  return 0;
}
```



![](https://my-md-1253484710.file.myqcloud.com/20200411000429.png)

官方给了很多测试[案例](https://github.com/unicorn-engine/unicorn/tree/master/samples)

python3:

```python
from __future__ import print_function
from unicorn import *
from unicorn.x86_const import *

X86_CODE32 = b"\x41\x4a" #INC ecx; DEC edx

ADDRESS = 0x1000000

print("Emulate i386 code")

try:
    ## 初始化模拟器为x86 32bit
    mu = Uc(UC_ARCH_X86,UC_MODE_32)
    # mu = Uc(UC_ARCH_ARM64,UC_MODE_64)
    ## 2MB 的memory
    mu.mem_map(ADDRESS,2*1024*1024)
    ## 
    mu.mem_write(ADDRESS,X86_CODE32)
    ##
    mu.reg_write(UC_X86_REG_ECX,0x1234)
    mu.reg_write(UC_X86_REG_EDX,0x7890)
    ##
    mu.emu_start(ADDRESS, ADDRESS + len(X86_CODE32))

    print("Emulation done. Below is the CPU context")

    r_ecx = mu.reg_read(UC_X86_REG_ECX)
    r_edx = mu.reg_read(UC_X86_REG_EDX)
    print(">>> ECX = 0x%x" %r_ecx)
    print(">>> EDX = 0x%x" %r_edx)

except UcError as e:
    print("ERROR: %s" %e)
```



### 0x03 一些基于unicorn的项目简单使用

#### [AndroidNativeEmu](https://github.com/P4nda0s/AndroidNativeEmu)

AndroidNativeEmu 让你能够跨平台模拟Android Native库函数，比如JNI_OnLoad、Java_XXX_XX等函数

这是看雪无名侠大佬二次修改一个版本，真的太强了。



1. 安装报错解决

   安装有些坑。。

   首先要求使用python3.7。之后

   ``` bash
   git https://github.com/AeonLucid/AndroidNativeEmu.git
   pip install -r requirements.txt 
   cd samples
   # 运行python example.py开始报错。。
   ```

   在win上先搞定`keystone-engine`

   ``` bash
git clone https://github.com/keystone-engine/keystone
   cd keystone/bindings/python
   python setup.py install
   ```
   
   [下载](http://www.keystone-engine.org/download/) `Windows - Core engine` x64

   解压找到`keystone.dll` 放到`X:\location_to_python\Lib\site-packages\keystone\`

   还有可能报`fail to load the dynamic library.`

   下载 [vcredist_x64](https://www.microsoft.com/en-gb/download/details.aspx?id=40784) 安装。

   之后直接`python example.py` 继续报`ModuleNotFoundError: No module named 'androidemu'`

   加入代码

   ``` python
import sys
   sys.path.append('../')
   ······
   ```
   
   又报`FileNotFoundError: [Errno 2] No such file or directory: 	'samples/example_binaries/libc.so'`

   删掉代码里的 `samples/example_binaries/libc.so`的`samples/`

   能跑起来但是报错`unicorn.unicorn.UcError: Invalid instruction (UC_ERR_INSN_INVALID)`

   改成`emulator.load_library("example_binaries/libc.so", do_init=False)`

   就成功运行

   ![](https://my-md-1253484710.file.myqcloud.com/20200413002530.png)

2. 使用

```python
import logging
import posixpath
import sys
sys.path.append('../')
from unicorn import UcError, UC_HOOK_CODE, UC_HOOK_MEM_UNMAPPED
from unicorn.arm_const import *

from androidemu.emulator import Emulator
from androidemu.java.java_class_def import JavaClassDef
from androidemu.java.java_method_def import java_method_def
import debug_utils

class MainActivity(metaclass=JavaClassDef, jvm_name='com/example/debugdemo/MainActivity'):

    def __init__(self):
        pass

    @java_method_def(name='stringFromJNI', signature='()Ljava/lang/String;', native=True)
    def string_from_jni(self, mu):
        pass

    def test(self):
        pass

# Configure logging
logging.basicConfig(
    stream=sys.stdout,
    level=logging.DEBUG,
    format="%(asctime)s %(levelname)7s %(name)34s | %(message)s"
)
logger = logging.getLogger(__name__)

## 先创建java class

## 初始化emulater
emulator = Emulator(
  vfs_root=posixpath.join(posixpath.dirname(__file__),"vfs"),
  vfp_inst_set=True
)

## 注册 java class
emulator.java_classloader.add_class(MainActivity)

## load libraries
emulator.load_library("example_binaries/libdl.so")
emulator.load_library("example_binaries/libc.so")
emulator.load_library("example_binaries/libstdc++.so")
emulator.load_library("example_binaries/libm.so")

tar_lib = emulator.load_library("example_binaries/libtest.so")

# Show loaded modules.
logger.info("Loaded modules:")

for module in emulator.modules:
    logger.info("=> 0x%08x - %s" % (module.base, module.filename))


try:
    ## Run JNI_OnLoad.
    emulator.call_symbol(tar_lib, 'JNI_OnLoad', emulator.java_vm.address_ptr, 0x00)
    emulator.mu.hook_add(UC_HOOK_MEM_UNMAPPED, debug_utils.hook_unmapped)

    # Do native stuff.
    emulator.mu.hook_add(UC_HOOK_CODE, debug_utils.hook_code)
    main_activity = MainActivity()
    logger.info("Response from JNI call: %s" % main_activity.string_from_jni(emulator))

    # Dump natives found.
    logger.info("Exited EMU.")
    logger.info("Native methods registered to MainActivity:")

    for method in MainActivity.jvm_methods.values():
        if method.native:
            logger.info("- [0x%08x] %s - %s" % (method.native_addr, method.name, method.signature))

except UcError as e:
    print("Exit at %x" % emulator.mu.reg_read(UC_ARM_REG_PC))
    raise
```

- samples 文件夹下几个具体的实例

#### [unicorefuzz](https://github.com/fgsect/unicorefuzz)

合并到[AFL系列总结]()

#### 参考

- 中文文档https://github.com/kabeor/Micro-Unicorn-Engine-API-Documentation
- https://github.com/unicorn-engine/unicorn
- https://github.com/P4nda0s/AndroidNativeEmu
- https://app.yinxiang.com/fx/a6cc6633-a93f-4111-a06a-cccd5fa39e0f
- https://github.com/fgsect/unicorefuzz