---
title: EnvRecord
---

# EnvRecord

> 用来不时记录平时所用环境配置，以及遇到的各种问题以及解决办法。主要是为了防丢失，以及总是和空气斗志斗勇。

## **Win10**

> 日常使用

### **WSL**

默认不是root 设置默认root用户。`ubuntu config --default-user root`

- on my zsh 装上

	[官方github](https://github.com/robbyrussell/oh-my-zsh)

	需要先安装[ZSH](https://github.com/robbyrussell/oh-my-zsh/wiki/Installing-ZSH)。

	安装on my zsh:

	via curl：`sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"`

	via wget: `sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"`

	然后配置喜欢的 插件+主题。`vi ~/.zshrc` 修改`plugins`属性

	必备插件git、autojump、zsh-autosuggestions。[主题](https://github.com/robbyrussell/oh-my-zsh/wiki/Themes)  经常使用ys

	



- GDB装上+pwndbg+peda+gef

	wsl的ubuntu不支持x86，所以主要只能调试x64的程序，而且可能会出现莫名其妙的错误。

	不过可以使用qemu来运行x86的程序，调试还是不行会出错。[参考](https://github.com/Microsoft/WSL/issues/2468)

	```shell
	sudo apt update
	sudo apt install qemu-user-static
	sudo update-binfmts --install i386 /usr/bin/qemu-i386-static --magic '\x7fELF\x01\x01\x01\x03\x00\x00\x00\x00\x00\x00\x00\x00\x03\x00\x03\x00\x01\x00\x00\x00' --mask '\xff\xff\xff\xff\xff\xff\xff\xfc\xff\xff\xff\xff\xff\xff\xff\xff\xf8\xff\xff\xff\xff\xff\xff\xff'
	# 运行下面的一条命令就行跑x86 不过每次打开wsl都得运行一次，很麻烦，可以写脚本自动开启。
	sudo service binfmt-support start
	
	
	```

	- pwndbg+peda+gef
	
		三个都可能用到，三个工具特性不一样。各有强项，所以三个都装，使用脚本`gdb.sh`启动选项
	
		安装pwndbg:
	
		```shell
		git clone https://github.com/pwndbg/pwndbg
		cd pwndbg
		./setup.sh
		
		
		```
	
		peda:
	
		```shell
		git clone https://github.com/longld/peda.git ~/peda
		echo "source ~/peda/peda.py" >> ~/.gdbinit
		
		
		```
	
		gef:
	
		```shell
		# via the install script
		$ wget -q -O- https://github.com/hugsy/gef/raw/master/scripts/gef.sh | sh
		
		# manually
		$ wget -O ~/.gdbinit-gef.py -q https://github.com/hugsy/gef/raw/master/gef.py
		$ echo source ~/.gdbinit-gef.py >> ~/.gdbinit
		
		
		```
	
		gdb.sh 把该文件放在`/usr/local/sbin` 参考
	
		```shell
		#!/bin/bash
		function Mode_change {
		        name=$1
		        gdbinitfile=~/.gdbinit#这个路径按照你的实际情况修改# gdbinitfile=/root/Desktop/mode#路径按照你的实际情况修改
		        peda="source ~/peda/peda.py"
		        gef="source ~/.gdbinit-gef.py"
		        pwndbg="source /home/pwndbg/gdbinit.py"
		
		        sign=$(cat $gdbinitfile | grep -n "#this place is controled by user's shell")
		#此处上面的查找内容要和你自己的保持一致
		
		        pattern=":#this place is controled by user's shell"
		        number=${sign%$pattern}
		        location=$[number+2]
		
		        parameter_add=${location}i
		        parameter_del=${location}d
		
		        message="TEST"
		
		        if [ $name -eq "1" ];then
		                sed -i "$parameter_del" $gdbinitfile
		                sed -i "$parameter_add $peda" $gdbinitfile
		                echo -e "Please enjoy the peda!\n"
		        elif [ $name -eq "2" ];then
		                sed -i "$parameter_del" $gdbinitfile
		                sed -i "$parameter_add $gef" $gdbinitfile
		                echo -e "Please enjoy the gef!\n"
		        else
		                sed -i "$parameter_del" $gdbinitfile
		                sed -i "$parameter_add $pwndbg" $gdbinitfile
		                echo -e "Please enjoy the pwndbg!\n"
		        fi
		}
		
		echo -e "Please choose one mode of GDB?\n1.peda    2.gef    3.pwndbg"
		
		read -p "Input your choice:" num
		
		if [ $num -eq "1" ];then
		        Mode_change $num
		elif [ $num -eq "2" ];then
		        Mode_change $num
		elif [ $num -eq "3" ];then
		        Mode_change $num
		else
		        echo -e "Error!\nPleasse input right number!"
		fi
		
		gdb $1 $2 $3 $4 $5 $6 $7 $8 $9#
		
		```
	
	



- r2全家桶 （逆向调试神器）

	```shell
	git clone https://github.com/radare/radare2.git
	cd radare2
	sys/install.sh#Install / Update
	
	```



- gcc arm aarch aarch64

	`sudo apt install gcc-arm-linux-gnueabi` 使用命令`arm-linux-gnueabi-gcc`

	`sudo apt install gcc-aarch64-linux-gnu` 使用命令`aarch64-linux-gnu-gcc`



- clang+llvm

	- 方法一，手动编译安装，费时费力

	- 方法二 apt
	
		完整方法这个[地址](http://apt.llvm.org/)
	
		只写ubuntu 18,04
	
		1. 编辑 /etc/apt/sources.list，添加源
	
		```shell
		# i386 not available
		deb http://apt.llvm.org/bionic/ llvm-toolchain-bionic main
		deb-src http://apt.llvm.org/bionic/ llvm-toolchain-bionic main
		# 7
		deb http://apt.llvm.org/bionic/ llvm-toolchain-bionic-7 main
		deb-src http://apt.llvm.org/bionic/ llvm-toolchain-bionic-7 main
		# 8
		deb http://apt.llvm.org/bionic/ llvm-toolchain-bionic-8 main
		deb-src http://apt.llvm.org/bionic/ llvm-toolchain-bionic-8 main
		
		
		```
	
		1. 添加证书
	
		```shell
		wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key|sudo apt-key add -
		# Fingerprint: 6084 F3CF 814B 57C1 CF12 EFD5 15CF 4D18 AF4F 7421
		
		```
	
		1. 安装 版本8
	
		```shell
		# LLVM
		apt-get install libllvm-8-ocaml-dev libllvm8 llvm-8 llvm-8-dev llvm-8-doc llvm-8-examples llvm-8-runtime
		# Clang and co
		apt-get install clang-8 clang-tools-8 clang-8-doc libclang-common-8-dev libclang-8-dev libclang1-8 clang-format-8 python-clang-8
		# libfuzzer
		apt-get install libfuzzer-8-dev
		# lldb
		apt-get install lldb-8
		# lld (linker)
		apt-get install lld-8
		# libc++
		apt-get install libc++-8-dev libc++abi-8-dev
		# OpenMP
		apt-get install libomp-8-dev
		
		
		```
	
	



- python3+pip

	`apt install python3 python3-pip、python-pip`



- python库

	- frida （hook) 同win下使用

	- pwntools (py2)

	- gmpy2 (py2-3)



- r2Frida

- Brida

### **vscode （666）**

> 主要是各种插件

- zh-ch （汉化包）

- background （右下角小萌人）

- 

### **[cmder](https://cmder.net/)**** (Win下强大的终端工具)**

- [官网](https://cmder.net/)下载安装

- 简单配置：

- wsl vim 无法使用上下左右键解决

### **py2-py3**

[官网](https://www.python.org/)找想要的包下载，同时装两个版本。

修改环境变量，日常使用py3,把py3的环境变量放在前面，去py2的安装目录复制一份python.exe 更名为python2.exe，就可以使用python2作为命令输入。

pip配置为国内源会快很多

### **Java8+jdk最新**

[官网下载安装](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

下个jdk最新版防止部分工具需要。

可能有时候还需要配置环境变量JAVA_HOME为JDK路径。

### **Golang**

也是只需[下载](https://golang.google.cn/)安装就ok.

### **NodeJS**

也是只需[下载安装](https://nodejs.org/en/)就ok. 推荐稳定版。

会默认安装npm，然而下载速度实在太慢，使用淘宝镜像[cnpm](https://npm.taobao.org/) ,用法相同

`npm install -g cnpm --registry=https://registry.npm.taobao.org`

### **git**

[官网](https://git-scm.com/)下载安装完事

### **Yarn**

[官网](https://www.yarnpkg.com/zh-Hant/)下载安装完事

### **CMake**

[下载](https://cmake.org/download/)安装ok

### **hugo (博客工具)**

[下载exe](https://github.com/gohugoio/hugo)到本地，配置环境变量

### **Haskell Stack**

[官方文档](https://docs.haskellstack.org/en/stable/README/#how-to-install)、win直接下载安装

### **StartlsBack (win下的美化工具)**

[下载安装](http://www.cnbigger.com/330.html) 配置底部透明和居中

### **Notepad++ (轻便的编辑器)**

[下载安装](https://notepad-plus-plus.org/)

### **Android SDK 配置**

- adb工具 在目录`platform-tools`

- emulator、monitor 在`tools`下

- NDK-build 在`ndk-bundle`

### **flutter (Google 跨平台框架)**

有[官网](https://flutter.dev/)了

下载[SDK](https://flutter.dev/docs/development/tools/sdk/releases#windows)->配置环境变量` flutter\bin` 。添加名为”PUB_HOSTED_URL”和”FLUTTER_STORAGE_BASE_URL”的条目。

- Andorid Studio设置

	安装插件

	- `Flutter`插件： 支持Flutter开发工作流 (运行、调试、热重载等).

	- `Dart`插件： 提供代码分析 (输入代码时进行验证、代码补全等).



- VScode设置

	安装插件

	- `flutter`



### **Genymotion+逍遥Android (Android 模拟器)**

下载安装ok

### **CUDA （N卡xxx)**

- 根据自己的显卡官网[下载包](https://developer.nvidia.com/cuda-downloads)

- 根据需求安装。

有个坑，如果为pytorch 或TensorFlow做前提 先看看这两支持的版本再安装相应的版本。

### **pytorch**

- 需求前置环境也得装好

[官方](https://pytorch.org/get-started/locally/)有很方便的安装方法 根据不同平台和环境

### **[TensorFlow](http://www.baidu.com/link?url=AOFeAQZSb4aqHWKb6uUy47eeb5qMTcf0_8xfIZfoJ8iSah2g4UsrjsXcv1lXP2NX)**

- 前置环境查官网

```shell
# GPU版本 py3
pip3 install tensorflow-gpu# stable
pip3 install tf-nightly-gpu# preview
pip3 install tensorflow-gpu==2.0.0-alpha0##TensorFlow 2.0 Alpha# CPU 版本
pip3 install --user --upgrade tensorflow


```

`tensorflow` - 仅支持 CPU 的最新稳定版（建议新手使用）

`tensorflow-gpu` - [支持 GPU](https://tensorflow.google.cn/install/gpu) 的最新稳定版（适用于 Ubuntu 和 Windows）

`tf-nightly` - 仅支持 CPU 的预览每夜版（不稳定）

`tf-nightly-gpu` - [支持 GPU](https://tensorflow.google.cn/install/gpu) 的预览每夜版（不稳定，适用于 Ubuntu 和 Windows）

`tensorflow==2.0.0-alpha0` - 仅支持 CPU 的预览 TF 2.0 Alpha 版（不稳定）

`tensorflow-gpu==2.0.0-alpha0` - [支持 GPU](https://tensorflow.google.cn/install/gpu) 的预览 TF 2.0 Alpha 版（不稳定，Ubuntu 和 Windows）

### **VMware pro （虚拟机）**

- 装MacOS记录

### **VBox （虚拟机）**

- 安装拓展包

### **Xshell、Xftp (free for Home/School)**

free 的要去[官网下载](https://www.netsarang.com/zh/free-for-home-school/) 填写信息，邮箱打开链接下载。

### **TeamViewer (远程连接)**

### **各种IDE、集成环境 只记录**

- Visual Studio 2019

- Pycharm

- IDEA

- 微信web开发工具

- phpStudy

- Android Studio (风扇~~ ~~)

### **Other**

不做记录

## **Ubuntu 18.04**

> 一般用来调代码。 大部分配置同上面WSL,只记录不做过多介绍

### **on my zsh**

### **gdb+pwndbg+peda+gef**

### **美化**

## **manjaro**

很喜欢的Linux发行版。基于**ArchLinux**，软件多，好看又好用。

## **Mac m2 没法用 放弃 等等**

![img](https://my-md-1253484710.file.myqcloud.com/20190519150336.jpg)

