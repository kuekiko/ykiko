---
title: "PWN 小tools的使用"
date: 2018-10-22T20:39:26+08:00
draft: false
tags: ["PWN","GCC","tools"]
categories : ["PWN","工具环境"]
description : ""
---


### GCC 编译常用命令

|    不带选项    |                                                              | gcc test.c        将test.c预处理、汇编、编译并链接形成可执行文件。这里未指定输出文件，默认输出为a.out。 |
| :------------: | ------------------------------------------------------------ | ------------------------------------------------------------ |
|       -o       | 指定生成的输出文件；                                         | gcc test.c -o test  将test.c预处理、汇编、编译并链接形成可执行文件test。-o选项用来指定输出文件的文件名。 |
|       -E       | 仅执行编译预处理；                                           | gcc -E test.c -o test.i   将test.c预处理输出test.i文件。     |
|       -S       | 将C代码转换为汇编代码；                                      | gcc -S test.i   将预处理输出文件test.i汇编成test.s文件。     |
|       -c       | 仅执行编译操作，不进行连接操作。                             | gcc -c test.s   将汇编输出文件test.s编译输出test.o文件。     |
|     -wall      | 显示警告信息；                                               |                                                              |
| **无选项链接** |                                                              | gcc test.o -o test 将编译输出文件test.o链接成最终可执行文件test。 |
|       -O       | 使用编译优化级别1编译程序。级别为1~3，级别越大优化效果越好，但编译时间越长 | gcc -O1 test.c -o test                                       |
|                | 关掉DEP/NX（堆栈不可执行）                                   | gcc  -z execstack -o level level.c                           |
|                | 关掉Stack Protector/Canary（栈保护）                         | gcc -fno-stack-protector -o level level.c                    |
|                | 关掉程序ASLR/PIE（程序随机化保护）                           | gcc -no-pie level level.c                                    |
|                | 64位linux下面的GCC编译出一个32位可执行程序                   | gcc -m32 -z execstack -fno-stack-protector -o level level.c  |



### GDB常用调试命令

```C
gcc -g  main.c                      //在目标文件加入源代码的信息
gdb a.out       

(gdb) start                         //开始调试
(gdb) n                             //一条一条执行
(gdb) step/s                        //执行下一条，如果函数进入函数
(gdb) backtrace/bt                  //查看函数调用栈帧
(gdb) info/i locals                 //查看当前栈帧局部变量
(gdb) frame/f                       //选择栈帧，再查看局部变量
(gdb) print/p                       //打印变量的值
(gdb) finish                        //运行到当前函数返回
(gdb) set var sum=0                 //修改变量值
(gdb) list/l 行号或函数名             //列出源码
(gdb) display/undisplay sum         //每次停下显示变量的值/取消跟踪
(gdb) break/b  行号或函数名           //设置断点
(gdb) continue/c                    //连续运行
(gdb) info/i breakpoints            //查看已经设置的断点
(gdb) delete breakpoints 2          //删除某个断点
(gdb) disable/enable breakpoints 3  //禁用/启用某个断点
(gdb) break 9 if sum != 0           //满足条件才激活断点
(gdb) run/r                         //重新从程序开头连续执行
(gdb) watch input[4]                //设置观察点
(gdb) info/i watchpoints            //查看设置的观察点
(gdb) x/7b input                    //打印存储器内容，b--每个字节一组，7--7组
(gdb) disassemble                   //反汇编当前函数或指定函数
(gdb) si                            // 一条指令一条指令调试 而 s 是一行一行代码
(gdb) info registers                // 显示所有寄存器的当前值
(gdb) x/20 $esp                    //查看内存中开始的20个数
ni 单步执行不进入 
si 单步执行并进入
disas addr 对地址addr处的指令进行反汇编，addr可以是函数名 
checksec 查看elf编译的保护选项。 
```
#### 查壳

> upx -d file 

#### objjump

> objdump是二进制文件快速查看工具。   常用命令：   
>
> 1. `objdump -d [file]` 查看文件的所有汇编代码   
> 2. `objdump -f [file]` 查看文件的每个文件的整体头部摘要 

####python

> 1. `python -c "..." | ./file` python以命令方式执行并把结果传递给file
> 2. `python -c "..." | xargs ./file` python以命令方式执行并当作命令行参数传递给file，具体的是：“它的作用是将参数列表转换成小块分段传递给其他命令，以避免参数列表过长的问题。”存在这个命令是因为很多的参数不支持以管道的方式传递。
> 3. `os.system()` 创建一个子进程
> 4. `os.putenv("name", "value")` 添加一个环境变量

#### pwntools

> 参考 
> <http://pwntools.readthedocs.io/en/stable/>   （官网介绍）
>
> <http://brieflyx.me/2015/python-module/pwntools-intro/>
>
> <http://brieflyx.me/2015/python-module/pwntools-advanced/>

> 一般直接用from pwn import * 或者import pwn将所有模块导入到当前命名空间，这条语句还会把os、sys等常用的系统库一并导入。
>
> 常用的模块有下面几个： 
> \- ==asm==:汇编与反汇编 
> \- ==dynelf==:用于远程符号泄露，需要提供leak方法 
> \- ==elf==:对elf文件进行操作 
> \- ==gdb==:配合gdb进行调试 
> \- ==memleak==:用于内存泄漏 
> \- ==shellcraft==: shellcode的生成器 
> \- ==tubes==:包括tubes: 包括tubes.sock, tubes.process, tubes.ssh, tubes.serialtube，分别适用于不同场景的PIPE 
> \- ==utils==:一些实用的小功能，例如CRC计算，cyclic pattern等

#### pwndbg

- arena 堆检查
- mp 显示堆
- bins,fastbins,unsorted,smallbins,largebins
- heap
- top_chunk
- procinfo 查看当前进程状态
- rop `rop --grep "pop rdi" -- --nojop --nosys --depth 2`
- search `search -s “/bin/sh”`
- vvmap 虚拟内存映射
- telescope 检查内存转储

