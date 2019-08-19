---
title:       "CentOS7-防火墙"
subtitle:    ""
description: ""
date:        2019-08-19
author:      "麦子"
image:       "https://c.pxhere.com/images/bf/95/76b7d5d335a773e933f8085ced47-1419036.jpg!d"
tags:        ["服务脚本", "防火墙相关知识"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：<https://blog.csdn.net/Joe68227597/article/details/75207859>**

# firewalld基本操作

## 查看状态

```shell
systemctl status firewalld
```



## 安装firewalld

```shell
yum install firewalld
```



## 开启

```shell
systemctl start firewalld.service
```



## *关闭*

```shell
systemctl stop firewalld.service
```



## *设置开机自动启动*

```shell
systemctl enable firewalld.service
```



## *设置关闭开机制动启动*

```shell
systemctl disable firewalld.service
```



## *重新加载防火墙*

```shell

firewall-cmd --reload //在不改变状态的条件下重新加载防火墙

systemctl restart firewalld.service //修改配置后需要重启服务使其生效
```

# 黑白名单

**转载地址：<https://www.linode.com/docs/security/firewalls/introduction-to-firewalld-on-centos/>**

## 开启某个端口

```shell

firewall-cmd --permanent --zone=public --add-port=8080-8081/tcp  //永久

firewall-cmd  --zone=public --add-port=8080-8081/tcp   //临时
```

**命令含义：**
--zone #作用域
--add-port=80/tcp #添加端口，格式为：端口/通讯协议
--permanent #永久生效，没有此参数重启后失效



## 查看想开的端口是否已开

```shell
firewall-cmd --query-port=8888/tcp
```

提示yes表示已开通，提示no表示未开通。



下面这个也可以， 而且显示的信息更多

```
lsof -i:8888
[root@centos-7 parallels]# lsof -i:5006
COMMAND   PID USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
java    16749 root    5u  IPv4 7225939      0t0  TCP centos-7.shared:wsm-server->bogon:63833 (ESTABLISHED)

```



## 若要关闭端口

```shell
firewall-cmd --permanent --remove-port=8888/tcp
```



## 查看已开放的端口

```
firewall-cmd --zone=public --list-ports
```



## 添加白名单

允许来自主机192.168.0.14的所有IPv4流量。

```shell
sudo firewall-cmd --zone=public --add-rich-rule 'rule family="ipv4" source address=192.168.0.14 accept'
```



## 添加黑名单

通过TCP拒绝从主机192.168.1.10到端口22的IPv4流量。

```shell
sudo firewall-cmd --zone=public --add-rich-rule 'rule family="ipv4" source address="192.168.1.10" port port=22 protocol=tcp reject'
```



## 查看防火墙的相关配置

```shell
firewall-cmd --zone=public --list-all
```















