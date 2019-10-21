---
title:       "shiro-权限"
subtitle:    ""
description: ""
date:        2019-10-10
author:      "麦子"
image:       "https://c.pxhere.com/images/e4/46/bd237f6583029424694a0d16589b-1435053.jpg!d"
tags:        ["shiro","用户角色权限设计基本设计"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：https://www.cnblogs.com/tianlifitting/p/8005327.html**

**下面截图来源尚学堂视频：https://www.bilibili.com/video/av29862843/?p=27**

# 常用用户角色权限设计

![Xnip2019-09-30_01-34-09](/img/Xnip2019-09-30_01-34-09.png)

这个权限表中添加权限分栏表的id进行关联，这就控制到了基本权限。如上图也可以看成， 管理员和权限分栏表进行关联， 然后权限分栏和权限表关联， 如果需要控制到按钮级别如下：

## 控制到按钮

这里假定把菜单、按钮都看成是一种资源，一个菜单上面有多个按钮。

菜单表（权限分栏表）： menu(menu_id, nemu_name, menu_url)

按钮表： operation(btn_id, btn_code, btn_name, btn_title, menu_id)  btn_title 为提示

按钮表，其中menu_id 区分这个按钮是属于那一个页面，btn_code 存这个按钮在页面上的组件ID，这个在一个页面下应该是唯一的，方便后续页面定位她。比如我的HTML页面A，有一个按钮 <input type='button' id='saveBtn' ......这里 btn_code 就存 saveBtn。

# shiro简介

## 功能简介

![Xnip2019-09-30_16-40-35](/img/Xnip2019-09-30_16-40-35.png)

![Xnip2019-09-30_16-35-01](/img/Xnip2019-09-30_16-35-01.png)



## shiro外部架构

![Xnip2019-09-30_16-43-06](/img/Xnip2019-09-30_16-43-06.png)

**是一个门面模式**

![Xnip2019-09-30_16-44-10](/img/Xnip2019-09-30_16-44-10.png)

## shiro内部架构

![Xnip2019-09-30_16-46-56](/img/Xnip2019-09-30_16-46-56.png)

![Xnip2019-09-30_16-48-58](/img/Xnip2019-09-30_16-48-58.png)

##  shiro中默认的过滤器

![Xnip2019-10-10_14-18-33](/img/Xnip2019-10-10_14-18-33.png)

### URL 匹配模式

![Xnip2019-10-10_14-22-05](/img/Xnip2019-10-10_14-22-05.png)

### URL 匹配顺序

![Xnip2019-10-10_14-22-34](/img/Xnip2019-10-10_14-22-34.png)



## 验证策略

![Xnip2019-10-10_14-52-53](/img/Xnip2019-10-10_14-52-53.png)

## 授权

![Xnip2019-10-10_14-54-35](/img/Xnip2019-10-10_14-54-35.png)

### 授权方式

![Xnip2019-10-10_14-55-03](/img/Xnip2019-10-10_14-55-03.png)

### Permissions

![Xnip2019-10-10_14-59-37](/img/Xnip2019-10-10_14-59-37.png)

### Shiro 标签

![Xnip2019-10-10_15-09-51](/img/Xnip2019-10-10_15-09-51.png)

### 权限注解

![Xnip2019-10-10_15-11-12](/img/Xnip2019-10-10_15-11-12.png)

# 会话管理

![Xnip2019-10-10_15-13-12](/img/Xnip2019-10-10_15-13-12.png)

# RememberMe

![Xnip2019-10-10_15-52-57](/img/Xnip2019-10-10_15-52-57.png)

