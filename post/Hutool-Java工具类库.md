---
title:       "Hutool-Java工具类库"
subtitle:    ""
description: "Hutool主要主件，ASC、MD5、SHA1，.jar和sources.jar及javadoc.jar三者的关系"
date:        2020-02-27
author:      "麦子"
image:       "https://c.pxhere.com/photos/50/b0/tree_road_archway_symmetry_and_black_white-99031.jpg!d"
tags:        ["开发工具", "Hutool工具类学习"]
categories:  ["Tech" ]
---

[TOC]

**官方地址：https://hutool.cn**

# [包含组件]

一个Java基础工具类，对文件、流、加密解密、转码、正则、线程、XML等JDK方法进行封装，组成各种Util工具类，同时提供以下组件：

| 模块               | 介绍                                                         |
| ------------------ | ------------------------------------------------------------ |
| hutool-aop         | JDK动态代理封装，提供非IOC下的切面支持                       |
| hutool-bloomFilter | 布隆过滤，提供一些Hash算法的布隆过滤                         |
| hutool-cache       | 简单缓存实现                                                 |
| hutool-core        | 核心，包括Bean操作、日期、各种Util等                         |
| hutool-cron        | 定时任务模块，提供类Crontab表达式的定时任务                  |
| hutool-crypto      | 加密解密模块，提供对称、非对称和摘要算法封装                 |
| hutool-db          | JDBC封装后的数据操作，基于ActiveRecord思想                   |
| hutool-dfa         | 基于DFA模型的多关键字查找                                    |
| hutool-extra       | 扩展模块，对第三方封装（模板引擎、邮件、Servlet、二维码、Emoji、FTP、分词等） |
| hutool-http        | 基于HttpUrlConnection的Http客户端封装                        |
| hutool-log         | 自动识别日志实现的日志门面                                   |
| hutool-script      | 脚本执行封装，例如Javascript                                 |
| hutool-setting     | 功能更强大的Setting配置文件和Properties封装                  |
| hutool-system      | 系统参数调用封装（JVM信息等）                                |
| hutool-json        | JSON实现                                                     |
| hutool-captcha     | 图片验证码实现                                               |
| hutool-poi         | 针对POI中Excel和Word的封装                                   |
| hutool-socket      | 基于Java的NIO和AIO的Socket封装                               |

**可以根据需求对每个模块单独引入，也可以通过引入`hutool-all`方式引入所有模块。**

# ASC、MD5、SHA1 是干什么的呢？ 

为了确保你得到的文件是正确的版本，而没有被注入病毒和木马程序。例如我们经常在网上下载软件，而这些软件已经被注入了一些广告和病毒等，如果不进行文件与原始发布商的一致性校验的话，可能会给我们带来一定的损失。

二 文件一致性校验原理
要进行文件的一致性校验，我们不可能像文本文件比较那样，将两个文件放到一起对比，因为很多的时候文件很大。目前最理想的办法就是，是通过加密算法，对文件生成对应的值，通过生成的值与发布商提供的值比较来确认两个文件是否一致。

ASC、MD5、SHA1就是目前使用的几种加密算法。

Linux下的学习开始总是艰难的，但有的时候，却发现Linux下远比Windows的操作来的实在的多——这下载文件的完整性就是其中一件，让本人觉着很爽的一件事情。在编译安装各种软件的时候，总要到各个网站上收集下软件源码包。正由于此，软件的入口就非常复杂，校验下载的文件是否被修改过就显得非常有必要了。而校验方法当前一般是MD5，SHA1，PGP三种。在Windows那个漫长的岁月里（沧桑有木有），一般只能接触到前两种——**前提是你会去校验的话**。

# .jar和sources.jar及javadoc.jar三者的关系

.jar是我们需要用的jar包。
sources.jar是jar包的源码。
javadoc.jar解压之后会有很多的.html文件，这些文件是在线的api帮助文档。导入后，鼠标放上去就回有注释出来。 

# 问题

vscode目前还不知道怎么导入Javadoc.jar的包。



