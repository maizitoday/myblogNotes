---
title:       "常用终端命令"
subtitle:    ""
description: ""
date:        2019-06-27
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["mac", "终端命令"]
categories:  ["Tech" ]
---

[TOC]

# 查看端口

```shell
 lsof -i tcp:port 
 （port替换成端口号，比如6379）可以查看该端口被什么程序占用，并显示PID，方便KILL（kill pid）
```

