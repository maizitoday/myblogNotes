---
title:       "URL和URI的区别"
subtitle:    ""
description: "URL，URI，URN"
date:        2020-04-11
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["java相关概念", "URL和URI的区别"]
categories:  ["Tech" ]
---

[TOC]

**链接：https://www.zhihu.com/question/21950864/answer/154309494   作者：daixinye**

**链接：https://lovoedu.gitee.io/javablog/2017/11/22/20171122/** 

# 简介

统一资源标志符URI就是在某一规则下能把一个资源独一无二地标识出来。
拿人做例子，假设这个世界上所有人的名字都不能重复，那么名字就是URI的一个实例，通过名字这个字符串就可以标识出唯一的一个人。
现实当中名字当然是会重复的，所以身份证号才是URI，通过身份证号能让我们能且仅能确定一个人。
那统一资源定位符URL是什么呢。也拿人做例子然后跟HTTP的URL做类比，就可以有：

动物住址协议://地球/中国/浙江省/杭州市/西湖区/某大学/14号宿舍楼/525号寝/张三.人

可以看到，这个字符串同样标识出了唯一的一个人，起到了URI的作用，所以URL是URI的子集。URL是以描述人的位置来唯一确定一个人的。
在上文我们用身份证号也可以唯一确定一个人。对于这个在杭州的张三，我们也可以用：

身份证号：[123456789](tel:123456789)

来标识他。
所以不论是用定位的方式还是用编号的方式，我们都可以唯一确定一个人，都是URl的一种实现，而URL就是用定位的方式实现的URI。

# URL

URL：（全称：Uniform Resource Locator） 统一资源定位符。

它是对可以从互联网上得到的资源的位置和访问方法的一种简洁的表示，是互联网上标准资源的地址

URL 的常见定义格式为

scheme://host[:port#]/path/…/[;url-params][?query-string][#anchor]

```java
scheme //有我们很熟悉的http、https、ftp以及著名的ed2k，迅雷的thunder等。
host   //HTTP服务器的IP地址或者域名
port#  //HTTP服务器的默认端口是80，这种情况下端口号可以省略。如果使用了别的端口，必须指明，
例如tomcat的默认端口是8080  http://localhost:8080/
path   //访问资源的路径
url-params  //所带参数 
query-string    //发送给http服务器的数据
anchor //锚点定位
```

URL的格式一般由下列三部分组成:

1. 协议(或称为服务方式);
2. 存有该资源所在的服务器的名称或IP地址(包括端口号);
3. 主机资源的具体地址。



## 一个简单的url 



### 1 — 协议

```java
常见的协议
http     超文本传输协议资源
https    用安全套接字层传送的超文本传输协议
ftp      文件传输协议
mailto   电子邮件地址
```



### 2 — 存有该资源所在的服务器的名称或IP地址(包括端口号)

端口:相当于一种数据的传输通道。用于接受某些数据，然后传输给相应的服务，而电脑将这些数据处理后，再将相应的回复通过开启的端口传给对方。

端口的作用：因为 IP 地址与网络服务的关系是一对多的关系。所以实际上因特网上是通过 IP 地址加上端口号来区分不同的服务的。

端口是通过端口号来标记的，端口号只有整数，范围是从0 到65535。



### 3 — 主机资源的具体地址

```java
例如：  /webProject/index.html 
```

一般的URL为：

```java
URL:  http://127.0.0.1:8080/webProject/index.html 
```

# URI

URI：（全称：Uniform Resource Identifier） 统一资源标识符，它是一个字符串**用来标示抽象或物理资源**

Web上可用的每种资源（ HTML文档、图像、音频、视频片段、程序等）都由一个通用资源标识符（Uniform Resource Identifier, 简称”URI”）进行定位。

URI的格式也由三部分组成:

1. 访问资源的命名机制。
2. 存放资源的主机名。
3. 资源自身的名称，由路径表示。



# 联系与区别

URI ：Uniform Resource Identifier，统一资源标识符；
URL：Uniform Resource Locator，统一资源定位符；
URN：Uniform Resource Name，统一资源名称。

![1](/img/1.png)

URI 属于 URL 更高层次的抽象，一种字符串文本标准。

就是说，URI 属于父类，而 URL 属于 URI 的子类。URL 是 URI 的一个子集 

URI 表示请求服务器的路径，定义这么一个资源。而 URL 同时说明要如何访问这个资源（[http://）。](http://)./)

URI可以分为URL,URN，或同时具备locators 和names特性的一个东西。URN作用就好像一个人的名字，URL就像一个人的地址。换句话说：URN确定了东西的身份，URL提供了找到它的方式。”



# 总结

URL是一种具体的URI，它不仅唯一标识资源，而且还提供了定位该资源的信息。

URI是一种语义上的抽象概念，可以是绝对的，也可以是相对的，而URL则必须提供足够的信息来定位。

URI：统一资源标识
URL：统一资源定位
URN：统一资源名称

例如：
www.baidu.com是URL.
www.baidu.com/index.html 是URL 同时也是URI。
所以，URL 就是 URI 的 定位。

但 URI 不一定是 URL。
因为 URI有一类子集是 URN，它是命名资源 但不指定如何定位资源。
如： mailto 需要 加上 相应的结构参数，才能进行 统一资源定位。
如： mailto: xxxxx@qq.com

因此，三者之间的关系是：
URI 一定是 URL
URN + URL 就是 URI。