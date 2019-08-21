---
title:       "Dockerfile安装常用环境"
subtitle:    ""
description: ""
date:        2019-08-21
author:      "麦子"
image:       "https://c.pxhere.com/photos/1e/b3/workers_construction_site_hardhats_silhouettes_building_tools_job-749554.jpg!d"
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