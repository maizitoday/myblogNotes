---
title:       "JNDI理解"
subtitle:    ""
description: ""
date:        2020-03-02
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["java相关概念", "JNDI"]
categories:  ["Tech" ]
---

# 简介

JNDI就是以中间件， 比如我们常用的JDBC的数据库配置就是通过配置文件来注入到程序中的， 定义了一套约定的接口，         然后我们去实现这个接口，通过查找这个然后直接获取数据源了， 而不是在我们的代码中写入用户名和密码， 这样的话进         行解耦， 我们常用的 J2EE程序默认必须实现这个， 分成和解耦进行约定的接口来执行。  我们在获取第三方资源的时             候， 约定的一套标准的感觉。JNDI 技术产生后，就可方便的查找远程或是本地对象。

# 概念

链接：https://www.jianshu.com/p/ad4c16e0b8ba

JNDI（Java Naming and Directory Interface ），类似于在一个中心注册一个东西，以后要用的时候，只需要根据名字去注册中心查找，注册中心返回你要的东西。web程序，我们可以将一些东西（比如数据库相关的）交给服务器软件去配置和管理（有全局配置和单个web程序的配置），在程序代码中只要通过名称查找就能得到我们注册的东西，而且如果注册的东西有变，比如更换了数据库，我们只需要修改注册信息，名称不改，因此代码也不需要修改。

 