---
title:       "面试题系列-Java基础"
subtitle:    ""
description: "java基础问题面试题收集"
date:        2020-06-01
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["面试题系列","java基础"]
categories:  ["Tech" ]
---

[TOC]



**说明：以下文字来源网络各相关面试题目收集**

# java版本差异对比

转载：https://mp.weixin.qq.com/s/zX_O-uv8RKRh1K6pI9s8ig?

![16ca4e326c3132c8](/img/16ca4e326c3132c8.png)

最近jdk14的相关文章：https://mp.weixin.qq.com/s/WjTz1-jSIeFd4c06NewxPg

如今OracleJdk是要开发免费，上线的时候就收费的了， 我们可以用https://adoptopenjdk.net/ 这个地址的进行下载最新jdk进行开发，他是免费的。 





# final类型的数据， 可以进行反射修改其中的值否

转载地址：https://www.jianshu.com/p/50830768bd52  



# 关于 JVM JDK 和 JRE 最详细通俗的解答

Java 虚拟机（JVM）是运行 Java 字节码的虚拟机。JVM 有针对不同系 统的特定实现（Windows，Linux，macOS），目的是使用相同的字节码，它们 都会给出相同的结果。字节码和不同系统的 JVM 实现是 Java 语言“一次编 译，随处可以运行”的关键所在。



## JDK 和 JRE

JDK 是 Java Development Kit，它是功能齐全的 Java SDK。它拥有 JRE 所拥有的一切，还有编译器（javac）和工具（如 javadoc 和 jdb）。它能够创建和编 译程序。

JRE 是 Java 运行时环境。它是运行已编译 Java 程序所需的所有内容的集合， 包括 Java 虚拟机（JVM），Java 类库，java 命令和其他的一些基础构件。但 是，它不能用于创建新程序。

如果你只是为了运行一下 Java 程序的话，那么你只需要安装 JRE 就可以了。 如果你需要进行一些 Java 编程方面的工作，那么你就需要安装 JDK 了。但 是，这不是绝对的。有时，即使您不打算在计算机上进行任何 Java 开发，仍然需要安装 JDK。例如，如果要使用 JSP 部署 Web 应用程序，那么从技术上讲， 您只是在应用程序服务器中运行 Java 程序。那你为什么需要 JDK 呢？因为应用 程序服务器会将 JSP 转换为 Java servlet，并且需要使用 JDK 来编译 servlet。



# Oracle JDK 和 OpenJDK 的对比

对于 Java 7，没什么关键的地方。OpenJDK 项目主要基于 Sun 捐赠的 HotSpot 源代码。此外，OpenJDK 被选为 Java 7 的参考实现，由 Oracle 工程师维护。 关于 JVM，JDK，JRE 和 OpenJDK 之间的区别：

1. Oracle JDK 版本将每三年发布一次，而 OpenJDK 版本每三个月发布一次；

2. OpenJDK 是一个参考模型并且是完全开源的，而 Oracle JDK 是 OpenJDK 的一个实现，并不是完全开源的；

3. Oracle JDK 比 OpenJDK 更稳定。OpenJDK 和 Oracle JDK 的代码几乎 相同，但 Oracle JDK 有更多的类和一些错误修复。因此，如果您想开发 企业/商业软件，我建议您选择 Oracle JDK，因为它经过了彻底的测试和 稳定。某些情况下，有些人提到在使用 OpenJDK 可能会遇到了许多应 用程序崩溃的问题，但是，只需切换到 Oracle JDK 就可以解决问题；

4. 顶级公司正在使用 Oracle JDK，例如 Android Studio，Minecraft 和 IntelliJ IDEA 开发工具，其中 Open JDK 不太受欢迎；

5. 在响应性和 JVM 性能方面，Oracle JDK 与 OpenJDK 相比提供了更好的 性能；

6. Oracle JDK 不会为即将发布的版本提供长期支持，用户每次都必须通过

   更新到最新版本获得支持来获取最新版本；

7. Oracle JDK 根据二进制代码许可协议获得许可，而 OpenJDK 根据 GPL v2 许可获得许可。



# String StringBuffer 和 StringBuilder 的区别 是什么 String 为什么是不可变的



## 可变性

简单的来说：String 类中使用 final 关键字字符数组保存字符串，private　 final　char　value[]，所以 String 对象是不可变的。而 StringBuilder 与 StringBuffer 都继承自 AbstractStringBuilder 类，在 AbstractStringBuilder 中 也是使用字符数组保存字符串 char[]value 但是没有用 final 关键字修饰，所以 这两种对象都是可变的。



## 线程安全性

