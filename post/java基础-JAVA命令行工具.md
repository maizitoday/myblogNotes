---
title:       "JAVA命令行工具"
subtitle:    ""
description: "vmArgs，args，springboot启动优化，-Xmx,-Xmn"
date:        2019-07-17
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["java基础", "JAVA命令行工具"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：https://www.jianshu.com/p/87637b150026   作者：Hypercube **

# 总述

`java`命令用来启动一个JAVA应用。有以下两种用法:

```java
 java [options] mainclass [args...] 
 java [options] -jar jarfile [args...]
 
 /**
  options
      由空格分隔的命令行选项，
  mainclass
      待启动类包含包路径的类全名，其中需要含有`main()`方法。
  jarfile
      待启动jar包的路径，其中需要`manifest`文件指明含有`main()`方法的启动类。
  args
      传给启动类`main()`方法的参数，由空格分隔。
 */
```

第一种从指定JAVA类启动，第二种从可运行jar包启动。启动过程有三步，首先启动JAVA运行时环境JRE，然后加载所需的类，最后调用类的`main()`方法。正确的`main()`方法形式如下：

```java
    public static void main(String[] args)
```

# 命令行选项 

java的命令行选项分为三类：

## 标准选项

JVM必须实现的选项，实现通用的功能，比如：检查JRE的版本，设置类路径等。 

### -version

打印版本信息然后退出。经常使用该选项来查看JAVA的版本或验证java命令是否可用。

### jps

**作用：**显示当前系统用户的Java进程情况及其Id号

```java
68341 BootLanguagServerBootApp
49574 Jps
68287 org.eclipse.equinox.launcher_1.5.600.v20191014-2022.jar
```

### jstack(查看线程)

```java
jstack  68341
```



### jmap(查看内存)

```java
jmap  68341
```



### jstat(性能分析)

说明，这些直接可以用VisualVM 工具来查看处理。 或者下面的命令直接查看。 



### jconsole

jconsole是jdk自带的工具，[使用jconsole工具来监控java运行情况]

```java
jconsole 68341
```

![Xnip2019-11-16_18-15-47](/img/Xnip2019-11-16_18-15-47.png)



### -Dproperty=value

指定一个系统属性值。属性和属性值都为字符形式，其中属性名不能含有空白字符，属性值如果需要空白字符，需要使用双引号`"`包裹。一个正确的示例如下：

```java
    -Dfoo="foo bar"
    该值可以在JAVA程序中使用如下代码获取：
    System.getProperty("foo")
```



## 扩展选项

HotSpot实现常用功能的选项，其他JVM不一定实现。**此类选项前缀为`-X`。**

### -Xmx size

指定堆的最大大小。指定2GB的最大堆，可用如下的任意方式：

```java
    -Xmx2G -Xmx2048m -Xmx2097152K -Xmx2147483648
```



### -Xms size

指定堆的初始化大小。指定1GB的初始堆，可用如下的任意方式：



### -Xmn size

指定**堆中新生代**的初始化和最大大小。指定256MB的新生代，可用如下的任意方式：

```java
 -Xmn256m -Xmn262144k -Xmn268435456
```

如果将新生代的值设置过大，那么一次新生代GC的时间就会过长；如果将新生代的值设置过小，又会引起频繁新生代GC。**官方建议该值设置在堆最大值的25%-50%区间内。**



### -Xss size

指定线程栈的大小。指定1MB的栈大小，可用如下的任意方式:

```java
-Xss1m -Xss1024k -Xss1048576
```

该选项的默认值随操作系统不同而不同：Linux环境下默认为1MB，Windows环境则取决于虚拟内存的大小。

 

## 高级选项

高级选项是开发者选项，不保证在所有JVM上被实现，并可能会改变。有以下四种类型：

1. 高级运行时选项：控制JVM运行时行为。
2. 高级维护性选项：支持收集系统信息和调试。
3. 高级GC选项：控制JVM的垃圾收集行为。
4. 高级JIT选项：控制JVM的及时编译行为。

设置选项值时，如果值为布尔`Boolean`型，表示开启或关闭某一功能。使用加号`+`表示开启某一功能，减号`-`表示关闭某一功能。示例如下：

```java
  -XX:+OptionName -XX:-OptionaName		
```

如果选项需要参数，可在选项之后跟随参数或使用特定分隔符隔开。常见的分隔方式有：空白字符分隔、冒号`：`分隔和等号分隔`=`。一些示例如下：

```java
-Xmx2G // 不分隔
-cp a.jar // 空白字符分隔
-agentlib:foo // 冒号分隔
-XX:ThreadStackSize=size // 等号分隔
```

如果参数值表示字节大小，可以直接使用整数或使用一些后缀，如`g`和`G`、`m`和`M`、`k`和`K`。

###  -XX:MaxHeapSize=size

   设置最大堆大小，见`-Xmx`。

###  -XX:InitialHeapSize=size

   设置初始堆大小，见`-Xms`。

###  -XX:NewSize

   设置新生代初始大小，见`-Xmn`。

###  -XX:MaxNewSize=size

   设置新生代最大大小，见`-Xmn`。

###  -XX:ThreadStackSize=size

   设置线程栈大小，见`-Xss`。

###  -XX:MaxMetaspaceSize=size

   设置永久代大小，JDK8以下使用`-XX:MaxPermSize=size`。



## 高级可维护性选项

### -XX:+HeapDumpOnOutOfMemoryError

开启堆转储功能。当Java应用抛出`OutOfMemoryError`异常时可以使用堆转储工具（HPROF）将Java堆存储到文件中。默认文件存储在当前路径下，文件名为java_pid[pid].hprof。此外，还可使用选项`-XX:HeapDumpPath=path`指定文件地址。一个示例如下(其中%p表示进程PID)：

```java
   -XX:HeapDumpPath=/data/log/java_pid%p.hprof
```



### -XX:OnOutOfMemoryError=string

设置发生`OutOfMemoryError`后执行的处理命令，多个命令可使用分号`;`分隔。如果命令中含有空格，需要使用双引号`"`包裹命令。



### -XX:ErrorFile=filename

指定Java进程发生不可恢复的错误时，保存错误文件的地址和文件名。默认情况下，错误文件存储在当前路径下，文件名为hs_err_pid[pid].log。一个示例如下：

```java
-XX:ErrorFile=/data/log/java_error_%p.log
```

如果文件由于权限问题、硬盘已满或是其他问题而不能在指定目录下创建，那么该文件将创建在临时目录下。对于Linux环境，临时目录为/tmp；对于Windows环境，临时目录由环境变量`TMP`指定，如果该环境变量没有指定，那么将使用`TEMP`环境变量的值。

### -XX:LogFile=path

设置日志文件的地址。默认情况下，文件被写到当前目录下，文件名为hotspot.log。一个示例如下：

```java
 -XX:LogFile=/data/log/hotspot.log
```



# 执行指定的JDK版本运行jar

```
#!/bin/bash
JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_101.jdk/Contents/Home
JAVA=$JAVA_HOME/bin/java
nohup $JAVA -jar zookeeper-visualizer.jar -Djava.ext.dirs=$JAVA_HOME/lib &
```

# -D

**原文链接：https://blog.csdn.net/cyony/article/details/74375251**

-D 是设置系统的属性

## java.ext.dirs

java命令引入jar时可以-cp参数，但时-cp不能用通配符(多个jar时什么烦要一个个写,不能*.jar)，面通常的jar都在同一目录，且多于1个。前些日子找到（发现）-Djava.ext.dirs太好。

-Djava.ext.dirs会覆盖Java本身的ext设置，java.ext.dirs指定的目录由ExtClassLoader加载器加载，如果您的程序没有指定该系统属性，那么该加载器默认加载$JAVA_HOME/jre/lib/ext目录下的所有jar文件。但如果你手动指定系统属性且忘了把$JAVA_HOME/jre/lib/ext路径给加上，那么ExtClassLoader不会去加载$JAVA_HOME/lib/ext下面的jar文件，这意味着你将失去一些功能，例如java自带的加解密算法实现。

# 注意

还有很多命令请查看这篇原文， 上面主要记录常用的命令。 





**-XX:PermSize=64M -XX:MaxPermSize**

-XX:PermSize分配非堆最小内存，默认为物理内存的1/64；-XX:MaxPermSize分配非堆最大内存，默认为物理内存的1/4。



- Cmd + Shift + F（Win 用户是 Ctrl + Shift +F）：在全局的文件夹中进行搜索。效果如下：







cloudfoundry-manifest.ls.java.vmargs

spring-boot.ls.java.vmargs



```json
"spring-boot.ls.java.vmargs": [
  "-Xms100m",
  "-Xmx300m",
  "-XX:CompressedClassSpaceSize=128m",
  "-XX:MetaspaceSize=200m",
  "-XX:MaxMetaspaceSize=200m"
],
"cloudfoundry-manifest.ls.java.vmargs": [
  "-Xms100m",
  "-Xmx300m",
  "-XX:CompressedClassSpaceSize=128m",
  "-XX:MetaspaceSize=200m",
  "-XX:MaxMetaspaceSize=200m"
],
```





