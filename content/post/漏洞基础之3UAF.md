---
title: "漏洞基础之3UAF"
date: 2019-03-20T15:37:13+08:00
draft: false

tags : ["漏洞基础","UAF"]
categories : ["漏洞基础"]
description : "漏洞基础之3UAF"
images : []
---

<!--more-->

- 重写ing

### 0x00原理

堆内存在释放后被直接再次使用(释放了堆块之后，未将该指针值为NULL,导致指针处于悬空状态，被释放的内存能被恶意利用) 在浏览器中比较常见的漏洞

**根本原因是：**

> 应用程序调用free()释放内存时，如果内存块小于256kb，dlmalloc并不马上将内存块释放回内存，而是将内存块标记为空闲状态。这么做的原因有两个：一是内存块不一定能马上释放会内核（比如内存块不是位于堆顶端），二是供应用程序下次申请内存使用（这是主要原因）。当dlmalloc中空闲内存量达到一定值时dlmalloc才将空闲内存释放会内核。如果应用程序申请的内存大于256kb，dlmalloc调用mmap()向内核申请一块内存，返回返还给应用程序使用。如果应用程序释放的内存大于256kb，dlmalloc马上调用munmap()释放内存。dlmalloc不会缓存大于256kb的内存块，因为这样的内存块太大了，最好不要长期占用这么大的内存资源。

### 利用

##### 简单利用

```c
#include <stdio.h>
#include <stdlib.h>
typedef void (*func_ptr)(char *);
void evil_fuc(char command[])
{
system(command);
}
void echo(char content[])
{
printf("%s",content);
}
int main()
{
    func_ptr *p1=(func_ptr*)malloc(4*sizeof(int));
    printf("malloc addr: %p\n",p1);
    p1[3]=echo;
    p1[3]("hello world\n");
    free(p1); //在这里free了p1,但并未将p1置空,导致后续可以再使用p1指针
    p1[3]("hello again\n"); //p1指针未被置空,虽然free了,但仍可使用.
    func_ptr *p2=(func_ptr*)malloc(4*sizeof(int));//malloc在free一块内存后,再次申请同样大小的指针会把刚刚释放的内存分配出来.
    printf("malloc addr: %p\n",p2);
    printf("malloc addr: %p\n",p1);//p2与p1指针指向的内存为同一地址
    p2[3]=evil_fuc; //在这里将p1指针里面保存的echo函数指针覆盖成为了evil_func指针.
    p1[3]("/bin/sh");
    return 0;
}
```

![](http://my-md-1253484710.coscd.myqcloud.com/20180912215411.png)



##### pwnable.kr uaf

![](http://my-md-1253484710.coscd.myqcloud.com/20180912220453.png)

先看看源码：

```cpp
#include <fcntl.h>
#include <iostream> 
#include <cstring>
#include <cstdlib>
#include <unistd.h>
using namespace std;

class Human{
private:
    virtual void give_shell(){
        system("/bin/sh");
    }
protected:
    int age;
    string name;
public:
    virtual void introduce(){
        cout << "My name is " << name << endl;
        cout << "I am " << age << " years old" << endl;
    }
};

class Man: public Human{
public:
    Man(string name, int age){
        this->name = name;
        this->age = age;
        }
        virtual void introduce(){
        Human::introduce();
                cout << "I am a nice guy!" << endl;
        }
};

class Woman: public Human{
public:
        Woman(string name, int age){
                this->name = name;
                this->age = age;
        }
        virtual void introduce(){
                Human::introduce();
                cout << "I am a cute girl!" << endl;
        }
};

int main(int argc, char* argv[]){
    Human* m = new Man("Jack", 25);
    Human* w = new Woman("Jill", 21);

    size_t len;
    char* data;
    unsigned int op;
    while(1){
        cout << "1. use\n2. after\n3. free\n";
        cin >> op;

        switch(op){
            case 1:
                m->introduce();
                w->introduce();
                break;
            case 2:
                len = atoi(argv[1]);
                data = new char[len];
                read(open(argv[2], O_RDONLY), data, len);
                cout << "your data is allocated" << endl;
                break;
            case 3:
                delete m;
                delete w;
                break;
            default:
                break;
        }
    }

    return 0;    
}
```



##### HCFt2016 fheap

##### 网鼎杯CTF2018 第一场 Pwn Babyheap



参考资料：

- https://www.cnblogs.com/Ox9A82/p/5320857.html
- https://www.cnblogs.com/alert123/p/4918041.html
- https://blog.csdn.net/qq_31481187/article/details/73612451
- https://www.anquanke.com/post/id/85281
- https://xz.aliyun.com/t/2609?accounttraceid=ce44f2b3-4957-4509-b7ba-f2bd6eed34d3#toc-4
- https://www.anquanke.com/post/id/85281