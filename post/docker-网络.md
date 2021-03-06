---
title:       "doceker系列-网络"
subtitle:    ""
description: ""
date:        2019-05-20
author:      "麦子"
image:       "https://c.pxhere.com/images/38/35/ee28969602557aff0677ded7da20-1590317.jpg!d"
tags:        ["docker", "网络"]
categories:  ["Tech" ]
---

[TOC]

**转载地址： https://www.cnblogs.com/yy-cxd/p/6553624.html**

# 四种网络模式

## bridge模式

当Docker进程启动时，会在主机上创建一个名为docker0的虚拟网桥，此主机上启动的Docker容器会连接到这个虚拟网桥上。虚拟网桥的工作方式和物理交换机类似，这样主机上的所有容器就通过交换机连在了一个二层网络中。

从docker0子网中分配一个IP给容器使用，并设置docker0的IP地址为容器的默认网关。在主机上创建一对虚拟网卡veth pair设备，Docker将veth pair设备的一端放在新创建的容器中，并命名为eth0（容器的网卡），另一端放在主机中，以vethxxx这样类似的名字命名，并将这个网络设备加入到docker0网桥中。可以通过brctl show命令查看。

bridge模式是docker的默认网络模式，不写--net参数，就是bridge模式。使用docker run -p时，docker实际是在iptables做了DNAT规则，实现端口转发功能。可以使用iptables -t nat -vnL查看。

bridge模式如下图所示：

![5-20](/img/5-20.jpeg)



## none模式

使用none模式，Docker容器拥有自己的Network Namespace，但是，并不为Docker容器进行任何网络配置。也就是说，这个Docker容器没有网卡、IP、路由等信息。需要我们自己为Docker容器添加网卡、配置IP等。

Node模式示意图:

![5-21](/img/5-21.jpeg)

## host模式

如果启动容器的时候使用host模式，那么这个容器将不会获得一个独立的Network Namespace，而是和宿主机共用一个Network Namespace。容器将不会虚拟出自己的网卡，配置自己的IP等，而是使用宿主机的IP和端口。但是，容器的其他方面，如文件系统、进程列表等还是和宿主机隔离的。

Host模式如下图所示：

![5-22](/img/5-22.jpeg)

## container模式，

这个模式指定新创建的容器和已经存在的一个容器共享一个 Network Namespace，而不是和宿主机共享。新创建的容器不会创建自己的网卡，配置自己的 IP，而是和一个指定的容器共享 IP、端口范围等。同样，两个容器除了网络方面，其他的如文件系统、进程列表等还是隔离的。两个容器的进程可以通过 lo 网卡设备通信。

Container模式示意图：

![5-23](/img/5-23.jpeg)

 

## $ docker network create somenetwork 

https://blog.csdn.net/gezhonglei2007/article/details/51627821

**默认网络与自定义bridge网络的差异**

默认网络docker0：网络中所有主机间只能用IP相互访问。通过--link选项创建的容器可以对链接的容器名(container-name)作为hostname进行直接访问。
自定义网络(bridge)：网络中所有主机除ip访问外，还可以直接用**容器名**(container-name)作为hostname相互访问。