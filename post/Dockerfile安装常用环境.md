---
title:       "Dockerfile安装常用环境"
subtitle:    ""
description: ""
date:        2019-08-21
author:      "麦子"
image:       "https://c.pxhere.com/photos/50/b0/tree_road_archway_symmetry_and_black_white-99031.jpg!d"
tags:        ["开发工具", "Dockerfile安装常用环境"]
categories:  ["Tech" ]
---

[TOC]

# mysql

```dockerfile
 FROM mysql
 EXPOSE 3306
 
 # 编译 
 # docker build -t mysql-master .

 # 启动
 # docker run -p 3306:3306  --name mysql-master -e MYSQL_ROOT_PASSWORD=root -itd mysql-master
 
```