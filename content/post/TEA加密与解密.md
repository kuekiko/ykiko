---
title: "TEA加密与解密"
created_Time: 2023-05-05 16:02:00 +0000 UTC
lastmod: 2023-08-20 15:55:00 +0000 UTC
author: "ukiko"
last_edit_author: "ukiko"
categories: [密码学]
description: "TEA加密原理简单研究"
tags: [密码学,Crypto]
date: 2018-01-17T00:00:00.000+08:00
draft: false
---

# TEA加密与解密

## TEA加密与解密

> TEA算法由剑桥大学计算机实验室的David Wheeler和Roger Needham于1994年发明。它是一种分组密码算法，其明文密文块为64比特，密钥长度为128比特。TEA算法利用不断增加的Delta(黄金分割率)值作为变化，使得每轮的加密是不同，该加密算法的迭代次数可以改变，建议的迭代次数为32轮。

在游戏项目中，一般需要对资源或数据进行加密保护，最简单高效的加密算法就是采用位与或之类的，但是比较容易被人分析出来。 TEA加密算法不但比较简单，而且有**很强的抗差分分析能力**，**加密速度也比较快**。可以根据项目需求设置加密轮数来增加加密强度。*主要运用了移位和异或运算。密钥在加密过程中始终不变。*

> 差分分析是一种选择明文攻击，其基本思想是：通过分析特定明文差分对相对应密文差分影响来获得尽可能大的密钥。它可以用来攻击任何由迭代一个固定的轮函数的结构的密码以及很多分组密码（包括DES），它是由Biham和Shamir于1991年提出的选择明文攻击。

1. 加密核心函数

	```c++
	void EncryptTEA(unsigned int *firstChunk, unsigned int *secondChunk, unsigned int* key){    unsigned int y = *firstChunk;    unsigned int z = *secondChunk;    unsigned int sum = 0;    unsigned int delta = 0x9e3779b9;    for (int i = 0; i < 8; i++)  //8轮运算(需要对应下面的解密核心函数的轮数一样)    {        sum += delta;        y += ((z << 4) + key[0]) ^ (z + sum) ^ ((z >> 5) + key[1]);        z += ((y << 4) + key[2]) ^ (y + sum) ^ ((y >> 5) + key[3]);    }    *firstChunk = y;    *secondChunk = z;}
	```



> 算法使用了一个神秘常数δ作为倍数，它来源于黄金比率，以保证每一轮加密都不相同。但δ的精确值似乎并不重要，这里 TEA 把它定义为 δ=「(√5 - 1)231」–> delta = 0x9e3779b9;

1. 解密核心函数

	```c++
	void DecryptTEA(unsigned int *firstChunk, unsigned int *secondChunk, unsigned int* key){    unsigned int  sum = 0;    unsigned int  y = *firstChunk;    unsigned int  z = *secondChunk;    unsigned int  delta = 0x9e3779b9;    sum = delta << 3; //32轮运算，所以是2的5次方；16轮运算，所以是2的4次方；8轮运算，所以是2的3次方    for (int i = 0; i < 8; i++) //8轮运算    {        z -= (y << 4) + key[2] ^ y + sum ^ (y >> 5) + key[3];        y -= (z << 4) + key[0] ^ z + sum ^ (z >> 5) + key[1];        sum -= delta;    }    *firstChunk = y;    *secondChunk = z;}
	```

	





