---
title:       "web安全-CSRF"
subtitle:    ""
description: ""
date:        2019-06-01
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["web安全", "CSRF"]
categories:  ["Tech" ]
---

[TOC]

# csrf 是什么

**转载地址：https://blog.csdn.net/sinat_41898105/article/details/80783551**

CSRF即跨站请求攻击。简单的说，是攻击者通过一些技术手段欺骗用户的浏览器去访问一个自己以前认证过的站点并运行一些操作（如发邮件，发消息，甚至财产操作（如转账和购买商品））。因为浏览器之前认证过，所以被访问的站点会绝点是这是真正的用户操作而去运行。  这就利用了web中用户身份认证验证的一个漏洞：简单的身份验证仅仅能保证请求发自某个用户的浏览器，却不能保证请求本身是用户自愿发出的。

![2018062314565055](/img/2018062314565055.png)