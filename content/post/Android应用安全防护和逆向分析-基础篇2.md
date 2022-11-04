---
title: "Android应用安全防护和逆向分析 基础篇②"
date: 2018-07-13T15:01:09+08:00
lastmod: 2018-07-13T15:01:09+08:00
draft: false
keywords: [“NDK","开发"]
description: "Android中NDK的开发"
tags: [
    "android安全",
    "读书笔记"
]
categories: ["读书笔记","Android安全"]
author: "Vorblock"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed


---

# 一、 基础篇②

## 第二章 Android中NDK的开发

### 1.  相关环境

   相关环境参考另外一篇文章[Android安全和开发环境搭建](https://naivete.cc/post/android%E5%AE%89%E5%85%A8%E5%92%8C%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA/)

### 2.  JNI基础

#### 2.1 第一行代码(书上使用Eclipse,我使用AS(简单方便很多))

​	参考文章[Android安全和开发环境搭建](https://naivete.cc/post/android%E5%AE%89%E5%85%A8%E5%92%8C%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA/)中的JNI开发章节

#### 2.2  JNIEnv类型和jobject类型

* **AS 默认自动生成**  

 ```JAVA
 public native String stringFromJNI();
 ```

```java
Java_com_naivete_jni_1study_MainActivity_stringFromJNI(
        JNIEnv *env,
        jobject /* this */) {
    std::string hello = "Hello from C++";
    return env->NewStringUTF(hello.c_str());
```

- **JNIEnv类型**

  通过JNIEnv* 指针就可以对Java端的代码进行操作  

  Jni的所有函数可以查看jni.h文件  

  下面是一些函数eg：

  - NewObject : 创建Java类中的对象。

  - NewString : 创建Java类中的String对象。

  - New<Type>Array : 创建类型为Type的数组对象

  - Get<Type>Field: 获取型为Type的字段。

  - Set<Type>Fileld: 设置类型为Type的字段的值。

  - GetStatic<Type>Field: 获取类型为Type的static的字段。

  - SetStatic<Type>Field:设置类型为Typede的static的字段的值。

  - Call<type>Method: 调用返回类型为Type的方法。

  - CallStatic<Type>Method: 调用返回值类型为Type的Static方法

    .....

- **Jobject参数obj**

  如果native 方法不是static ,obj就代表native方法的实例。

  如果narive方法是static,obj 就代表native方法的类的class对象实例。

- **Java和C++中的基本类型的映射关系**：

  具体查看jni.h文件的详细说明

| Java类型 |    本地类型     | JNI定义的别名  |
| :------: | :------------: | :-----------: |
|   int    |      long      |  jint/jsize   |
|   long   |     _int64     |     jlong     |
|   byte   |  signed char   |     jbyte     |
| boolean  | unsigned char  |   jboolean    |
|   char   | unsigned short |     jchar     |
|  short   |     short      |    jshort     |
|  float   |     float      |    jfloat     |
|  double  |     double     |    jdouble    |
|  Object  |   _jobject*    |    jobject    |

- **jclass类型**

  jclass 表示java中的class 类：

  JNIEnv 类中有如下几个简单的函数可以取得jclass:

  - jclass FindClass( const char* clsName):通过类的名称来获取jclass

  - jcalss GetObjectClass( jobject obj ):通过对象实例来获取jcalss,相当于java 中的getclass方法

  - jclass GetSuperClass(jclass obj):通过jclass 可以获取其父类的jclass对象。

- **native中访问Java层代码**

  常见的应用就是获取类的属性和调用类的方法

  JNi在jni.h头文件中定义了jfieldId、jmethodID类型分别代表JAVA端的属性和方法。

  使用JNI的以下方法来取得相应的jfieldId、jmethodID：

  - GetFieldID、GetMethodID
  - GetStaticFieldID、GetStaticMethodID

  查看jni.h中源函数

```java
 GetFieldID(jclass clazz, const char* name, const char* sig)
```

​	参数说明：

 - clazz 方法依赖的类对象的class对象

- name: 字段的名称

- sig : 字段的签名

  查看签名命令

  ` javap -s -p 字节码.class 文件`

GetMethodID 也能够会的构造函数的jmethodID，创建一个Java对象是可以调用指定的构造方法，eg：

` env->GetmethodID(data_class,"<init>","()v");`

签名的格式：

|  类型   | 相应的签名                               |
| :-----: | ---------------------------------------- |
| boolean | Z                                        |
|  byte   | B                                        |
|  chat   | C                                        |
|  short  | S                                        |
|   int   | I                                        |
|  long   | L                                        |
|  float  | F                                        |
| double  | D                                        |
|  void   | V                                        |
| object  | L用/分割包的完整类名；Ljava/lang/String; |
|  Array  | [签名 [I [Ljava/lang/Object              |
| Method  | (参数类型签名··· .) 返回值类型签名       |

eg:

```java
package com.naivete.jni_study;

import java.util.Date;

public class Hello {
    public int property;
    public int function(int foo, Date date,int[] arr){
        System.out.println("function");
        return 0;
    }

    public native void test();
}
```

```C++
extern "C"
JNIEXPORT void JNICALL
Java_com_naivete_jni_1study_Hello_test(JNIEnv *env, jobject instance) {

    // TODO
    jclass helloclazz = env->GetObjectClass(instance);
    jfieldID field_prop = env->GetFieldID(helloclazz,"property","I");   //取到property字段
    jmethodID method_fun = env->GetMethodID(helloclazz,"function","ILjava/util/Date;[I)I"); //取到function函数
    env->CallIntMethod(instance,method_fun,0L,NULL,NULL);
}
```

GetStaticFieldID与GetStaticMethodID这两个方法的用法大同小异。

#### 2.3 JNIEnv类型中方法的使用

 - 在java中定义一个属性，再从C++代码中将其设置成另外的值

#####  2.3.1 native中获取方法的ID

   ```java
       private static String TAG = "Hello";
       public int number = 0;
       public native void sayHello();
   
       public static void main() {
           Hello hello = new Hello();
           hello.sayHello();
           System.out.print(hello.number);
           Log.d(TAG, ""+hello.number);
       }
   }
   ```

   ```C++
   JNIEXPORT void JNICALL
   Java_com_naivete_jni_1study_Hello_sayHello(JNIEnv *env, jobject instance) {
   
       // TODO
       jclass helloclazz = env->GetObjectClass(instance);
       jfieldID id_number = env->GetFieldID(helloclazz,"number","I"); //获取numberID
       jint number = env->GetIntField(instance,id_number); //获取number的值;
       cout<<number<<endl;  //输出到控制台
       env->SetIntField(instance,id_number,100L); //设置number的值;注意jint对应c++ long类型
   }
   ```

   ![1](http://my-md-1253484710.coscd.myqcloud.com/android-note-2-1.png)

   JNIEnv 还提供了许多Call<Type>Method 和CallStatic<Type>Method 还有CallNovirtual<Type>Method函数，需要通过GetMethodID来取得相应的方法的jmethodId传入到上述函数的参数中

   调用示例方法的三种形式如下：

   `Call<Type>Method(jobject obj,jmethodID id,id,·······); ` //常用的方式

   `Call<Type>Method(jobject obj,jmethodID id,id,va_list lst);` //有指向参数表的va_list变量（很少使用）

   `Call<Type>Method(jobject obj,jmethodID id,id,jvalue * v);` //有指向jvalue或jvalue数组指针时用的

   jvalue 是union联合体，定义jvalue数组传递到方法中，这样可以包含多种类型的参数：

   ```C
   typedef union jvalue{
       jboolean z;
       jbytpe   b;
       jchar    c;
       jshort   s;
       jint     i;
       jlong    j;
       jfloat   f;
       jdouble  d;
       jobject  l;
   }jvalue;
   ```

   比如在Java中有这样一个方法：

   ```java
   boolean function(int a,double b,char c){
   
   ·····
   
   }
   
   ```

   1）在C++中使用第一种方法调用function方法：

   `env->CallbooleanMethod(obj,id_function,10L，3.4，L'a')`

   obj:functon对象，id_function:functiond的id,10L、3.4、L'a'是对应的参数。

   L'a' 中的L是因为Java中的字符是Unicode双字节的，而C++中的字节是单字节的，所以要变成宽字符。

   2）在C++中使用第三种方法function调用：

   ```C++
   jvalue* args = new Jvalue[3]
   args[0] = 10L;
   args[1] = 3.22;
   args[2] = L'a';
   env->GetBooleanMethod(obj,id_function,args);
   delete[] args;  //是否指针堆内存
   ```
##### 2.3.2 Java和C++中的多态机制

   JNIEnv中的特殊方法CallNovirtual<type>Method。来帮助java调用Java中父类的方法。

   - 介绍了-C++和java多态的基础知识。
   - 步骤：
     - 获取obj中对象的class 对象 GetObjectClass(obj)
     - 获取java中father字段的id GetFieldID()
     - 获取father字段的对象类型 GetObjectField
     - 获取father对象的class对象 FindClass
     - 获取father对象中function方法ID GetMethodID()
     - 调用父类中的function方法（会执行子类的方法）CallvoidMethod
     - 调用父类中的function方法（会执行父类的方法）CallNonvirtualVoidMethod()


#### 2.4 创建Java对象及字符串的操作方法

##### 2.4.1 native中创建Java对象 

​	两种方法：

 - 第一种：

   ` jobject Newobject(jclass clazz,jmethodID methodID,·····)`

   - clazz 需要创建的Java对象的Class对象。
   - methodID :传递一个方法的ID: 构造方法
   - 第三个参数：构造函数需要传入的参数值（默认不传递） 默认构造方法返回值签名始终是"()V",方法的名称始终是"<init>"。

   在C++中构造Java中的Date对象调用方法getTime():

   ```C++
   jclass clazz_date = env->FindClass("java/util/Date"); //获取date对象
   jmethodID mid_date = env->GetMethodID(clazz_date,"<init>","()V"); //获取构造方法的ID
   jobject now = env->NewObject(clazz_date,mid_date); //生成Date对象
   jmethodID mid_date_getTime = env->GetMethodID(clazz_date,"getTime()","()J"); //获取getTime的ID
   jlong time = env->CallLongMethod(now,mid_date_getTime);//调用getTime返回时间
   printf("%I64d",time);
   ```

- 第二种：

  用AllocObject函数创建一个对象，可以根据传入的jclass创建一个java对象，但是状态时未初始化的，在这个对象之前绝对要用CallNonvirtualVoidMethod来调用该jclass的构造函数这样可以延迟构造函数的调用。用的比较少。

  eg：略；

##### 2.4.2 native中操作Java字符串

​	Java-String对象是Unicode(UTF-16)码 一个字符总是占用两个字节 可以通过JNI接口将Java中的字符串转换到C++的宽字符串（wchar_t*),或者传回一个UTF-8编码的字符串（char * )到C++ 反过来同理。

JNIEnv中的一些C++方法：

1）获取字符串的长度：

​	` jsize GetStringLength(jstring j_msg)` 

1) 将jstring 对象拷贝到const jchar* 指针字符串：

​	//拷贝Java字符串并以UTF-8编码传入jstr:

​	` env->GetStringRegion(jstring j_msg.jsize start,jszie len,jchar* jstr);`

​	////拷贝Java字符串并以UTF-16编码传入jstr:

​	` env->GetStringUTFRegion(jstring j_msg.jsize start,jszie len, char* jstr);`

3) 生成一个jstring 对象

​	`jobject NewString(const jchar* jstr,int size);` 

​	将字符串指针jstr转换成jstring。

4) 将jstring对象转换成const jchar* 字符串指。

 - GetStringChars 开内存 指针指向先开的内存

   `const* jchar * GetStringChars(jstring j_msg,jboolean* copied)`

   返回一个UTF-16编码的宽字符串（jchar*);

   对应的释放内存方法：

   `ReleaseStringChars(jstring j_msg,const jchar* jstr)`

- GetStringUTFChars 不开内存直接指向Java中string的指针

  ` const char* GetStringUTFChars(jstring str,jboolean* copied)`

  取得UTF-8编码的字符串

  释放：

  `ReleaseStringUTFChars(jstring j_msg,const jchar* jstr)`

5) 将jstring 对象转化成const jchar* 字符串指针：

