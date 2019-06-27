---
title:       "SSO单点登录"
subtitle:    ""
description: ""
date:        2019-09-27
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["web相关概念", "SSO", "单点登录"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：<https://blog.csdn.net/zhangjingao/article/details/81735041>**

# 什么是SSO

SSO（Single Sign On）单点登录是实现多个系统之间统一登录的验证系统，简单来说就是：有A，B，C三个系统，在A处登录过后，再访问B系统，B系统就已经处于了登录状态，C系统也是一样。举个生活中栗子：你同时打开天猫和淘宝，都进入login界面，都要求你登录的，现在你在淘宝处登录后，直接在天猫处刷新，你会发现，你已经登录了，而且就是你在淘宝上登录的用户。说明他们实现了SSO，并且持有相同的信息。

当然这个特性意味着它的使用场景是：同一公司下的不同子系统，因为对于SSO来说，每一个子系统拥有的信息都一样，是用户的全部信息，如果是不同公司，那这肯定不合适。现在的天猫和淘宝就是这样的一套SSO。

SSO简单来说就是一句话：**一处登录，全部访问。** 

# JWT实现单点登录