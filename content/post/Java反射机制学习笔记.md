---
title: "Java反射机制学习笔记"
date: 2018-07-25T16:40:20+08:00
lastmod: 2018-07-25T16:40:20+08:00
draft: false
description: "Java反射机制"
tags: ["Java"]
categories: ["Java","笔记"]
author: "Vorblock"
---

## Java 反射机制学习记录

在逆向中反射也是能经常看见，之前理解不是很深透，现在来重点学习一下，做个笔记。

### 什么是反射机制？

反射(Reflection)是Java 程序开发语言的特征之一，它允许运行中的 Java 程序获取自身的信息，并且可以操作类或对象的内部属性。 通俗一点：在动态运行时，获取到一个类的所有方法以及成员。简而言之，通过反射，我们可以在**运行时**获得程序或程序集中每一个类型的成员和成员的信息。 

### 作用？

- 1.在运行时判断任意一个对象所属的类；
- 2.在运行时构造任意一个类的对象；
- 3.在运行时判断任意一个类所具有的成员变量和方法（通过反射甚至可以调用private方法）；
- 4.在运行时调用任意一个对象的方法

##### **是运行时而不是编译时**

- 获取某些类的一些变量，调用某些类的私有方法。 
- 增加代码的灵活性。很多主流框架都使用了反射技术.像ssh框架都采用两种技术 xml做配置文件+反射技术.

### 基本使用

反射相关的类一般都在java.lang.relfect 包里。

1. 获取Class对象 3种方法

   (1)使用Class类的forName静态方法:

   ```java
   public static Class<?> forName(String className)
   //在JDBC开发中常用此方法加载数据库驱动:
   Class.forName(driver);
   ```

   (2)直接获取某一个对象的class，比如:

   ```java
   Class<?> klass = int.class;
   Class<?> classInt = Integer.TYPE;
   ```

   (3)调用某个对象的getClass()方法,比如:

   ```java
   StringBuilder str = new StringBuilder("123");
   Class<?> klass = str.getClass();
   ```

2. 判断是否为某一个类的实例

   一般使用instanceof来判断，也可以借助反射中的Class对象的isInstance()方法来判断  是一个Native方法：

   ```java
   public native boolean isInstance(Object obj);
   ```

3. 创建实例

   通过放射来生成对象两种方式：

   （1）使用Class对象的newInstance()方法来创建Class对象对应类的实例。

   ```
   Class<?> c = String.class;
   Object str = c.newInstance();
   ```

   （2）先通过Class对象获取指定的Constructor对象，再调用Constructor对象的newInstance()方法来创建实例。这种方法可以用指定的构造器构造类的实例。

   ```
   //获取String所对应的Class对象
   Class<?> c = String.class;
   //获取String类带一个String参数的构造器
   Constructor constructor = c.getConstructor(String.class);
   //根据构造器创建实例
   Object obj = constructor.newInstance("23333");
   System.out.println(obj);
   ```

4. 获取方法

   getDeclaredMethods()方法返回类或接口声明的所有方法，包括公共、保护、默认（包）访问和私有方法，但不包括继承的方法。

   ```
   public Method[] getDeclaredMethods() throws SecurityException
   ```

   getMethods()方法返回某个类的所有公用（public）方法，包括其继承类的公用方法。

   ```
   public Method[] getMethods() throws SecurityException
   ```

   getMethod方法返回一个特定的方法，其中第一个参数为方法名称，后面的参数为方法的参数对应Class的对象

   ```
   public Method getMethod(String name, Class<?>... parameterTypes)
   ```

eg:

```java
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;


public class test {
    public static void test1() throws IllegalAccessException, InstantiationException,
            NoSuchMethodException, InvocationTargetException {
        Class<?> c = methodClass.class;
        Object object = c.newInstance();
        Method[] methods = c.getMethods();
        Method[] declaredMethods = c.getDeclaredMethods();
        Method method = c.getMethod("add", int.class, int.class); //获取add方法
        System.out.println("getMethods获取的方法：");
        for (Method m : methods)
            System.out.println(m);
        System.out.println("getDeclaredMethods获取的方法：");
        for (Method m : declaredMethods)
            System.out.println(m);
    }

    public static void main(String[] args) {
        try{
            test1();

        }catch (Exception e){
            e.printStackTrace();
        }

    }
}

class methodClass {
    public final int fuck = 3;
    public int add(int a,int b) {
        return a+b;
    }
    public int sub(int a,int b) {
        return a+b;
    }
}
```

