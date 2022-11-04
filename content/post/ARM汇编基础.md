---
title: "ARM汇编基础"
date: 2018-08-10T19:34:10+08:00
lastmod: 2018-08-10T19:34:10+08:00
draft: false
tags: ["ARM"]
categories: ["ARM"]
description: "ARM汇编"
---

## ARM汇编基础(简)

经常忘记，做个笔记，好作复习。。  

内容主要来源于《Android软件安全与逆向分析》和《逆向工程权威指南》以及 [ARM 汇编](https://www.anquanke.com/post/id/86383) 和[Azeria-labs](https://azeria-labs.com/writing-arm-assembly-part-1/)

### ARM架构

ARM属于RISC CPU，

- ARM模式   4个字节opcode  32位
- Thumb模式  2个字节opcode  16位
- Thumb-2模式  同上（只是有部分4个字节的opcode)
- 64位ARM  4个字节opcode 
- ARM机器码在版本3之前是小端。但是之后默认采用大端格式，但可以设置切换到小端。 

![](http://my-md-1253484710.coscd.myqcloud.com/20180810112415.png)

### 数据类型

数据类型在汇编语言中的扩展后缀为**-h**或者**-sh**对应着半字，**-b**或者**-sb**对应着字节，但是对于字并没有对应的扩展 

![](http://my-md-1253484710.coscd.myqcloud.com/20180810113108.png)

```asm
ldr = 加载字，宽度四字节
ldrh = 加载无符号的半字，宽度两字节
ldrsh = 加载有符号的半字，宽度两字节
ldrb = 加载无符号的字节
ldrsb = 加载有符号的字节
str = 存储字，宽度四字节
strh = 存储无符号的半字，宽度两字节
strsh = 存储有符号的半字，宽度两字节
strb = 存储无符号的字节
strsb = 存储有符号的字节
```

### 字节序

在内存中有两种字节排布顺序，大端序(BE)或者小端序(LE)。两者的主要不同是对象中的每个字节在内存中的存储顺序存在差异。一般X86中是小端序，最低的字节存储在最低的地址上。在大端机中最高的字节存储在最低的地址上。 

![](https://p0.ssl.qhimg.com/t01b6d7f41b02b0a58d.png)

数据访问时采取大端序还是小端序使用程序状态寄存器(CPSR)的第9比特位来决定的。 

![](http://my-md-1253484710.coscd.myqcloud.com/20180810120701.png)



### 寄存器

37个32位寄存器，其中31个为基础寄存器，6个为状态寄存器。

用户模式下有

- 不分组寄存器（R0-R7） R7一般存放系统调用号
- 分组寄存器（R8-R14）
- 程序计数器（R15）
- 单前程序状态寄存器（CPSR）

两种状态：

| ARM状态（32位对齐） |         Thumb状态(16位对齐)          |
| :-----------------: | :----------------------------------: |
|        R0-R7        |             R0-R7(相同)              |
|        CPSR         |              CPSR（同）              |
|         R11         |            FP（栈帧指针）            |
|         R12         |          IP（内部程序调用）          |
|         R13         |             SP（栈指针）             |
|         R14         | LR（链接寄存器）一般存放函数返回地址 |
|         R15         |           PC（程序计数器）           |

和x86对比：![](https://p5.ssl.qhimg.com/t01a8e5d24fa91f9f0f.jpg)

CSPR:![](http://my-md-1253484710.coscd.myqcloud.com/20180810122226.png)

32位的CPSR寄存器的比特位含义，左边是最大比特位，右边是最小比特位。每个单元代表一个比特。

![](http://my-md-1253484710.coscd.myqcloud.com/20180810122258.png)

| 条件码 | 助记符后缀 | 标志                | 含义               |
| ------ | ---------- | ------------------- | ------------------ |
| 0000   | EQ         | Z置位               | 相等               |
| 0001   | NE         | Z清零               | 不相等             |
| 0010   | CS         | C置位               | 无符号数大于或等于 |
| 0011   | CC         | C清零               | 无符号数小于       |
| 0100   | MI         | N置位               | 负数               |
| 0101   | PL         | N清零               | 正数或零           |
| 0110   | VS         | V置位               | 溢出               |
| 0111   | VC         | V清零               | 未溢出             |
| 1000   | HI         | C置位Z清零          | 无符号数大于       |
| 1001   | LS         | C清零Z置位          | 无符号数小于或等于 |
| 1010   | GE         | N等于V              | 带符号数大于或等于 |
| 1011   | LT         | N不等于V            | 带符号数小于       |
| 1100   | GT         | Z清零且（N等于V）   | 带符号数大于       |
| 1101   | LE         | Z置位或（N不等于V） | 带符号数小于或等于 |
| 1110   | AL         | 忽略                | 无条件执行         |

### 程序结构

Android平台采用的是GUN ARM汇编格式，汇编器为GAS

参数传递：R0-R3这4个寄存器用来传递函数调用的第1到4个参数，超出的参数通过堆栈来传递。R0还用来存放函数调用的返回值。

![](http://my-md-1253484710.coscd.myqcloud.com/20180810112712.gif)

##### 汇编器指令

- `.file`:源文件名
- `.align`:代码对齐方式
- `.ascii`:声明字符串
- `.global`:声明全局符号
- `.type`：指定符号的类型
- `.word`：存放地址值
- ` .size`：设置指定符号的大小
- `.ident`：编译器标识

### 寻址方式

- 立即寻址

  `MOV R0, #1234`     ->R0=1234

- 寄存器寻址

  `MOV R1 = R2`   ->R0=R1

- 寄存器移位寻址

  - LSL ：逻辑左移，移位后寄存器空出的低位补0
  - LSR：逻辑右移，移位后寄存器空出的高位补0
  - ASR：算术右移，移位过程中符号位保持不变，若源操作数为正数，则移位后空出的高位补0，否则补1。
  - ROR：循环右移，移位后移出的低位填入移位空出的高位。
  - RRX：带扩展的循环右移，操作数右移一位，移出的空位用C标志的值填充。

  `MOV R0, R1, LSL #2 `  ->R1左移两位（R1<<2）赋值给R0,相当于R0 = R1*4

- 寄存器间接寻址

  `LDR RO, [R1]`  ->将R1寄存器的数值作为地址，取出此地址中的值赋给R0寄存器

- 基址寻址

  多用于查表、数组访问操作。

  `LDR R0, [R1,#-4]` ->将R1寄存器的数值减4作为地址，取出此地址的值赋给R0寄存器。

- 多寄存器寻址

  一条指令最多完成16个通用寄存器值的传送。

  `LDMIA R0,{R1,R2,R3,R4}` ->LDM为数据加载指令，指令的后缀IA表示每次执行完加载操作后R0寄存器的值自增1个字，ARM指令集中，子表示的是一个32位数值。这条指令作用为：R1 = [R0],R2 = [R0+#4],R3 = [R0+#8],R4 = [R0+#12]。

- 堆栈寻址

  特定的指令来完成：`LDMFA/STMFA`、`LDMEA/STMEA`、`LDMFD/STMFD`、`LDMED/STMED`。

  LDM和STM为指令前缀，表示多寄存器寻址，即一次传送多个寄存器的值。后面的后缀为指令后缀。

  `STMFD SP!, {R1-R7,LR}`  ->将R1~R7,LR入栈，多用于保存子程序的现场。

  `LDMFD SP!, {R1-R7,LR}` ->出栈，恢复现场。

- 块拷贝寻址

  实现从连续地址数据从存储器的某一位置拷贝到另外一个位置，指令有：`LDMIA/STMIA`、`LDMDA/STMDA`、`LDMIB/STMIB`、`LDMDB/STMDB`。

  `LDMIA R0! {R0-R3}`  从R0寄存器指向的存储单元中读取3个字到R1-R3寄存器

  `STMIA R0! {R0-R3}` 存储从R1-R3寄存器的内容到R0寄存器指向的存储单元

- 相对寻址

  以程序计数器PC的当前值为基地址，指令中的地址标号作为偏移量，将两者相加之后得到操作数的有效地址。

  ```asm
  BL NEXT
  	····
  NEXT:
  	········
  ```

### ARM和Thumb指令集

##### 基本指令简述

`MNEMONIC{S}{condition} {Rd}, Operand1, Operand2 `

`助记符{是否使用CPSR}{是否条件执行以及条件} {目的寄存器}, 操作符1, 操作符2 `

> MNEMONIC     - 指令的助记符如ADD 
>
> {S}                      - 可选的扩展位，如果指令后加了S，则需要依据计算结果更新CPSR寄存器中的条件跳转相关	   的FLAG  
>
> {condition}        - 如果机器码要被条件执行，那它需要满足的条件标示 
>
> {Rd}                    - 存储结果的目的寄存器 
>
> Operand1     - 第一个操作数，寄存器或者是一个立即数 
>
> Operand2     - 第二个(可变的)操作数，可以是一个立即数或者寄存器或者有偏移量的寄存器 

第二操作数还有如下操作： 

```asm
#123                    - 立即数
Rx                      - 寄存器比如R1
Rx, ASR n               - 对寄存器中的值进行算术右移n位后的值
Rx, LSL n               - 对寄存器中的值进行逻辑左移n位后的值
Rx, LSR n               - 对寄存器中的值进行逻辑右移n位后的值
Rx, ROR n               - 对寄存器中的值进行循环右移n位后的值
Rx, RRX                 - 对寄存器中的值进行带扩展的循环右移1位后的值
```

```asm
ADD   R0, R1, R2         - 将第一操作数R1的内容与第二操作数R2的内容相加，将结果存储到R0中。
ADD   R0, R1, #2         - 将第一操作数R1的内容与第二操作数一个立即数相加，将结果存到R0中
MOVLE R0, #5             - 当满足条件LE(Less and Equal,小于等于0)将第二操作数立即数5移动到R0中,注意这条指令与MOVLE R0, R0, #5相同
MOV   R0, R1, LSL #1     - 将第二操作数R1寄存器中的值逻辑左移1位后存入R0
```

![](http://my-md-1253484710.coscd.myqcloud.com/20180810144732.png)



##### 内存访问相关指令

通常，LDR被用来从内存中加载数据到寄存器，STR被用作将寄存器的值存放到内存中。 

![](http://my-md-1253484710.coscd.myqcloud.com/20180810145231.png)

例子：

```asm
.data          /* 数据段是在内存中动态创建的，所以它的在内存中的地址不可预测*/
var1: .word 3  /* 内存中的第一个变量 */
var2: .word 4  /* 内存中的第二个变量 */
.text          /* 代码段开始 */ 
.global _start
_start:
    ldr r0, adr_var1  @ 将存放var1值的地址adr_var1加载到寄存器R0中 
    ldr r1, adr_var2  @ 将存放var2值的地址adr_var2加载到寄存器R1中 
    ldr r2, [r0]      @ 将R0所指向地址中存放的0x3加载到寄存器R2中  
    str r2, [r1]      @ 将R2中的值0x3存放到R1做指向的地址 
    bkpt             
adr_var1: .word var1  /* var1的地址助记符 */
adr_var2: .word var2  /* var2的地址助记符 */
```

![](http://my-md-1253484710.coscd.myqcloud.com/20180810145711.gif)

![](http://my-md-1253484710.coscd.myqcloud.com/20180810154906.png)



- 第一种偏移形式：立即数作偏移

  ```asm
  STR    Ra, [Rb, imm]
  LDR    Ra, [Rc, imm]
  ```

  ```asm
  
  .data
  var1: .word 3
  var2: .word 4
  .text
  .global _start
  _start:
      ldr r0, adr_var1  @ 将存放var1值的地址adr_var1加载到寄存器R0中 
      ldr r1, adr_var2  @ 将存放var2值的地址adr_var2加载到寄存器R1中 
      ldr r2, [r0]      @ 将R0所指向地址中存放的0x3加载到寄存器R2中  
      str r2, [r1, #2]  @ 取址模式：基于偏移量。R2寄存器中的值0x3被存放到R1寄存器的值加2所指向地址处。
      str r2, [r1, #4]! @ 取址模式：基于索引前置修改。R2寄存器中的值0x3被存放到R1寄存器的值加4所指向地址处，之后R1寄存器中存储的值加4,也就是R1=R1+4。
      ldr r3, [r1], #4  @ 取址模式：基于索引后置修改。R3寄存器中的值是从R1寄存器的值所指向的地址中加载的，加载之后R1寄存器中存储的值加4,也就是R1=R1+4。
      bkpt
  adr_var1: .word var1
  adr_var2: .word var2
  ```

![](http://my-md-1253484710.coscd.myqcloud.com/20180810160844.gif)

- **第二种偏移形式：寄存器作偏移** 

  ```asm
  STR    Ra, [Rb, Rc]
  LDR    Ra, [Rb, Rc]
  ```

  ```asm
  .data
  var1: .word 3
  var2: .word 4
  .text
  .global _start
  _start:
      ldr r0, adr_var1  @ 将存放var1值的地址adr_var1加载到寄存器R0中 
      ldr r1, adr_var2  @ 将存放var2值的地址adr_var2加载到寄存器R1中 
      ldr r2, [r0]      @ 将R0所指向地址中存放的0x3加载到寄存器R2中  
      str r2, [r1, r2]  @ 取址模式：基于偏移量。R2寄存器中的值0x3被存放到R1寄存器的值加R2寄存器的值所指向地址处。R1寄存器不会被修改。 
      str r2, [r1, r2]! @ 取址模式：基于索引前置修改。R2寄存器中的值0x3被存放到R1寄存器的值加R2寄存器的值所指向地址处，之后R1寄存器中的值被更新,也就是R1=R1+R2。
      ldr r3, [r1], r2  @ 取址模式：基于索引后置修改。R3寄存器中的值是从R1寄存器的值所指向的地址中加载的，加载之后R1寄存器中的值被更新也就是R1=R1+R2。
      bx lr
  adr_var1: .word var1
  adr_var2: .word var2
  ```

  ![](http://my-md-1253484710.coscd.myqcloud.com/20180810162217.gif)

- **第三种偏移形式：寄存器缩放值作偏移** 

  ```asm
  LDR    Ra, [Rb, Rc, <shifter>]
  STR    Ra, [Rb, Rc, <shifter>]
  ```

  ```asm
  .data
  var1: .word 3
  var2: .word 4
  .text
  .global _start
  _start:
      ldr r0, adr_var1         @ 将存放var1值的地址adr_var1加载到寄存器R0中 
      ldr r1, adr_var2         @ 将存放var2值的地址adr_var2加载到寄存器R1中 
      ldr r2, [r0]             @ 将R0所指向地址中存放的0x3加载到寄存器R2中  
      str r2, [r1, r2, LSL#2]  @ 取址模式：基于偏移量。R2寄存器中的值0x3被存放到R1寄存器的值加(左移两位后的R2寄存器的值)所指向地址处。R1寄存器不会被修改。
      str r2, [r1, r2, LSL#2]! @ 取址模式：基于索引前置修改。R2寄存器中的值0x3被存放到R1寄存器的值加(左移两位后的R2寄存器的值)所指向地址处，之后R1寄存器中的值被更新,也就R1 = R1 + R2<<2。
      ldr r3, [r1], r2, LSL#2  @ 取址模式：基于索引后置修改。R3寄存器中的值是从R1寄存器的值所指向的地址中加载的，加载之后R1寄存器中的值被更新也就是R1 = R1 + R2<<2。
      bkpt
  adr_var1: .word var1
  adr_var2: .word var2
  ```

  ![](http://my-md-1253484710.coscd.myqcloud.com/20180810162414.png)

  如何区分取址模式：

  如果有一个叹号!，那就是索引前置取址模式，即使用计算后的地址，之后更新基址寄存器。

  如果在[]外有一个寄存器，那就是索引后置取址模式，即使用原有基址寄存器重的地址，之后再更新基址寄存器 

  除此之外，就都是偏移取址模式了 

- **关于PC相对取址的LDR指令** 

  有时候LDR并不仅仅被用来从内存中加载数据。还有如下这操作: 

  ```asm
  .section .text
  .global _start
  _start:
     ldr r0, =jump        /* 加载jump标签所在的内存位置到R0 */
     ldr r1, =0x68DB00AD  /* 加载立即数0x68DB00AD到R1 */
  jump:
     ldr r2, =511         /* 加载立即数511到R2 */ 
     bkpt
  ```

  这些指令学术上被称作伪指令。 

- **在ARM中使用立即数的规律** 

  在ARM中不能像X86那样直接将立即数加载到寄存器中。因为你使用的立即数是受限的。 

  立即数的值：v = n ror 2*r 有效的立即数都可以通过循环右移来得到 

  ```asm
  #256        // 1 循环右移 24位 --> 256
  #384        // 6 循环右移 26位 --> 384
  #484        // 121 循环右移 30位 --> 484
  #16384      // 1 循环右移 18位 --> 16384
  #2030043136 // 121 循环右移 8位 --> 2030043136
  #0x06000000 // 6 循环右移 8位 --> 100663296 (十六进制值0x06000000)
  Invalid values:
  #370        // 185 循环右移 31位 --> 31不在范围内 (0 – 30)
  #511        // 1 1111 1111 --> 比特模型不符合
  #0x06010000 // 1 1000 0001.. --> 比特模型不符合
  ```

  这样并不能一次性加载所有的32位值。不过我们可以通过以下的两个选项来解决这个问题： 

  - 用小部分去组成更大的值。    MOV r0, #511  将511分成两部分：MOV r0, #256, and ADD r0, #255 

  - ```asm
    .section .text
    .global _start
    _start:
     mov r0, #256   /* 1 ror 24 = 256, so it's valid */
     add r0, #255   /* 255 ror 0 = 255, valid. r0 = 256 + 255 = 511 */
     ldr r1, =511   /* load 511 from the literal pool using LDR */
     bkpt
    ```

    

    计算立即数的有效值脚本：https://raw.githubusercontent.com/azeria-labs/rotator/master/rotator.py

    ```bash
    azeria@labs:~$ python rotator.py
    Enter the value you want to check: 511
    Sorry, 511 cannot be used as an immediate number and has to be split.
    azeria@labs:~$ python rotator.py
    Enter the value you want to check: 256
    The number 256 can be used as a valid immediate number.
    1 ror 24 --> 256
    ```

  下面的部分指令用到在详细查，记的话脑壳痛

  ##### 跳转指令   

  1. `B`
  2. `BL`
  3. `BX`
  4. `BXL`

  ##### 存储器操作指令

  1. `LDM`
  2. `STM`
  3. `PUSH`
  4. `POP`
  5. `SWP`

  ##### 数据处理

  1. `MOV`
  2. `MVN`
  3. `ADD`
  4. `ADC`
  5. `SUB`
  6. `RSB`
  7. `SBC`
  8. `RSC`
  9. `MUL`
  10. `MLS`
  11. `MLA`
  12. `UMULL`
  13. `UMLAL`
  14. `SMUULL`
  15. `SMLAL`
  16. `SMLAD`
  17. `SMLSD`
  18. `SDIV`
  19. `UDIV`
  20. `ASR`
  21. `AND`
  22. `ORR`
  23. `EOR`
  24. `BIC`
  25. `LSL`
  26. `LSR`
  27. `RRX`
  28. `ROR`
  29. `CMP`
  30. `CMN`
  31. `TSL`
  32. `TEQ`

  ##### 其他指令

  1. `SWI`

  2. `NOP`

  3. `MRS`

  4. `MSR`

      

      

  

  ### 

  

  