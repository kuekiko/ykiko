---
title: "Qemu+gdb调试Linux内核"
date: 2019-08-28T23:24:18+08:00
draft: false
categories:
- Linux内核
tags:
- Qemu
- Linux内核
description: Qemu+gdb调试Linux内核
---

### 前言


调试Linux内核可以使用VM双机调试，不过使用qemu来调试会更加方便。


### 环境搭建

#### 编译源码


首先到Linux [FTP仓库](https://mirrors.edge.kernel.org/pub/linux/kernel/v4.x/)或者[官网](https://www.kernel.org/)下载对应版本的源码。

这里下载的是`linux-4.10.10`


解压`tar -xvJf linux-4.10.10.tar.xz`


安装依赖

```bash
    sudo apt install build-essential ncurses-dev xz-utils libssl-dev bc fakeroot aptitude libncurses5-dev

​    sudo apt install qemu
```


#### 编译内核


```bash
    make menuconfig
```


![](https://my-md.oss-cn-shenzhen.aliyuncs.com/20190828152106.png)



进行配置：`KernelHacking` —>  `Compile-time checks and compiler options`选中

```
Compile the kernel with debug info
```

```
Compile the kernel with frame pointers
```

```
Provide GDB scripts for kernel debugging
```

```
Processor type and features→去掉Paravirtualized guest support
```

保存退出。

命令`make -jN` 进行编译

之后`make all`

```
make modules
```

编译完成之后，`vmlinux`在源码根目录、`bzImage`在`./arch/x86/boot/`下

#### 构建initramfs根文件系统

借助BusyBox构建极简initramfs，busybox最新版[下载地址](https://busybox.net/downloads/)

编译静态版Busybox 

```
make menuconfig
```

![](https://my-md.oss-cn-shenzhen.aliyuncs.com/20190828160608.png)



设置以下选项：

Settings -> Build Options -> Build static binary (no shared libs) 编译成静态文件

开始编译：

```bash
make -jN
make install 
```

等待编译完成源码目录下出现`_install`目录，进行配置：

```bash
    cd _install

​    mkdir proc sys dev etc etc/init.d

​    vim etc/init.d/rcS

​    \# 文件中的内容如下

​    \# #!/bin/sh

​    \# mount -t proc none /proc

​    \# mount -t sysfs none /sys

​    \# /sbin/mdev -s

​    chmod +x etc/init.d/rcS
```

创建文件系统

```bash
find . | cpio -o --format=newc > ../rootfs.img
```

#### 运行内核

```bash
    qemu-system-x86_64 \

​    -kernel ~/linux-4.10.10/arch/x86_64/boot/bzImage \

​    -initrd ~/busybox-1.31.0/rootfs.img \

​    -append "console=ttyS0 root=/dev/ram rdinit=/sbin/init" \

​    -cpu kvm64,+smep,+smap \

​    -nographic \

​    -gdb tcp::1234
```

**`-**cpu kvm64,**+**smep,**+**smap` 设置CPU的安全选项，这里开启了smap和smep

**`-**kernel` 设置内核bzImage文件的路径

**`-**initrd` 设置刚才利用busybox创建的rootfs.img，作为内核启动的文件系统

**`-**gdb tcp::1234` 设置gdb的调试端口为1234 在GDB中通过 target remote localhist:1234进行连接

#### 驱动

`insmod` 加载驱动

`rmmod` 卸载驱动

`lsmod` 查看加载了的驱动

### 调试测试

qemu启动，启动后的界面

![](https://my-md.oss-cn-shenzhen.aliyuncs.com/20190828162012.png)

启动GDB

![](https://my-md.oss-cn-shenzhen.aliyuncs.com/20190828164008.png)



![](https://my-md.oss-cn-shenzhen.aliyuncs.com/20190828164103.png)



使用内核提供的GDB辅助调试功能：(gdb)apropos lx

调试内核模块：add-symbol-file 添加模块文件

断点测试 `b cmdline_proc_show`

`cat /proc/cmdline` 触发断点

### 引用