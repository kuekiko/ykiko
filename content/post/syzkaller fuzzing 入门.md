---
title: "Syzkaller fuzzing 入门"
date: 2020-04-10T18:59:00+08:00
draft: false
categories:
- Fuzzing
tags:
- Syzkaller
- Fuzzing
description: 简单学习使用Syzkaller
---

- ### 0x00 介绍

  [syzkaller](https://github.com/google/syzkaller)是google的安全研究人员开发并维护的内核fuzz工具。它主要是用go写的，也有少部分C代码，支持akaros/fuchsia/linux/android/freebsd/netbsd/openbsd/windows等系统，发现的漏洞多达上千。

  ### 0x01 环境配置

  **环境要求**：

  - C/C++ 编译器
    
    - GCC 6.1.0+
  - linux kernel
    - 编译 v4.6以后编译时确保`CONFIG_KCOV=y` 之前版本：[这样](https://github.com/torvalds/linux/commit/5c9a8750a6409c63a0f01d51a9024861022f6593)添加支持
    - 一些额外[选项](https://github.com/google/syzkaller/blob/master/docs/linux/kernel_configs.md)
  - VM 一般QEMU
    - 支持QEMU、kvmtool和GCE虚拟机、Android设备和Odroid C2开发板
    - 需要进行通信：vm要提供网络支持
    - vm配置需要ssh服务器
    - 要能执行`ssh -i  $SSHID -p $PORT root@localhost`
    - 需要将`debugfs` 挂在到 `/sys/kernel/debug`

  - [Golang](https://golang.org/) 安装

    ```bash
    wget https://dl.google.com/go/go1.14.1.linux-amd64.tar.gz
    tar -C ~/goroot -xzf go1.14.1.linux-amd64.tar.gz
    vim /etc/profile
    mkdir /usr/local/gopath
    # 添加export 
    export GOROOT=/home/kuekiko/goroot
    export GOPATH=/home/kuekiko/gopath
    export PATH=$GOROOT/bin:$PATH
    export PATH=$GOPATH/bin:$PATH
    source etc/profile
    ```

  - syzkaller

    ```bash
    # build
    go get -u -d github.com/google/syzkaller/...
    cd $GOROOT/src/github.com/google/syzkaller/
    make -j4
    # build之后在bin/下
    ```

    - 如果要cross-OS/arch 进行测试的话，记得修改`TARGETOS`, `TARGETVMARCH` 和`TARGETARCH`参数再`make`

  

  ### 0x02 Init syzkaller

  生成镜像

  ```bash
  sudo apt install debootstrap
  cd gopath/src/github.com/google/syzkaller/tools/
  # 使用国内源 修改create-image.sh
  sudo debootstrap --include=$PREINSTALL_PKGS --components=main,contrib,non-free $RELEASE $DIR
  # 改成
  sudo debootstrap --include=$PREINSTALL_PKGS --components=main,contrib,non-free $RELEASE $DIR http://mirrors.163.com/debian/
  ./create-image.sh 
  # 选项 --distribution wheezy --feature full
  # 生成了stretch.id_rsa stretch.id_rsa.pub stretch.img
  ```

  ##### 编译启动 内核

  ```bash
  export KERNEL=/home/kuekiko/linux_kernel
  export IMG=/home/kuekiko/linux_kernel/img
  export PATH=$KERNEL:$PATH
  export PATH=$IMG:$PATH
  # 默认编译
  cd $KERNEL/xxx
  make CC="$GCC/bin/gcc" defconfig
  make CC="$GCC/bin/gcc" kvmconfig
  # 直接 make menuconfig 
  # 编辑.config
  CONFIG_KCOV=y
  CONFIG_DEBUG_INFO=y
  CONFIG_KASAN=y
  CONFIG_KASAN_INLINE=y
  # 新
  CONFIG_CONFIGFS_FS=y
  CONFIG_SECURITYFS=y
  
  make CC="$GCC/bin/gcc" olddefconfig
  make CC="$GCC/bin/gcc" -j4
  ```

  ##### qemu

  ```bash
  qemu-system-x86_64 \
    -kernel $KERNEL/arch/x86/boot/bzImage \
    -append "console=ttyS0 root=/dev/sda earlyprintk=serial"\
    -hda $IMAGE/stretch.img \
    -net user,hostfwd=tcp::10021-:22 -net nic \
    -enable-kvm \
    -nographic \
    -m 2G \
    -smp 2 \
    -pidfile vm.pid \
    2>&1 | tee vm.log
  ```

  ssh连接

  `ssh -i $IMG/stretch.id_rsa -p 10021 -o "StrictHostKeyChecking no" root@localhost`

  ![](https://my-md-1253484710.file.myqcloud.com/20200417165654.png)

  kill qemu 使用`kill $(cat vm.pid)`

  ### 0x03 Start Fuzzing

  添加配置文件my.cfg

  ```json
  {
  	"target": "linux/amd64",
  	"http": "127.0.0.1:56741",
  	"workdir": "$GOPATH/src/github.com/google/syzkaller/workdir",
  	"kernel_obj": "$KERNEL",
  	"image": "$IMAGE/stretch.img",
  	"sshkey": "$IMAGE/stretch.id_rsa",
  	"syzkaller": "$GOPATH/src/github.com/google/syzkaller",
  	"procs": 4,
  	"type": "qemu",
  	"vm": {
  		"count": 4,
  		"kernel": "$KERNEL/arch/x86/boot/bzImage",
  		"cpu": 2,
  		"mem": 2048
  	}
  }
  ```

  启动fuzzing

  `./bin/syz-manager -config=my.cfg`

  ![](https://my-md-1253484710.file.myqcloud.com/20200417185540.png)

  浏览器打开http://127.0.0.1:56741/

  ![](https://my-md-1253484710.file.myqcloud.com/20200417185612.png)

  虚拟机性能太垃圾，太慢了 fuzzing半天没也啥crashes。还是得用服务器。

  ![](https://my-md-1253484710.file.myqcloud.com/20200417190242.png)

  挂机了大概3小时，出了几个没啥用的crach。

  ![](https://my-md-1253484710.file.myqcloud.com/20200417205533.png)

  ### 0x04 总结

  还有很多选项可以开启，没试。

  看看实现源码，还有就是之后试试Fuzzing Android

  #### 参考

  - https://github.com/google/syzkaller
  - https://www.freebuf.com/sectool/142969.html
  - [http://blog.douluodalu.wang/2020/03/22/syz-fuzz%E5%88%9D%E6%8E%A2/](http://blog.douluodalu.wang/2020/03/22/syz-fuzz初探/)
  - https://github.com/google/syzkaller/blob/master/docs/linux/setup.md