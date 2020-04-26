---
title:       "mac-常用技巧和工具"
subtitle:    ""
description: "mac常用技巧，查看java所有版本，查看端口"
date:        2019-06-27
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["开发工具"]
categories:  ["Tech" ]
---

[TOC]

# 查看端口

```shell
 lsof -i tcp:port 
 （port替换成端口号，比如6379）可以查看该端口被什么程序占用，并显示PID，方便KILL（kill pid）
```

# 查看当前Mac中所有JDK

```shell
➜  blog /usr/libexec/java_home -V
Matching Java Virtual Machines (3):
    14.0.1, x86_64:	"Java SE 14.0.1"	/Library/Java/JavaVirtualMachines/jdk-14.0.1.jdk/Contents/Home
    1.8.0_101, x86_64:	"Java SE 8"	/Library/Java/JavaVirtualMachines/jdk1.8.0_101.jdk/Contents/Home
    1.7.0_71, x86_64:	"Java SE 7"	/Library/Java/JavaVirtualMachines/jdk1.7.0_71.jdk/Contents/Home

/Library/Java/JavaVirtualMachines/jdk-14.0.1.jdk/Contents/Home
➜  blog
```

可以看到当前安装了3个JDK版本， 最下面的jdk-14表示的是当前的版本。 

# Iterm工具

快捷键，马上显示出来

```
ctrl 按两下
```