String 中的对象是不可变的，也就可以理解为常量，线程安全。

String 中的对象是不可变的，也就可以理解为常量，线程安全。 AbstractStringBuilder 是 StringBuilder 与 StringBuffer 的公共父类，定义了 一些字符串的基本操作，如 expandCapacity、append、insert、indexOf 等公 共方法。**StringBuffer 对方法加了同步锁或者对调用的方法加了同步锁，所以 是线程安全的。StringBuilder 并没有对方法进行加同步锁，所以是非线程安全 的。**



## 性能

每次对 String 类型进行改变的时候，都会生成一个新的 String 对象，然后将 指针指向新的 String 对象。StringBuffer 每次都会对 StringBuffer 对象本身进行操作，而不是生成新的对象并改变对象引用。相同情况下使用 StringBuilder 相比使用 StringBuffer 仅能获得 10%~15% 左右的性能提升， 但却要冒多线程不安全的风险。



# 在一个静态方法内调用一个非静态成员为什么是非法的

由于静态方法可以不通过对象进行调用，因此在静态方法里，不能调用其他非 静态变量，也不可以访问非静态变量成员。



# 在 Java 中定义一个不做事且没有参数的构造方法的作用

Java 程序在执行子类的构造方法之前，如果没有用 super() 来调用父类特定 的构造方法，则会调用父类中“没有参数的构造方法”。因此，如果父类中只定 义了有参数的构造方法，而在子类的构造方法中又没有用 super() 来调用父类 中特定的构造方法，则编译时将发生错误，因为 Java 程序在父类中找不到没 有参数的构造方法可供执行。解决办法是在父类里加上一个不做事且没有参数 的构造方法。 　



# == 与 equals

== : 它的作用是判断两个对象的地址是不是相等。即，判断两个对象是不是同 一个对象。(基本数据类型==比较的是值，引用数据类型==比较的是内存地址)

equals() : 它的作用也是判断两个对象是否相等。但它一般有两种使用情况：

**情况 1：类没有覆盖 equals() 方法。则通过 equals() 比较该类的两个 对象时，等价于通过“==”比较这两个对象。**

情况 2：类覆盖了 equals() 方法。一般，我们都覆盖 equals() 方法来 两个对象的内容相等；若它们的内容相等，则返回 true (即，认为这两 个对象相等)。

**注意：String 中的 equals 方法是被重写过的，因为 object 的 equals 方法是 比较的对象的内存地址，而 String 的 equals 方法比较的是对象的值。当创建 String 类型的对象时，虚拟机会在常量池中查找有没有已经存 在的值和要创建的值相同的对象，如果有就把它赋给当前引用。如果没 有就在常量池中重新创建一个 String 对象。** 



# hashCode 与 equals



## hashCode

hashCode() 的作用是获取哈希码，也称为散列码；它实际上是返回一个 int 整 数。这个哈希码的作用是确定该对象在哈希表中的索引位置。hashCode() 定义 在 JDK 的 Object.java 中，这就意味着 Java 中的任何类都包含有 hashCode() 函 数。

**散列表存储的是键值对(key-value)，它的特点是：能根据“键”快速的检索出对应 的“值”。这其中就利用到了散列码！（可以快速找到所需要的对象）**



## 为什么要有 hashCode

我们以“HashSet 如何检查重复”为例子来说明为什么要有 hashCode：

当你把对象加入 HashSet 时，HashSet 会先计算对象的 hashcode 值来判断 对象加入的位置，同时也会与其他已经加入的对象的 hashcode 值作比较，如 果没有相符的 hashcode，HashSet 会假设对象没有重复出现。但是如果发现有 相同 hashcode 值的对象，这时会调用 equals（）方法来检查 hashcode 相 等的对象是否真的相同。如果两者相同，HashSet 就不会让其加入操作成功。 如果不同的话，就会重新散列到其他位置。



## hashCode() 与 equals()的相关规定

1. 如果两个对象相等，则 hashcode 一定也是相同的
2. 两个对象相等,对两个对象分别调用 equals 方法都返回 true
3. 两个对象有相同的 hashcode 值，它们也不一定是相等的
4. **因此，equals 方法被覆盖过，则 hashCode 方法也必须被覆盖**
5. hashCode() 的默认行为是对堆上的对象产生独特值。如果没有重写 hashCode()，则该 class 的两个对象无论如何都不会相等（即使这两个 对象指向相同的数据）



# 简述线程，程序、进程的基本概念。以及他们之 间关系是什么