​	`const jchar* GetStringCritical(jstring j_msg,Jboolean* copied)`

​	作用:增加直接传回指向Java字符串的指针的可能性（而不是拷贝）；

​	在`GetStringCritical/ReleaseStringCritical`之间的关键区域之间不能调用任何其他JNI函数。否则会造成关键区域代码执行期间垃圾回收器停止工作。任何触发垃圾回收器的的线程也将暂停。

​	释放：

​	`ReleaseStringCritical(jstring j_msg,const jchar* jstr)`

实例eg：（与书上不同,思路大概相同）

``` java
public class MainActivity extends AppCompatActivity {
    private EditText et;
    private TextView tv;
    private Button bt;
    public String text = null;

    // Used to load the 'native-lib' library on application startup.
    static {
        System.loadLibrary("native-lib");
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        et = findViewById(R.id.editText);
        tv = findViewById(R.id.tv);
        bt = findViewById(R.id.button);
        bt.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                text = et.getText().toString().trim();
                callCppFunction();
                tv.setText(text);
            }
        });

    }

    public native void callCppFunction();
}
```

```C++
#include <jni.h>
#include <string>
#include<iostream>
#include<algorithm>
using namespace std;

extern "C"
JNIEXPORT void JNICALL
Java_com_example_naivete_jnidemo_MainActivity_callCppFunction(JNIEnv *env, jobject instance) {

    // TODO
    //获取text
    jfieldID  fid_tx = env->GetFieldID(env->GetObjectClass(instance),"text","Ljava/lang/String;");
    //获取ext对象
    jstring j_tx = (jstring)env->GetObjectField(instance,fid_tx);
    //第一种方式
    //获得字符串指针：
    const jchar* jstr1 = env->GetStringChars(j_tx,NULL);
    //z转换成宽字符
    wstring wstr((const wchar_t*)jstr1);
    //释放指针
    env->ReleaseStringChars(j_tx,jstr1);
    //第一种END

    //第二种
    const jchar * jstr2 = env->GetStringCritical(j_tx,NULL);
    wstring wstr2((const wchar_t*)jstr2);
    env->ReleaseStringCritical(j_tx,jstr2);
    //END

    //第三种
    jsize len = env->GetStringLength(j_tx);  //获取长度
    jchar * jstr3 = new jchar[len+1];
    jstr3[len]=L'\0';
    //复制
    env->GetStringRegion(j_tx,0,len,jstr3);
    wstring wstr3((const wchar_t*)jstr3);
    delete[] jstr3;
    //End

    //倒序
    reverse(wstr.begin(),wstr.end());
    jstring j_new_str = env->NewString((const jchar*)wstr.c_str(),(jint)wstr.size());
    env->SetObjectField(instance,fid_tx,j_new_str);
}
```

