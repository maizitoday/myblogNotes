---
title:       "netdata监控工具"
subtitle:    ""
description: ""
date:        2019-10-31
author:      "麦子"
image:       "https://c.pxhere.com/photos/50/b0/tree_road_archway_symmetry_and_black_white-99031.jpg!d"
tags:        ["开发工具", "netdata监控工具"]
categories:  ["Tech" ]
---

[TOC]

# 安装

```shell
 yum install zlib-devel gcc make autoconf autogen automake pkgconfig libuuid-devel
 https://github.com/netdata/netdata/releases 找到最新的版本
 cd /home
 wget  https://github.com/netdata/netdata/releases/download/v1.18.1/netdata-vv1.18.1.tar.gz
 tar -xvf netdata-vv1.18.1.tar.gz
 cd  netdata-vv1.18.1.tar.gz/
 sh netdata-installer.sh #这个步骤要久一点
 
```

# 启动

```shell
systemctl start netdata
systemctl status netdata
systemctl stop netdata
systemctl restart netdata
```

# 效果

![Xnip2019-10-31_17-48-48](/img/Xnip2019-10-31_17-48-48.png)

# 注意

**如果有防火墙，记得需要开放端口。** 