![](http://my-md-1253484710.coscd.myqcloud.com/20180725151348.png)

5. 获取构造器信息

   通过Class类的getConstructor方法得到Constructor类的一个实例，而Constructor类有一个newInstance方法可以创建一个对象实例: 

   ```java
   public T newInstance(Object ... initargs)
   ```

6. 获取类的成员字段信息

   `getFiled`: 访问公有的成员变量

   `getDeclaredField`：所有已声明的成员变量。但不能得到其父类的成员变量 

   `getFileds`和`getDeclaredFields`用法同上（参照Method） 

7. 调用方法

   获取到方法后使用invoke()方法来调用这个方法 

   ```java
   public Object invoke(Object obj, Object... args)
           throws IllegalAccessException, IllegalArgumentException,
              InvocationTargetException
   ```

   eg：

   ```java
   public class test1 {
   
       public static void main(String[] args) throws IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException {
           Class<?> klass = methodClass.class;
           //创建methodClass的实例
           Object obj = klass.newInstance();
           //获取methodClass类的add方法
           Method method = klass.getMethod("add",int.class,int.class);
           //调用method对应的方法 => add(1,4)
           Object result = method.invoke(obj,1,4);
           System.out.println(result);
       }
   
   }
   
   class methodClass {
   
       public final int fuck = 3;
       public int add(int a,int b) {
           return a+b;
       }
       public int sub(int a,int b) {
           return a+b;
       }
   }
   ```

8. 创建数组

   ```java
   public static void testArray() throws ClassNotFoundException {
           Class<?> cls = Class.forName("java.lang.String");
           Object array = Array.newInstance(cls,25); //通过Array.newInstance()
           //往数组里添加内容
           Array.set(array,0,"hello");  //Array类为java.lang.reflect.Array
           Array.set(array,1,"Java");
           Array.set(array,2,"fuck");
           Array.set(array,3,"Scala");
           Array.set(array,4,"Clojure");
           //获取某一项的内容
           System.out.println(Array.get(array,3));
       }
   ```

   ```java
   public static Object newInstance(Class<?> componentType, int length)
           throws NegativeArraySizeException {
           return newArray(componentType, length);
       }
   ```

   ```java
   private static native Object newArray(Class<?> componentType, int length)
           throws NegativeArraySizeException;
   ```

   ```c++
   arrayOop Reflection::reflect_new_array(oop element_mirror, jint length, TRAPS) {
     if (element_mirror == NULL) {
       THROW_0(vmSymbols::java_lang_NullPointerException());
     }
     if (length < 0) {
       THROW_0(vmSymbols::java_lang_NegativeArraySizeException());
     }
     if (java_lang_Class::is_primitive(element_mirror)) {
       Klass* tak = basic_type_mirror_to_arrayklass(element_mirror, CHECK_NULL);
       return TypeArrayKlass::cast(tak)->allocate(length, THREAD);
     } else {
       Klass* k = java_lang_Class::as_Klass(element_mirror);
       if (k->oop_is_array() && ArrayKlass::cast(k)->dimension() >= MAX_DIM) {
         THROW_0(vmSymbols::java_lang_IllegalArgumentException());
       }
       return oopFactory::new_objArray(k, length, THREAD);
     }
   }
   ```

   Array类的set()和get()方法都为Native方法，在HotSpot JVM里分别对应Reflection::array_set和Reflection::array_get方法 

9. 获取泛型

    getGenericHelper(HashMap<String, Person> map) 

   ```java
   public static  void getGenericType() {
           try {
               Method method =TestHelper.class.getDeclaredMethod("getGenericHelper",HashMap.class);
               Type[] genericParameterTypes = method.getGenericParameterTypes();
               // 检验是否为空
               if (null == genericParameterTypes || genericParameterTypes.length < 1) {
                   return ;
               }
               // 取 getGenericHelper 方法的第一个参数
   
               ParameterizedType parameterizedType=(ParameterizedType)genericParameterTypes[0];
               Type rawType = parameterizedType.getRawType();
               System.out.println("----> rawType=" + rawType);
               Type[] actualTypeArguments = parameterizedType.getActualTypeArguments();
               if (actualTypeArguments==genericParameterTypes || actualTypeArguments.length<1) {
                   return ;
               }
               //  打印出每一个类型          
               for (int i = 0; i < actualTypeArguments.length; i++) {
                   Type type = actualTypeArguments[i];
                   System.out.println("----> type=" + type);
               }
           } catch (Exception e) {
   
           }
   
       }
   ```

   10. 获得 Metho,Field,Constructor 的访问权限

       ```java
       int modifiers = method.getModifiers(); 
       Modifier.toString(modifiers); 
       ```

### Invoke方法 

比较重点

invoke方法的实现：

```java
@CallerSensitive
public Object invoke(Object obj, Object... args)
    throws IllegalAccessException, IllegalArgumentException,
       InvocationTargetException
{
    if (!override) {
        if (!Reflection.quickCheckMemberAccess(clazz, modifiers)) {
            Class<?> caller = Reflection.getCallerClass();
            checkAccess(caller, clazz, obj, modifiers);
        }
    }
    MethodAccessor ma = methodAccessor;             // read volatile
    if (ma == null) {
        ma = acquireMethodAccessor();
    }
    return ma.invoke(obj, args);
}
```

1. 权限检查

   首先检查AccessibleObject的override属性的值 。AccessibleObject 类是 Field、Method 和 Constructor 对象的基类 。override 默认为false,调试需要权限调用规则，反正不需要。

   默认情况下首先用Reflection.quickCheckMemberAccess(clazz, modifiers)方法检查方法是否为public，如果是的话跳出本步；如果不是public方法，那么用Reflection.getCallerClass()方法获取调用这个方法的Class对象，这是一个native方法: 

   ```java
   @CallerSensitive
       public static native Class<?> getCallerClass();
   ```

   ```java
   JNIEXPORT jclass JNICALL Java_sun_reflect_Reflection_getCallerClass__
   (JNIEnv *env, jclass unused)
   {
       return JVM_GetCallerClass(env, JVM_CALLER_DEPTH);
   }
   ```

   ```c++
   JVM_ENTRY(jclass, JVM_GetCallerClass(JNIEnv* env, int depth))
     JVMWrapper("JVM_GetCallerClass");
   
     // Pre-JDK 8 and early builds of JDK 8 don't have a CallerSensitive annotation; or
     // sun.reflect.Reflection.getCallerClass with a depth parameter is provided
     // temporarily for existing code to use until a replacement API is defined.
     if (SystemDictionary::reflect_CallerSensitive_klass() == NULL || depth != JVM_CALLER_DEPTH) {
       Klass* k = thread->security_get_caller_class(depth);
       return (k == NULL) ? NULL : (jclass) JNIHandles::make_local(env, k->java_mirror());
     }
   
     // Getting the class of the caller frame.
     //
     // The call stack at this point looks something like this:
     //
     // [0] [ @CallerSensitive public sun.reflect.Reflection.getCallerClass ]
     // [1] [ @CallerSensitive API.method                                   ]
     // [.] [ (skipped intermediate frames)                                 ]
     // [n] [ caller                                                        ]
     vframeStream vfst(thread);
     // Cf. LibraryCallKit::inline_native_Reflection_getCallerClass
     for (int n = 0; !vfst.at_end(); vfst.security_next(), n++) {
       Method* m = vfst.method();
       assert(m != NULL, "sanity");
       switch (n) {
       case 0:
         // This must only be called from Reflection.getCallerClass
         if (m->intrinsic_id() != vmIntrinsics::_getCallerClass) {
           THROW_MSG_NULL(vmSymbols::java_lang_InternalError(), "JVM_GetCallerClass must only be called from Reflection.getCallerClass");
         }
         // fall-through
       case 1:
         // Frame 0 and 1 must be caller sensitive.
         if (!m->caller_sensitive()) {
           THROW_MSG_NULL(vmSymbols::java_lang_InternalError(), err_msg("CallerSensitive annotation expected at frame %d", n));
         }
         break;
       default:
         if (!m->is_ignored_by_security_stack_walk()) {
           // We have reached the desired frame; return the holder class.
           return (jclass) JNIHandles::make_local(env, m->method_holder()->java_mirror());
         }
         break;
       }
     }
     return NULL;
   JVM_END
   ```

   获取了这个Class对象caller后用checkAccess方法做一次快速的权限校验 :

   ```java
   volatile Object securityCheckCache;
   
       void checkAccess(Class&lt;?&gt; caller, Class&lt;?&gt; clazz, Object obj, int modifiers)
           throws IllegalAccessException
       {
           if (caller == clazz) {  // 快速校验
               return;             // 权限通过校验
           }
           Object cache = securityCheckCache;  // read volatile
           Class&lt;?&gt; targetClass = clazz;
           if (obj != null
               &amp;&amp; Modifier.isProtected(modifiers)
               &amp;&amp; ((targetClass = obj.getClass()) != clazz)) {
               // Must match a 2-list of { caller, targetClass }.
               if (cache instanceof Class[]) {
                   Class&lt;?&gt;[] cache2 = (Class&lt;?&gt;[]) cache;
                   if (cache2[1] == targetClass &amp;&amp;
                       cache2[0] == caller) {
                       return;     // ACCESS IS OK
                   }
                   // (Test cache[1] first since range check for [1]
                   // subsumes range check for [0].)
               }
           } else if (cache == caller) {
               // Non-protected case (or obj.class == this.clazz).
               return;             // ACCESS IS OK
           }
   
           // If no return, fall through to the slow path.
           slowCheckMemberAccess(caller, clazz, obj, modifiers, targetClass);
       }
   ```

   先快速检查，未通过的话建立缓存，中间再检查；

   如果都没有通过：进行更详细的检查;

   ```java
   // Keep all this slow stuff out of line:
   void slowCheckMemberAccess(Class&lt;?&gt; caller, Class&lt;?&gt; clazz, Object obj, int modifiers,
                              Class&lt;?&gt; targetClass)
       throws IllegalAccessException
   {
       Reflection.ensureMemberAccess(caller, clazz, obj, modifiers);
   
       // Success: Update the cache.
       Object cache = ((targetClass == clazz)
                       ? caller
                       : new Class&lt;?&gt;[] { caller, targetClass });
   
       // Note:  The two cache elements are not volatile,
       // but they are effectively final.  The Java memory model
       // guarantees that the initializing stores for the cache
       // elements will occur before the volatile write.
       securityCheckCache = cache;         // write volatile
   }
   ```

   用Reflection.ensureMemberAccess方法继续检查权限，若检查通过就更新缓存，这样下一次同一个类调用同一个方法时就不用执行权限检查了，这是一种简单的缓存机制。

  2. 调用MethodAccessor的invoke方法

     由sun.reflect.MethodAccessor 处理

     ```java
       /** This interface provides the declaration for
           java.lang.reflect.Method.invoke(). Each Method object is
           configured with a (possibly dynamically-generated) class which
           implements this interface.
       */
       	public interface MethodAccessor {    //是一个接口
           /** Matches specification in {@link java.lang.reflect.Method} */
           public Object invoke(Object obj, Object[] args)
               throws IllegalArgumentException, InvocationTargetException;
       }
     ```

       分析其Usage可得它的具体实现类有: 

       - sun.reflect.DelegatingMethodAccessorImpl
       - sun.reflect.MethodAccessorImpl
       - sun.reflect.NativeMethodAccessorImpl

       methodAccessor实例由reflectionFactory对象操控生成，它在AccessibleObject下的声明如下: 

       ```java
       // Reflection factory used by subclasses for creating field,
       // method, and constructor accessors. Note that this is called
       // very early in the bootstrapping process.
       static final ReflectionFactory reflectionFactory =
           AccessController.doPrivileged(
               new sun.reflect.ReflectionFactory.GetReflectionFactoryAction());
       ```

       sun.reflect.ReflectionFactory类的源码： 

       ```java
       public class ReflectionFactory {
       
           private static boolean initted = false;
           private static Permission reflectionFactoryAccessPerm
               = new RuntimePermission("reflectionFactoryAccess");
           private static ReflectionFactory soleInstance = new ReflectionFactory();
           // Provides access to package-private mechanisms in java.lang.reflect
           private static volatile LangReflectAccess langReflectAccess;
       
           // 这里设计得非常巧妙
           // "Inflation" mechanism. Loading bytecodes to implement
           // Method.invoke() and Constructor.newInstance() currently costs
           // 3-4x more than an invocation via native code for the first
           // invocation (though subsequent invocations have been benchmarked
           // to be over 20x faster). Unfortunately this cost increases
           // startup time for certain applications that use reflection
           // intensively (but only once per class) to bootstrap themselves.
           // To avoid this penalty we reuse the existing JVM entry points
           // for the first few invocations of Methods and Constructors and
           // then switch to the bytecode-based implementations.
           //
           // Package-private to be accessible to NativeMethodAccessorImpl
           // and NativeConstructorAccessorImpl
           private static boolean noInflation        = false;
           private static int     inflationThreshold = 15;
       
           //......
       
       	//这是生成MethodAccessor的方法
           public MethodAccessor newMethodAccessor(Method method) {
               checkInitted();
       
               if (noInflation && !ReflectUtil.isVMAnonymousClass(method.getDeclaringClass())) {
                   return new MethodAccessorGenerator().
                       generateMethod(method.getDeclaringClass(),
                                      method.getName(),
                                      method.getParameterTypes(),
                                      method.getReturnType(),
                                      method.getExceptionTypes(),
                                      method.getModifiers());
               } else {
                   NativeMethodAccessorImpl acc =
                       new NativeMethodAccessorImpl(method);
                   DelegatingMethodAccessorImpl res =
                       new DelegatingMethodAccessorImpl(acc);
                   acc.setParent(res);
                   return res;
               }
           }
       
           //......
       
           /** We have to defer full initialization of this class until after
           the static initializer is run since java.lang.reflect.Method's
           static initializer (more properly, that for
           java.lang.reflect.AccessibleObject) causes this class's to be
           run, before the system properties are set up. */
           private static void checkInitted() {
               if (initted) return;
               AccessController.doPrivileged(
                   new PrivilegedAction<Void>() {
                       public Void run() {
                           // Tests to ensure the system properties table is fully
                           // initialized. This is needed because reflection code is
                           // called very early in the initialization process (before
                           // command-line arguments have been parsed and therefore
                           // these user-settable properties installed.) We assume that
                           // if System.out is non-null then the System class has been
                           // fully initialized and that the bulk of the startup code
                           // has been run.
       
                           if (System.out == null) {
                               // java.lang.System not yet fully initialized
                               return null;
                           }
       
                           String val = System.getProperty("sun.reflect.noInflation");
                           if (val != null && val.equals("true")) {
                               noInflation = true;
                           }
       
                           val = System.getProperty("sun.reflect.inflationThreshold");
                           if (val != null) {
                               try {
                                   inflationThreshold = Integer.parseInt(val);
                               } catch (NumberFormatException e) {
                                   throw new RuntimeException("Unable to parse property sun.reflect.inflationThreshold", e);
                               }
                           }
       
                           initted = true;
                           return null;
                       }
                   });
           }
       }
       ```

       MethodAccessor实现有两个版本，一个是Java版本，一个是native版本 。

       Java实现的版本在初始化时需要较多时间，但长久来说性能较好；native版本正好相反，启动时相对较快，但运行时间长了之后速度就比不过Java版了 

       为了尽可能地减少性能损耗，HotSpot JDK采用“inflation”的技巧：让Java方法在被反射调用时，开头若干次使用native版，等反射调用次数超过阈值时则生成一个专用的MethodAccessor实现类，生成其中的invoke()方法的字节码，以后对该Java方法的反射调用就会使用Java版本。 这项优化是从JDK 1.4开始的。 

     3. 在JVM层面探究invoke0方法

        invoke0方法是一个native方法,它在HotSpot JVM里调用JVM_InvokeMethod函数: 

```c++
          JNIEXPORT jobject JNICALL Java_sun_reflect_NativeMethodAccessorImpl_invoke0
          (JNIEnv *env, jclass unused, jobject m, jobject obj, jobjectArray args)
          {
              return JVM_InvokeMethod(env, m, obj, args);
          }
```

openjdk/hotspot/src/share/vm/prims/jvm.cpp 

```c++
JVM_ENTRY(jobject, JVM_InvokeMethod(JNIEnv *env, jobject method, jobject obj, jobjectArray args0))
  JVMWrapper("JVM_InvokeMethod");
  Handle method_handle;
  if (thread->stack_available((address) &method_handle) >= JVMInvokeMethodSlack) {
    method_handle = Handle(THREAD, JNIHandles::resolve(method));
    Handle receiver(THREAD, JNIHandles::resolve(obj));
    objArrayHandle args(THREAD, objArrayOop(JNIHandles::resolve(args0)));
    oop result = Reflection::invoke_method(method_handle(), receiver, args, CHECK_NULL);
    jobject res = JNIHandles::make_local(env, result);
    if (JvmtiExport::should_post_vm_object_alloc()) {
      oop ret_type = java_lang_reflect_Method::return_type(method_handle());
      assert(ret_type != NULL, "sanity check: ret_type oop must not be NULL!");
      if (java_lang_Class::is_primitive(ret_type)) {
        // Only for primitive type vm allocates memory for java object.
        // See box() method.
        JvmtiExport::post_vm_object_alloc(JavaThread::current(), result);
      }
    }
    return res;
  } else {
    THROW_0(vmSymbols::java_lang_StackOverflowError());
  }
JVM_END
```

openjdk/hotspot/src/share/vm/runtime/reflection.cpp :

```c++
oop Reflection::invoke_method(oop method_mirror, Handle receiver, objArrayHandle args, TRAPS) {
  oop mirror             = java_lang_reflect_Method::clazz(method_mirror);
  int slot               = java_lang_reflect_Method::slot(method_mirror);
  bool override          = java_lang_reflect_Method::override(method_mirror) != 0;
  objArrayHandle ptypes(THREAD, objArrayOop(java_lang_reflect_Method::parameter_types(method_mirror)));

  oop return_type_mirror = java_lang_reflect_Method::return_type(method_mirror);
  BasicType rtype;
  if (java_lang_Class::is_primitive(return_type_mirror)) {
    rtype = basic_type_mirror_to_basic_type(return_type_mirror, CHECK_NULL);
  } else {
    rtype = T_OBJECT;
  }

  instanceKlassHandle klass(THREAD, java_lang_Class::as_Klass(mirror));
  Method* m = klass->method_with_idnum(slot);
  if (m == NULL) {
    THROW_MSG_0(vmSymbols::java_lang_InternalError(), "invoke");
  }
  methodHandle method(THREAD, m);

  return invoke(klass, method, receiver, override, ptypes, rtype, args, true, THREAD);
}
```

4. Java版的实现

   Java版MethodAccessor的生成使用MethodAccessorGenerator实现  下面是开头的注释：

```java
/** Generator for sun.reflect.MethodAccessor and
    sun.reflect.ConstructorAccessor objects using bytecodes to
    implement reflection. A java.lang.reflect.Method or
    java.lang.reflect.Constructor object can delegate its invoke or
    newInstance method to an accessor using native code or to one
    generated by this class. (Methods and Constructors were merged
    together in this class to ensure maximum code sharing.) */
```