![](http://my-md-1253484710.coscd.myqcloud.com/android-note-2-2.png)

​	

#### 2.5 C/C++中操作Java中的数组

​	在java中数组分为两种：

 - 基本类型数组

 - 对象类型数组

   一个能用于两种不同类型数组的函数是：GetArrayLength(jarray array)。

##### 2.5.1 操作基本类型的数组

1. Get<Type>ArrayElements方法  
    `Get<Type>ArrayElements(<Type>Array arr,jboolean* isCopide)`

    把Java中的基本类型的数组转换成C++中的数组 两种方式：

    一是拷贝一份传回本地，另外一种是把指向Java数组的指针直接传回到本地代码中

    处理完后，通过Release<Type>Arrayelements 来释放数组。

2. Release<Type>Arrayelement 方法  
    `Release<Type>Arrayelement(Type>Array arr,<Type>* array,jint mode)`

    这个函数可以选择如何处理Java和C++中的数组，是提交还是撤销····内存是否释放等等。

    mode的取值：

     - 0：对Java的数组进行更新并且释放C/C++数组
     - JNI_COMMIT：更新但是不释放
     - JNI_ABOUT：不更新，释放。

3. GetPrimittiveArrayCritical方法  
    `GetPrimittiveArrayCritical(jarray arr,jboolean* isCopied)`

4. ReleasePrimittiveArrayCritical方法  
    `ReleasePrimittiveArrayCritical(jarray arr,void* array,jint mode)`

5. Get<type>ArrayRegion方法  
    `Get<type>ArrayRegion(<Type>Arryay arr,jsize strat ,jsize len,<Type>* buffer)`

    在C++中开辟内存，拷贝数组到内存中。

6. Set<type>ArrayRegion  
    `Set<type>ArrayRegion(<Type>Arryay arr,jsize strat ,jsize len,const <Type>* buffer)`

    把Java基本类型数组中的指定范围的元素用C++数组中的元素来赋值。

7. <Type>ArrayNew方法  
    `<Type>ArrayNew<Type>Array(jszie sz)`

    指定一个长度然后返回相应的Java基本类型的数组。

##### 2.5.2 操作对象数组类型

​	JNI未提供把Java对象数组 直接转到C++对象数组的函数。而是通过`Get/SetObjectArrayaElement`这样的函数来对java中的对象数组进行操作。因为未拷贝 所以没有释放操作。`NewObjectArray`可以通过指定长度和初始值来创建某一个类的数组。

例子：两种类型的操作：

  略·····

​	注：书本P34-36



#### 2.6 C++/C中的引用类型和ID缓存

##### 2.6.1 引用类型

​	从Java创建对象传到本地C/C++代码时会产生引用，根据Java的垃圾回收机制，只要存在引用就不会触发改引用所指的Java对象垃圾回收。

​	几种C/C++中的引用类型：

1. 局部引用：（最常见）

    局部引用只在该native函数中有用，所有在该函数中产生的局部引用，都会在函数返回时自动释放，也可以使用DeleteLocalRef函数手动释放。

    有效期中能传递到别的本地函数中，千万不要用C++全局变量保存它，或者把它定义为C++静态局部变量。

2. 全局引用：

    可以跨越当前线程，在对个native函数中有效，需要手动释放。会阻止垃圾回收器回收这个引用所指的对象。

    不同于局部引用，全局引用的创建不是由JNI自动创建的，全局引用是需要调用NewGlobalRef函数，释放使用ReleaseGlobalRef函数。

3. 弱全局引用

    与全局引用相似。不一样的为不会阻止垃圾回收器回收这个引用所指对象，使用NewWeakGlobalRef和ReleaseWeakGlobalRef来产生和释放。

    关于引用的一些函数：

    `jobject NewGlobalRef(jobject obj)`

    `jobject NewLocalRef(jobject obj)`

    `jobject New WeakGlobalRef(jobject obj)`

    `void DeleteGlobalRef(jobject obj)`

    `void DeleteLocalRef(jobject obj)`

    `void DeleteWeakGlobalRef(jobject obj)`

    很容易理解上面6个函数

    `jboolean IsSameObject(jobject obj1,jobject obj2)`

    这个函数用来比较两个引用是否相等，但是对于弱引用有一个特别的功能，如果把NULL传入要比较的对象中就能判断弱全局引用所指的Java对象是否被回收。

    缓存jfieldID/jmethodID.减小查询开销。

##### 2.6.2 缓存方法

1. 在用的时候缓存

    在native代码中使用static局部变量来保存已经查询过的id,就缓存下了id。

2. 在Java类初始化时缓存

    比较好的方法，在native调用前把所有ID全部保存下来。可以让Java代码在第一次加载这个类的时候首先调用本地代码初始化所有的jfildID/jmethodID.这样可以省去多次确定ID是否存在的语句。这些jfildID/jmethodID定义在C++的全局。当java类卸载或者重新加载的时候，也会重新计算ID.

    ```java
    public class TestNative{
        static{
            initNativeIDs();  //静态代码块进行初始化
        }
        static native void initNativeIDs();
        int propInt = 0;
        String propStr = "";
        public native void otherNative();
        ···········
    }
    ```

    ```C++
    //全局变量
    jfieldID g_propInt_id = 0;
    jfieldID g_propStr_id = 0;
    
    JNIEXPORT void JNICALL Java_····init（JNIEnv* env,jobject clazz）{
        jfieldID g_propInt_id = GetfieldID(clazz,"propInt","I");
        jfieldID g_propStr_id = GetfieldID(clazz,"propStr","/Ljava/lang/String;");
    }
    JNIEXPORT void JNICALL Java_····other（JNIEnv* env,jobject clazz）{
        ············
    }
    
    ```

    

### 总结

​	主要是NDK开发相关。

​	感觉系统的学了一遍还是感觉不错的。

​	可以多找网上的例子来练习练习，加深对JNI 的了解。

​    
