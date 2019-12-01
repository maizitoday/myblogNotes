---
title:       "classpath理解，时间戳"
subtitle:    ""
description: ""
date:        2019-06-29
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["java基础", "classpath理解"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：**https://blog.csdn.net/weixin_40171603/article/details/81301687**

**转载地址：https://blog.csdn.net/weixin_40171603/article/details/81301687**

# 理解 java classpath

classpath 是一个命令行参数，或环境变量，它告诉 Java 虚拟机或者 Java 编译器到哪里寻找用户定义的类和包。JVM（java虚拟机）依据CLASSPATH中的路径信息来寻找可执行指令（.class文件）。 

# 概述

类似于经典的动态加载的行为，执行 Java 程序时，Java 虚拟机发现并进行懒加载类。classpath 告诉 Java 在哪里查找文件系统中定义这些类的文件。

虚拟机按如下顺序进行查找：

1. bootstrap classes ：这些类是 JAVA 平台的基础类，例如 java.lang.Object
2. extension classes ：包是在 JRE 或 JDK，jre/lib/ext 目录下
3. 用户自定义类和类库

**默认情况下的 JDK 的标准 API 和扩展包，无需设置在哪里可以找到他们。所有用户定义的包和库的路径必须在命令行中进行设置。**

使用 -classpath 选项可以覆盖 CLASSPATH 环境变量，如果不声明，则当前目录作为 classpath。这就意味着如果我们当前目录是 `D:\myprogram\` , 我们就不需要显式地指定类路径。 **linux环境下面是用 java -cp 命令重置classpath路径**

# path与classpath的区别及其意义

![Xnip2019-06-30_15-24-49](/img/Xnip2019-06-30_15-24-49.png)

配置path路径，是为了让系统知道你要用的命令在哪里（省去每次执行命令都要先定位到可执行文件所在目录，然后再执行命令这一麻烦步骤）

rt.jar是JAVA基础类库，dt.jar是关于运行环境的类库，tools.jar是工具类库 设置在classpath里是为了让jvm能根据路径找到这些所需的依赖。

CLASSPATH环境变量。作用是指定类搜索路径，要使用已经编写好的类，前提当然是能够找到它们了，JVM就是通过CLASSPATH来寻找类的.class文件

**总而言之，path是Windows查找.exe文件的路径；classpath是jvm查找.class文件的路径**

# classpath 和 classpath*

classpath：只会到你的class路径中查找找文件;
classpath*：不仅包含class路径，还包括jar文件中(class路径)进行查找。

# 时间戳

时间戳是指格林威治时间1970年01月01日00时00分00秒(北京时间1970年01月01日08时00分00秒)起至现在的总秒数。 通俗的讲， 时间戳是一份能够表示一份数据在一个特定时间点已经存在的完整的可验证的数据。

时间戳就是一种类型，只是精度很高，比datetime要精确的多，通常用来防止数据出现脏读现象 

