---
title: "syzkaller-android"
created_Time: 2023-05-05 16:35:00 +0000 UTC
lastmod: 2023-08-17 17:24:00 +0000 UTC
author: "ukiko"
last_edit_author: "ukiko"
date: 2019-09-15T00:00:00.000+08:00
draft: false
categories: [Fuzzing,二进制安全]
description: "Syzkaller fuzzing Android 食用教程"
tags: [PWN,Fuzzing,Syzkaller,Android,基础]
---

# syzkaller-android

### 0x00 前言

介绍使用syzkaller fuzz Android的配置教程。

### 0x01 环境

按要安装好环境

- go

- syzkaller

- 交叉编译aarch64-linux-android、g++-aarch64-linux-gnu、gcc-arm-linux-gnueabihf、g++-arm-linux-gnueabihf

### 0x02 配置

创建配置文件android.cfg

```json
{
	"target": "linux/arm64",  // arm
	"http": "127.0.0.1:56741",
	"workdir": "$GOPATH/src/github.com/google/syzkaller/workdir",
	"kernel_obj": "$KERNEL", //  内核路径
	"syzkaller": "$GOPATH/src/github.com/google/syzkaller",
	"sandbox": none,
	"procs": 8,
	"type": "adb",
	"cover": true,
	"vm": {
		"devices": [$DEVICES],
		"battery_check": true
	}
}
```

- KASAN+KCOV编译内核

	1. 首先编译要fuzz的Android版本输入手机。（过程省略）[1]

	1. 下载编译基准内核。[2] ，使用手动编译会报错，使用给KASCAN文档[3]的方法也可能报错。
	
		```shell
		## 创建文件夹
		mkdir android-kernel && cd android-kernel
		## 切换分支
		repo init -u https://android.googlesource.com/kernel/manifest -b BRANCH
		repo sync
		## 修改内核build文件 内核路径下的build.config制定make_deconfig
		KERNEL_DIR=private/msm-google
		. ${ROOT_DIR}/${KERNEL_DIR}/build.config.common ##这里
		## 构建内核
		build/build.sh
		```
	
	

	1. 刷入内核
	
		重新编译aosp启动镜像
	
		在服务器上编译的将out/dist目录打包备份
	
		```shell
		cd {aosp_dir}
		cp {kernel_dir}/out/{version}/dist/Image.lz4-dtb {aosp}/device/google/marlin-kernel
		. build/envsetup.sh
		lunch aosp_sailfish-userdebug 
		make -j64
		## 重新拷贝到本地 win
		adb reboot bootloader
		set ANDROID_PRODUCT_OUT=./
		fastboot flashall -w
		```
	
		> 之前 3.18.137-g72a7a6
		
			之后 … (手机显示无法显示内核)
		
			```shell
			sailfish:/ # uname -a
			Linux localhost 3.18.137-g8b62de70252d #1 SMP PREEMPT 2019-09-27 02:13:04 aarch64
			```
		
		
	
	

	1. 修改内核重新刷入
	
		- 失败的方法
	
		> 按文档说的
		
			```shell
			cd arch/arm64/configs
			cp marlin_defconfig marlin-kasan_defconfig
			```
		
			在配置文档中移除`CONFIG_KERNEL_LZ4=y`
		
			加入
		
			```plain text
			CONFIG_KASAN_INLINE=y
			CONFIG_KCOV=y
			CONFIG_SLUB=y
			CONFIG_SLUB_DEBUG=y
			```
		
			重新编译内核。
		
			发现任何变化
		
		
	
		> 尝试的方法
		
			修改build.config文件
		
			```shell
			. ${ROOT_DIR}/${KERNEL_DIR}/build.config.kasan (失败)
			```
		
			不修改，还是去修改marlin_defconfig添加KASAN
		
			报错：savedefconfig does not match private/msm-google
		
			修改build.config 删掉check_defconfig 检查 （失败）
		
		
	
		> 不修改文件
		
			使用命令
		
			```shell
			./build/build.sh BUILD_CONFIG=build.config.kasan
			```
		
		
	
		最后发现
	
		```shell
		cp private/msm-google/build.config.kasan ./
		BUILD_CONFIG=build.config.kasan build/build.sh ##放前面才管用
		```
	
		编译完成 ，修改aosp参数
	
		```shell
		aosp$ cd device/google/marlin/sailfish
		cp BoardConfig.mk BoardConfig.mk.bak
		vim BoardConfig.mk
		```
	
		```shell
		## 注释掉
		BOARD_KERNEL_BASE        := 0x80000000
		BOARD_KERNEL_PAGESIZE    := 4096
		## ifneq ($(filter sailfish_kasan, $(TARGET_PRODUCT)),)
		BOARD_KERNEL_OFFSET      := 0x80000
		BOARD_KERNEL_TAGS_OFFSET := 0x02500000
		BOARD_RAMDISK_OFFSET     := 0x02700000
		BOARD_MKBOOTIMG_ARGS     := --kernel_offset $(BOARD_KERNEL_OFFSET) --ramdisk_offset $(BOARD_RAMDISK_OFFSET) --tags_offset $(BOARD_KERNEL_TAGS_OFFSET)
		## else
		## BOARD_KERNEL_TAGS_OFFSET := 0x02000000
		## BOARD_RAMDISK_OFFSET     := 0x02200000
		## endif
		TARGET_KERNEL_ARCH := arm64
		TARGET_KERNEL_HEADER_ARCH := arm64
		```
	
		修改device/google/marlin/device-common.mk
	
		```shell
		lz4->gz
		LOCAL_KERNEL := device/google/marlin-kernel/Image.gz-dtb
		```
	
		新的问题
	
		```shell
		error: out/target/product/sailfish/boot-debug.img too large
		```
	
		解决方法：将BoardConfig.mk的`BOARD_BOOTIMAGE_PARTITION_SIZE`的值改大。
	
		重新编译aosp启动镜像 再刷入手机。
	
	



### 0x03 开启fuzzing

```shell
./bin/syz-manager -config=android.cfg
```

![img](https://my-md-1253484710.file.myqcloud.com/20200916163028.png)

fuzz

手机不断重启。。。。

查看msg:`adb shell dmesg -w`

`syz-manager -debug` 查看syz的相关问题。

### 参考

- [1]https://source.android.google.cn/setup/build?hl=zh-cn

- [2]https://source.android.google.cn/setup/build/building-kernels?hl=zh-cn

- [3]https://source.android.com/devices/tech/debug/kasan-kcov

- [4]https://github.com/google/syzkaller/blob/master/docs/linux/setup_linux-host_android-device_arm-kernel.md

