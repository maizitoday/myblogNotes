---
title:       "Runtime"
subtitle:    ""
description: ""
date:        2020-03-01
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["java基础", "Runtime"]
categories:  ["Tech" ]
---

**转载地址：https://blog.csdn.net/tomcmd/article/details/48971051?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task**

# 了解Runtime类

Runtime:运行时，是一个封装了JVM的类。每一个JAVA程序实际上都是启动了一个JVM进程，每一个JVM进程都对应一个Runtime实例，此实例是由JVM为其实例化的。所以我们不能实例化一个Runtime对象，应用程序也不能创建自己的 Runtime 类实例，但可以通过 getRuntime 方法获取当前Runtime运行时对象的引用。一旦得到了一个当前的Runtime对象的引用，就可以调用Runtime对象的方法去控制Java虚拟机的状态和行为。
查看官方文档可以看到，Runtime类中没有构造方法，本类的构造方法被私有化了， 所以才会有getRuntime方法返回本来的实例化对象，这与单例设计模式不谋而合

```java
public static Runtime getRuntime()
```

# 使用Runtime获取JVM的空间信息

**1.  得到JVM信息**

每一个Runtime对象都是JVM实例化的，所以可以通过Runtime类取得相关信息

```java
public class RuntimeDemo01{
	public static void main(String[] args){
		Runtime run = Runtime.getRuntime();	//通过Runtime类的静态方法获取Runtime类的实例
		System.out.println("JVM最大内存量："+run.maxMemory());
		System.out.println("JVM空闲内存量："+run.freeMemory());
	}
}
```

**2. public void gc()**

```java
public class RuntimeDemo01{
	public static void main(String[] args){
		Runtime run = Runtime.getRuntime();	//通过Runtime类的静态方法获取Runtime类的实例
		System.out.println("JVM最大内存量："+run.maxMemory());
		System.out.println("JVM空闲内存量："+run.freeMemory());
		String str = "Hello"+"World";
		System.out.println(str);
		for(int i=0;i<2000;i++){
			str = str + i;
		}
		System.out.println("操作String之后的JVM空闲内存量："+run.freeMemory());
		run.gc();
		System.out.println("垃圾回收之后的JVM空闲内存量："+run.freeMemory());
	}
}
```

**3. Runtime类与Process类**

除了观察内存使用量之外，还可以直接使用Runtime类运行**本机的可执行程序**。

```java
public Process exec(String command)throwsIOException
```

例如：调用记事本：

```java
public class RuntimeDemo03{
	public static void main(String[] args){
		Runtime run = Runtime.getRuntime();	//通过Runtime类的静态方法获取Runtime类的实例
		try{
			run.exec("notepad.exe");	//调用本机程序，此方法需要进行异常处理
		}catch(Exception e){
			e.printStackTrace();
		}		
	}
}
```

exec()方法的返回值是Process类，Process类也有一些方法可以使用，比如结束一个进程，通过destroy()结束

# 注意

Runtime类本身就是单例设计模式的一种应用，因为整个JVM中只存在一个Runtime类的对象，可以使用Runtime类取得JVM的系统信息，或者使用gc()方法释放掉垃圾空间，还可以运行本机的程序。
 