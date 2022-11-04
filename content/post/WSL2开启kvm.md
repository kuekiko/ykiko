---
title: "WSL2开启kvm"
date: 2020-07-24T22:47:12+08:00
draft: false
categories:
- qemu
tags:
- wsl2
- kvm
- qemu
description: 最新版win10 wsl2 开启kvm的方法
---

### 0x00 前言

之前想着用wsl2去跑qemu，发现很多内核pwn题给的环境需要开启`-enable-kvm`选项，还有自己编译的内核也没法，然而wsl2没法开启kvm，解决办法就是去把hype-v给关了，老老实实用vmware/vmbox开个虚拟机，这样每次都要切来切去很麻烦。

直到看到vmware和vmbox都可以与hype-v并存，以为看到了曙光，没想到是没法开启虚拟化选项的。

后来看到WSL上这个[Issues](https://github.com/microsoft/WSL/issues/4193)，当时也没这么多评论，不过暂且能用[脚本](https://gist.github.com/offlinehacker/b1d96515f87a47bd0b0bea574eab5583)实现。不过还是很麻烦。

最后看了这篇[文章](https://boxofcables.dev/accelerated-kvm-guests-on-wsl-2/)，跟着试了一遍，一直没报错，最后还是不行。某天又去看了看issuse，发现这个办法要最新阅览版。更新后一个新的发现就是wsl2速度明显变快了。。

### 0x01 环境

- Windows 10 2004 20175.1000

- Windows Feature Experience Pack 120.18201.0.0
- wsl2 - Ubuntu 20
- Intel  i7-8550U
- WSL-Linux-kernel: [4.19.121-microsoft-standard ](https://github.com/microsoft/WSL2-Linux-Kernel/releases/tag/4.19.121-microsoft-standard)
- qemu 5.0

### 0x02 编译Kernel

``` bash
sudo apt -y install build-essential libncurses-dev bison flex libssl-dev libelf-dev cpu-checker qemu-kvm
tar -xf WSL2-Linux-Kernel-4.19.121-microsoft-standard.tar.gz
cd WSL2-Linux-Kernel-4.19.104-microsoft-standard/
## cp Microsoft/config-wsl .config 有bug 使用最新版commit已修复
make menuconfig
```

`.config`文件用[这个](https://github.com/microsoft/WSL/files/4830946/config.txt) 或者[这个](https://github.com/microhobby/WSL2-Linux-Kernel/blob/5e3fa9a98ea1ac05b397e0acd5bf08ce0e60bd3e/Microsoft/config-wsl) 

确保里面的选项kvm的选项。

![](https://my-md-1253484710.file.myqcloud.com/20200724222001.png)

这里可选的编译为< M > 模块或者< * > 内置，建议内置吗模块的话后续每次重启都得自己加载一次。

确保Linux guest support enable选项。

![](https://my-md-1253484710.file.myqcloud.com/20200724221933.png)

退出保存。

``` bash
make -j8
```

waitting............

```Bash
cp arch/x86/boot/bzImage /mnt/c/Users/<username>/bzImage
nano /mnt/c/Users/<username>/.wslconfig
### 如果是 <M> 模块编译
sudo make modules_install
```

``` Toml
[wsl2]
nestedVirtualization=true
kernel=C:\\Users\\<username>\\bzImage
debugConsole=true  ##可选
pageReporting=true ##可选
```

```
wsl.exe --shutdown Ubuntu
wsl.exe -d Ubuntu
nano /etc/modprobe.d/kvm-nested.conf
```

mod加载的话还要写入：

``` bash
options kvm-intel nested=1
options kvm-intel enable_shadow_vmcs=1
options kvm-intel enable_apicv=1
options kvm-intel ept=1
```

之前模块编译的有bug版本的图 mount无法挂载。

![](https://my-md-1253484710.file.myqcloud.com/20200724214713.png)

成功的图 可以挂载。。无需手动加载mod

![](https://my-md-1253484710.file.myqcloud.com/20200724223913.png)

要先配置界面以及GPU支持可以参考这篇[文章](https://boxofcables.dev/accelerated-kvm-guests-on-wsl-2/)后续的部分，没这个需求。。

### issuse

还是会遇到奇奇怪怪的坑。

`An error occurred mounting one of your file systems. Please run 'dmesg' for more details.`

报错信息：

``` bash
[    2.268879] init: (1) ERROR: MountPlan9WithRetry:282: mount drvfs on /mnt/c (cache=mmap,noatime,msize=262144,trans=virtio,aname=drvfs;path=C:\;uid=0;gid=0;symlinkroot=/mnt/
[    2.268880] ) failed: 22
[    2.277772] 9pnet: Could not find request transport: virtio
[    2.280962] init: (1) ERROR: MountPlan9WithRetry:282: mount drvfs on /mnt/d (cache=mmap,noatime,msize=262144,trans=virtio,aname=drvfs;path=D:\;uid=0;gid=0;symlinkroot=/mnt/
[    2.280964] ) failed: 22
[    2.305491] init: (8) ERROR: CreateProcessParseCommon:874: Failed to translate D:\openSRC\WSL2-Linux-Kernel-4.19.121-microsoft-standard
[    2.312584] init: (8) ERROR: UtilTranslatePathList:2624: Failed to translate D:\Life_Tools\Sys_Tools\cmder\bin
[    2.319445] init: (8) ERROR: UtilTranslatePathList:2624: Failed to translate D:\Life_Tools\Sys_Tools\cmder\vendor\bin
```

这是内核的问题，很坑的地方。是官网没切换新的config文件造成的，换成新的重新编译即可。具体看着这个[issuse](https://github.com/microsoft/WSL/issues/5481)  [修复](https://github.com/microsoft/WSL2-Linux-Kernel/pull/146/commits/5e3fa9a98ea1ac05b397e0acd5bf08ce0e60bd3e)

重新编译内核。



#### 参考

- https://boxofcables.dev/accelerated-kvm-guests-on-wsl-2/
- https://github.com/microsoft/WSL/issues/4193
