---
title:       "java基础-jvm"
subtitle:    ""
description: ""
date:        2019-04-22
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["java基础", "jvm", "虚拟机", "jvm调优"]
categories:  ["Tech" ]
---

[TOC]

**说明： 本学习主要对尚硅谷周阳大神《jvm视频》记录，一下文字来源这个视频PPT**

# JVM体系结构概览



![4-23-1](/img/4-23-1.png)

JVM是运行在操作系统之上的，它和硬件没有直接的交互。

负责加载class文件，class文件在文件开头有特定的文件标示，并且ClassLoader只负责class文件的加载，至于它是否可以运行，则由ExecutionEngine决定。**Execution Engine*** 执行引擎负责解释命令，提交操作系统执行。

## 类转载器

![4-23-2](/img/4-23-2.png)

当我们在IDEA工具创建一个Test.java类时候， main运行的时候， 为什么可以运行？

因为我们在搭建java环境的时候，我们配置环境的时候，可以看到 $JAVAHOME/jre/lib/rt.jar 和   $JAVAHOME/jre/lib/ext/*.jar,需要相关的类，java虚拟机已经帮你写好了，也就是说他的父类和相关类都已经帮你写好了，这些加载器，已经把这些类已经加载到了JVM中了。

$CLASSPATH,我们在配置环境的时候CLASSPATH=.;%JAVA_HOME%\lib;%JAVA_HOME%\lib\tools.jar， 可以看到前面的分好，他回去搜索当前的目录，查找出所有的class文件，加载进去到JVM机。

自定义加载器一般很少重写。

**这样，我们创建的java文件，就被这个快递员送到了虚拟机中了。** 



### 双亲委派机制 

原文：https://blog.csdn.net/start_lie/article/details/79016312 

双亲委派模型的式作过程是：如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成，每一个层次的类加载器都是如此，因此所有的加载请求最终都应该传送到顶层的启动类加载器中，只有当父加载器反馈自己无法完全这个加载请求时，子加载器才会尝试自己去加载。



### 沙箱安全机制

原文：https://blog.csdn.net/start_lie/article/details/79016312 

沙箱机制是由基于双亲委派机制上 采取的一种JVM的自我保护机制,假设你要写一个java.lang.String 的类,由于双亲委派机制的原理,此请求会先交给Bootstrap试图进行加载,但是Bootstrap在加载类时首先通过包和类名查找rt.jar中有没有该类,有则优先加载rt.jar包中的类,因此就保证了java的运行机制不会被破坏.

有了双亲委派机制的前提，在沙箱安全机制下，黑客自定义的`java.lang.String`类永远都不会被加载进内存。因为首先是最顶端的类加载器加载系统的`java.lang.String`类，最终自定义的类加载器无法加载`java.lang.String`类，防止java源码被恶心修改。



### ClassLoader

原文：https://blog.csdn.net/briblue/article/details/54973413

ClassLoader翻译过来就是类加载器，普通的java开发者其实用到的不多，但对于某些框架开发者来说却非常常见。理解ClassLoader的加载机制，也有利于我们编写出更高效的代码。ClassLoader的具体作用就是将class文件加载到jvm虚拟机中去，程序就可以正确运行了。但是，jvm启动的时候，并不会一次性加载所有的class文件，而是根据需要去动态加载。想想也是的，一次性加载那么多jar包那么多class，那内存不崩溃。

#### 自定义ClassLoader

不知道大家有没有发现，不管是Bootstrap ClassLoader还是ExtClassLoader等，这些类加载器都只是加载指定的目录下的jar包或者资源。如果在某种情况下，我们需要动态加载一些东西呢？比如从D盘某个文件夹加载一个class文件，或者从网络上下载class主内容然后再进行加载，这样可以吗？

如果要这样做的话，需要我们自定义一个classloader。 

#### 自定义ClassLoader还能做什么

突破了JDK系统内置加载路径的限制之后，我们就可以编写自定义ClassLoader，然后剩下的就叫给开发者你自己了。你可以按照自己的意愿进行业务的定制，将ClassLoader玩出花样来。

常见的用法是将Class文件按照某种加密手段进行加密，然后按照规则编写自定义的ClassLoader进行解密，这样我们就可以在程序中加载特定了类，并且这个类只能被我们自定义的加载器进行加载，提高了程序的安全性。



### JAVA类加载流程

原文：https://blog.csdn.net/briblue/article/details/54973413

Java语言系统自带有三个类加载器:

1. Bootstrap ClassLoader 最顶层的加载类，主要加载核心类库，%JRE_HOME%\lib下的rt.jar、resources.jar、charsets.jar和class等。另外需要注意的是可以通过启动jvm时指定-Xbootclasspath和路径来改变Bootstrap ClassLoader的加载目录。比如java -Xbootclasspath/a:path被指定的文件追加到默认的bootstrap路径中。我们可以打开我的电脑，在上面的目录下查看，看看这些jar包是不是存在于这个目录。

2. **Extention ClassLoader** 扩展的类加载器，加载目录%JRE_HOME%\lib\ext目录下的jar包和class文件。还可以加载`-D java.ext.dirs`选项指定的目录。

3. **Appclass Loader也称为SystemAppClass** 加载当前应用的classpath的所有类。系统加载器， 来进行加载classpath里面的所有的jar的包。 所以以前我们是要把包都放到到classpath里面去的。 

   

我们看到了系统的3个类加载器，但我们可能不知道具体哪个先行呢？我可以先告诉你答案

1. Bootstrap CLassloder
2. Extention ClassLoader
3. AppClassLoader

 说明了它们加载的路径。并且还提到了`-Xbootclasspath`和`-D java.ext.dirs`这两个虚拟机参数选项。

### 类的加载方式

![Xnip2019-11-23_16-29-29](/img/Xnip2019-11-23_16-29-29.png)

### 主动加载类

![Xnip2019-11-23_16-07-56](/img/Xnip2019-11-23_16-07-56.png)



## native

用于，Java平台和本地C代码进行互操作的关键字，他主要和C对接，但是我们现在有更好的对接方式， 比如webService。这个是由于前期java用C/C++来写的，这个历史背景下面有了这个native关键字。



## 程序计数器(PC)

计数和调度用：每个线程都有一个程序计数器，是线程私有的,就是一个指针，指向方法区中的方法字节码（用来存储指向下一条指令的地址,也即将要执行的指令代码），由执行引擎读取下一条指令，是一个非常小的内存空间，几乎可以忽略不记。

比如： 我们controller调用Server，在调用Dao，他就是一条这样的指引，按照顺序，一个钩子一个钩子的挂住，有顺序的执行。他就是严格执行这个顺序的调度器。



## 方法区(Method Area)

简单说，所有定义的方法的信息都保存在该区域，此区属于共享区间。 静态变量+常量+类信息(构造方法/接口定义)+运行时常量池存在方法区中。实例变量存在堆内存中,和方法区无关。**也就是一个类的模板，相当于一个设计图，一个抽象类。**



实际而言，方法区（Method Area）和堆一样，是各个线程共享的内存区域，它用于存储虚拟机加载的：类信息+普通常量+静态常量+编译器编译后的代码等等，虽然JVM规范将方法区描述为堆的一个逻辑部分，但它却还有一个别名叫做Non-Heap(非堆)，目的就是要和堆分开。



## 栈区

首先栈区就是一个子弹夹，后进先出。

```java
public class Main {

    public void test01()
    {
        System.out.println("  test01  栈顶 ");
    }

    public void test02()
    {
        System.out.println("  test02   ");
    }

    public static void main(String[] args){
        Main  main = new Main();
        main.test01();
        main.test02();
        System.out.println("  main 栈区就是栈低 ");
    }
}
/**
  从这里可以看出， 执行的时候，最新执行的，test01执行后，出栈，然后test02出栈，然后main主线程出栈，
  程序结束。
**/
```

### ***Stack*** 栈是什么

栈也叫栈内存，主管Java程序的运行，是在线程创建时创建，它的生命期是跟随线程的生命期，线程结束栈内存也就释放，**对于栈来说不存在垃圾回收问题**，只要线程一结束该栈就Over，生命周期和线程一致，是线程私有的。8种基本类型的变量+对象的引用变量+实例方法都是在函数的栈内存中分配，**需要注意这个栈也就是子弹夹内存是有限的。**

### 栈存储什么

栈帧中主要保存3 类数据：

本地变量（Local Variables）:输入参数和输出参数以及方法内的变量；

栈操作（Operand Stack）:记录出栈、入栈的操作；

栈帧数据（Frame Data）:包括类文件、方法等等。

###  栈运行原理

栈中的数据都是以栈帧（Stack Frame）的格式存在，栈帧是一个内存区块，是一个数据集，是一个有关方法(Method)和运行期数据的数据集，当一个方法A被调用时就产生了一个栈帧 F1，并被压入到栈中，

A方法又调用了 B方法，于是产生栈帧 F2 也被压入栈，

B方法又调用了 C方法，于是产生栈帧 F3 也被压入栈，

……

执行完毕后，先弹出F3栈帧，再弹出F2栈帧，再弹出F1栈帧……

遵循“先进后出”/“后进先出”原则。

**注意这个地方只是一种声明的：调用**



### 运行流程图

![4-23-3](/img/4-23-3.png)



### 栈，堆，方法区交互关系

![4-23-4](/img/4-23-4.png)



## 堆区

存放所有new出来的对象；对象的引用放入到栈区。

![4-23-5](/img/4-23-5.png)

### 永久区：(1.7还有这个，1.8之后就是元空间) 



![4-23-6](/img/4-23-6.png)

###  第三方包直接放入永久区

![4-23-7](/img/4-23-7.png)



### 方法区和永久代区别

![4-23-8](/img/4-23-8.png)



## 堆参数调优

![4-23-9](/img/4-23-9.png)

-Xms1024m  -Xmx1024m : 这个地方是设置新生区和养老区      -XX:+PrintGCDetails:输出dump文件，用来分析内存。

-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/img/

设置如果出现OOM，把对应的hprof文件给输出，打开这个堆内存进行分析。 

**如下可以查看到，在 Full GC后，就报OOM错误了，同时查看jdk1.8是Metaspace元空间。**

![4-23-10](/img/4-23-10.png)





## GC

### GC算法

 ![4-23-11](/img/4-23-11.png)



###  GC什么时候调用

也就是GC的触发条件，eden 满了minor gc，升到老年代的对象大于老年代剩余空间full gc，或者小于时被HandlePromotionFailure参数强制full gc；gc与非gc时间耗时超过了GCTimeRatio的限制引发OOM，调优诸如通过NewRatio控制新生代老年代比例，通过 MaxTenuringThreshold控制进入老年前生存次数等。

## throwable和Exception的区别

如果是堆内存溢出了， 只有throwable能捕捉到，因为Exception内存不够用了，他是Exception的父类。

### Exception效果

![4-23-12](/img/4-23-12.png)



### throwable效果

![4-23-13](/img/4-23-13.png)





 