线程与进程相似，但线程是一个比进程更小的执行单位。一个进程在其执行的 过程中可以产生多个线程。与进程不同的是同类的多个线程共享同一块内存空 间和一组系统资源，所以系统在产生一个线程，或是在各个线程之间作切换工 作时，负担要比进程小得多，也正因为如此，线程也被称为轻量级进程。

程序是含有指令和数据的文件，被存储在磁盘或其他的数据存储设备中，也就 是说程序是静态的代码。

进程是程序的一次执行过程，是系统运行程序的基本单位，因此进程是动态 的。系统运行一个程序即是一个进程从创建，运行到消亡的过程。简单来说， 一个进程就是一个执行中的程序，它在计算机中一个指令接着一个指令地执行着，同时，每个进程还占有某些系统资源如 CPU 时间，内存空间，文件，文件，输入输出设备的使用权等等。换句话说，当程序在执行时，将会被操作系 统载入内存中。 线程是进程划分成的更小的运行单位。线程和进程最大的不同 在于基本上各进程是独立的，而各线程则不一定，因为同一进程中的线程极有 可能会相互影响。从另一角度来说，进程属于操作系统的范畴，主要是同一段时间内，可以同时执行一个以上的程序，而线程则是在同一程序内几乎同时执 行一个以上的程序段。



# Java 异常类层次结构图

![Xnip2020-06-01_12-12-34](/img/Xnip2020-06-01_12-12-34.png)

在 Java 中，所有的异常都有一个共同的祖先 java.lang 包中的 Throwable 类。Throwable： 有两个重要的子类：Exception（异常） 和 Error（错 误） ，二者都是 Java 异常处理的重要子类，各自都包含大量子类。



## Error（错误）

是程序无法处理的错误，表示运行应用程序中较严重问题。大 多数错误与代码编写者执行的操作无关，而表示代码运行时 JVM（Java 虚拟 机）出现的问题。例如，Java 虚拟机运行错误（Virtual MachineError），当 JVM 不再有继续执行操作所需的内存资源时，将出现 OutOfMemoryError。这 些异常发生时，Java 虚拟机（JVM）一般会选择线程终止。



## Exception（异常）

是程序本身可以处理的异常。

Exception 类有一个重要的 子类 RuntimeException。RuntimeException 异常由 Java 虚拟机抛出。



## 异常处理总结

1. try 块：用于捕获异常。其后可接零个或多个 catch 块，如果没有 catch 块，则必须跟一个 finally 块。

2. catch 块：用于处理 try 捕获到的异常。

3. finally 块：无论是否捕获或处理异常，finally 块里的语句都会被执行。 当在 try 块或 catch 块中遇到 return 语句时，finally 语句块将在方法返回 之前被执行。

   

**注意：异常和错误的区别：异常能被程序本身可以处理，错误是无法处理。**



# Java 序列化中如果有些字段不想进行序列化 怎么办

对于不想进行序列化的变量，使用 transient 关键字修饰。

transient 关键字的作用是：阻止实例中那些用此关键字修饰的的变量序列化；

当对象被反序列化时，被 transient 修饰的变量值不会被持久化和恢复。 **transient 只能修饰变量，不能修饰类和方法。**





# 获取用键盘输入常用的的两种方法

方法 1：通过 Scanner

```java
Scanner input = new Scanner(System.in);

String s = input.nextLine();

input.close();
```



方法 2：通过 BufferedReader

```java
BufferedReader input = new BufferedReader(new InputStreamReader(System.in));
String s = input.readLine();
```



# 集合

## Map

Map主要用于存储健值对，根据键得到值，因此不允许键重复,但允许值重复。

### Hashmap

Hashmap 是一个 最常用的Map,它根据键的HashCode 值存储数据,根据键可以直接获取它的值，具有很快的访问速度。HashMap最多只允许一条记录的键为Null;允许多条记录的值为Null;、

HashMap不支持线程的同步，即任一时刻可以有多个线程同时写HashMap;可能会导致数据的不一致。如果需要同步，可以用 Collections的synchronizedMap方法使HashMap具有同步的能力.

### Hashtable

Hashtable 与 HashMap类似,不同的是:它不允许记录的键或者值为空;它支持线程的同步，即任一时刻只有一个线程能写Hashtable,因此也导致了Hashtale在写入时会比较慢。

### LinkedHashMap

LinkedHashMap保存了记录的插入顺序，在用Iterator遍历LinkedHashMap时，先得到的记录肯定是先插入的.在遍历的时候会比HashMap慢。

### TreeMap

TreeMap能够把它保存的记录根据键排序,默认是按升序排序，也可以指定排序的比较器，当用Iterator 遍历TreeMap时，得到的记录是排过序的。





























