---
title:       "java基础-session,cookie"
subtitle:    ""
description: ""
date:        2019-05-20
author:      "麦子"
image:       "https://c.pxhere.com/photos/e7/82/road_forest_season_autumn_fall_landscape_nature_forest_landscape-839463.jpg!d"
tags:        ["java基础", "web基础", "session", "cookie", "分布式系列", "分布式session"]
categories:  ["Tech" ]
---

[TOC]

**说明转载地址：http://www.justdojava.com/2019/05/12/cookie-session/#**

# 什么是 Cookie

HTTP Cookie（也叫 Web Cookie或浏览器 Cookie）是服务器发送到用户浏览器并保存在本地的一小块数据，它会在浏览器下次向同一服务器再发起请求时被携带并发送到服务器上。通常，它用于告知服务端两个请求是否来自同一浏览器，如保持用户的登录状态。Cookie 使基于无状态的 HTTP 协议记录稳定的状态信息成为了可能。一般都是保存在一个域名下面。

# 什么是 Session

Session 代表着服务器和客户端一次会话的过程。Session 对象存储特定用户会话所需的属性及配置信息。这样，当用户在应用程序的 Web 页之间跳转时，存储在 Session 对象中的变量将不会丢失，而是在整个用户会话中一直存在下去。当客户端关闭会话，或者 Session 超时失效时会话结束。



## session在服务器存活时间 

### web容器中设置（此处以tomcat为例）

```xml
<session-config>  
        <session-timeout>30</session-timeout>  
</session-config>  
```



### 工程的web.xml中设置

```xml
<session-config>
      <session-timeout>15</session-timeout>
</session-config>
```



### 通过Java代码设置

```java
session.setMaxInactiveInterval（30*60）;//以秒为单位
```



## sessionid生成

session的id是从哪里来的，sessionID是如何使用的：当客户端第一次请求session对象时候，服务器会为客户端创建一个session，并将通过特殊算法算出一个session的ID，用来标识该session对象。

**创建**：sessionid第一次产生是在直到某server端程序调用 HttpServletRequest.getSession(true)这样的语句时才被创建。

**获取**：HttpSession session = MySessionContext.getSession(sessionId);

在客户端的cookie中，保存sessionid的名称是JSESSIONID，sessioid的值是一串复杂字符串组成的。这个不一定，网站可以随时修改这个key。



## session什么时候消失

**原文：https://blog.csdn.net/stanxl/article/details/47105051** 

一个是Session.invalidate()方法，不过这个方法在实际的开发中，并不推荐，可能在强制注销用户的时候会使用；

一个是当前用户和服务器的交互时间超过默认时间后，Session会失效

**为什么当我们关闭浏览器后，就再也访问不到之前的session了呢？**

其实之前的Session一直都在服务器端，而当我们关闭浏览器时，此时的Cookie是存在，于浏览器的进程中的，当浏览器关闭时，Cookie也就不存在了。

其实Cookie有两种:

**一种是存在于浏览器的进程中;**

**一种是存在于硬盘上**

而session的Cookie是存在于浏览器的进程中，那么这种Cookie我们称为会话Cookie，当我们重新打开浏览器窗口时，之前的Cookie中存放的Sessionid已经不存在了，此时服务器从HttpServletRequest对象中没有检查到sessionid，服务器会再发送一个新的存有Sessionid的Cookie到客户端的浏览器中，此时对应的是一个新的会话，而服务器上原先的session等到它的默认时间到之后，便会自动销毁。

# 为什么需要 Cookie 和 Session，他们有什么关联？



说起来为什么需要 Cookie ，这就需要从浏览器开始说起，我们都知道浏览器是没有状态的(HTTP 协议无状态)，这意味着浏览器并不知道是张三还是李四在和服务端打交道。这个时候就需要有一个机制来告诉服务端，本次操作用户是否登录，是哪个用户在执行的操作，那这套机制的实现就需要 Cookie 和 Session 的配合。

那么 Cookie 和 Session 是如何配合的呢？我画了一张图大家可以先了解下。

![5-21-1](/img/5-21-1.jpg)

根据以上流程可知，**SessionID 是连接 Cookie 和 Session 的一道桥梁**，大部分系统也是根据此原理来验证用户登录状态。

# 禁止了 Cookie，如何保障整个机制的正常运转

当用户第一次登录后，服务器根据提交的用户信息生成一个 Token，响应时将 Token 返回给客户端，以后客户端只需带上这个 Token 前来请求数据即可，无需再次登录验证。

# 分布式 Session 解决方案

在互联网公司为了可以支撑更大的流量，后端往往需要多台服务器共同来支撑前端用户请求，那如果用户在 A 服务器登录了，第二次请求跑到服务 B 就会出现登录失效问题。

分布式 Session 一般会有以下几种解决方案：



## Nginx ip_hash 策略

Nginx ip_hash 策略，服务端使用 Nginx 代理，每个请求按访问 IP 的 hash 分配，这样来自同一 IP 固定访问一个后台服务器，避免了在服务器 A 创建 Session，第二次分发到服务器 B 的现象。

**缺点**： 如果其中一台服务器挂了， 那么在这个服务器上面的用户就无法使用了。



## Session 复制

Session 复制，任何一个服务器上的 Session 发生改变（增删改），该节点会把这个 Session 的所有内容序列化，然后广播给所有其它节点。

**简单操作**：进行tomcat的集群配置，作为一个组。

**优点**：侵入性小， 适应各种nginx的负载各种策略。

**缺点：** 复制时候，时间有延迟，  其次占用我们内存。 占宽带。



## 共享 Session(一般用这种来管理)

共享 Session，服务端无状态话，将用户的 Session 等信息使用缓存中间件来统一管理，保障分发到每一个服务器的响应结果都一致。也就是redis来处理。

# 前端跨域

**原文地址：https://www.cnblogs.com/renjing/p/6394725.html**

跨域是指a页面想获取b**页面**资源，如果a、b页面的协议、域名、端口、子域名不同，所进行的访问行动都是跨域的，而浏览器为了安全问题一般都限制了跨域访问，也就是不允许跨域请求资源。注意：跨域限制访问，其实是浏览器的限制。理解这一点很重要！！！

## 跨域访问示例

假设有两个网站，A网站部署在：http://localhost:81 即本地ip端口81上；B网站部署在：http://localhost:82 即本地ip端口82上。现在A网站的页面想去访问B网站的信息，这种就是跨域， 直接处理会报错。

## nginx反向代理解决跨域问题

```nginx
server {
      listen       80; #监听80端口，可以改成其他端口
      server_name  localhost; # 当前服务的域名

      #charset koi8-r;
      #access_log  logs/host.access.log  main;

      location / {
        proxy_pass http://localhost:81;
        proxy_redirect default;
      }

      location /apis { #添加访问目录为/apis的代理配置
        rewrite  ^/apis/(.*)$ /$1 break;
        proxy_pass   http://localhost:82;
      }
  }
```



