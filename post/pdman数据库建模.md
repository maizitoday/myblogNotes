---
title:       "pdman数据库建模"
subtitle:    ""
description: ""
date:        2019-10-31
author:      "麦子"
image:       "https://c.pxhere.com/photos/50/b0/tree_road_archway_symmetry_and_black_white-99031.jpg!d"
tags:        ["开发工具", "pdman数据库建模"]
categories:  ["Tech" ]
---

[TOC]

**官网地址：https://my.oschina.net/skymozn/blog/1821184**

# 常用步骤

1.  连接数据源
2.  新增模块，在这里创建表，通过拉线处理表之间的关系，然后保存。 
3.  点击模型版本， 进行保存新版本， 然后进行同步。 
4.  同步操作的时候， 数据库就实际上进行了表的创建或者修改字段的操作。 
5.  如果需要导出数据库的结构，直接可以导出为pdf或者其他格式。 
6.  点击关系图，可以查看库里面的所有表的关系。 
7.  点击"开始"里面的设置， 可以修改创建表的模板。 

# 好处

通过他的建模， 我们可以直接创建表结构，同时通过mybatis直接创建为java对象。 同时，我们有记录每次修改和上一个版本的对比，这样更加的清晰， 是一个很多的建模软件。 

