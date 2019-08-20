---
title:       "sourcetrail源码查看工具"
subtitle:    ""
description: ""
date:        2019-08-21
author:      "麦子"
image:       "https://c.pxhere.com/photos/8a/20/scuba_diver_diver_diving_underwater_water_sea_ocean_deep-1087258.jpg!d"
tags:        ["开发工具", "源码查看工具", "连接vscode查看源码"]
categories:  ["Tech" ]
---

[TOC]



# sourcetrail插件配置

## vscode配置

```properties
 "sourcetrail.startServerAtStartup": true,  // 设置为true在VS Code启动时启动TCP侦听器
 "sourcetrail.ip": "127.0.0.1", // TCP服务器IP地址
 "sourcetrail.pluginPort": 8666, // 端口VS代码监听
 "sourcetrail.sourcetrailPort": 8667, // 端口Sourcetrail监听
```

## 客户端软件配置

![Xnip2019-08-20_23-24-09](/img/Xnip2019-08-20_23-24-09.png)

## 执行步骤

安装好后，执行下面命令。

```properties
Sourcetrail: (Re)start server
```

看到下面标志就表示成功了。

![Xnip2019-08-20_23-22-44](/img/Xnip2019-08-20_23-22-44.png)

就可以和sourcetrail客户端，进行代码的切换了。

**注意：如果没有成功，就重启一下vscode和客户端。我是重启后，他才自动连接成功的。**



# 插件基本使用

![Xnip2019-08-20_23-34-31](/img/Xnip2019-08-20_23-34-31.png)

**黄色块** ：表示的是这个类中，调用了哪些方法和创建了哪些对象。

**小房子图标** ：表示的是这个对象的属性。

**地球图标**：表示的这个类里面的方法分别是什么。

**黄色线条**：表示的从哪一方法调用了这些方法。比如看实际代码，main方法里面就调用了黄色块状的几个方法。 



**如图在进行点击 getMapper方法**

![Xnip2019-08-20_23-48-12](/img/Xnip2019-08-20_23-48-12.png)

可以看到， **getMapper**这个方法是从 **1号**过来， 然后这个**getMapper**方法里面又调用了**3号**方法。同时可以看到**4号**的这一条线，证明这个属性是作为了参数传入到了个这个**getMapper**方法中的。



方法是从哪里调用哪里，**注意看箭头的方法**。**同时点击最上面的白色字体(如：MySqlSession)**就可以打开这个类。

# 保存书签

如上图，可以看到一个**五角星**的图标，这个图标是可以把你当前看的这个类给直接mark一下， 下次你如果想看的话，就可以直接打开查看了。 

# 注意

主要注意看箭头的方法，同时需要知道， 当你打开这一个类的时候， 他会把这个类里面的所有的对象和属性都会显示在两边。