---
title:       "centos安装docker环境"
subtitle:    ""
description: "安装k8s环境"
date:        2019-05-08
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["docker", "centos关闭防火墙"]
categories:  ["Tech" ]

---

[TOC]

**转载地址：https://blog.csdn.net/weixin_40654252/article/details/86472179**

# 关闭CentOS自带防火墙

```shell
systemctl disable firewalld
systemctl stop firewalld
```

# 安装 etcd 和 kubernetes 软件

**会自动安装docker软件**

```shell
yum -y install etcd kubernetes

#修改下面配置文件
vim /etc/sysconfig/docker
OPTIONS='--selinux-enabled=false --insecure-registry gcr.io'

#修改下面配置文件
vim /etc/kubernetes/apiserver
KUBE_ADMISSION_CONTROL="--admission-control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ResourceQuota"
```

# 启动所有服务

```shell
systemctl start etcd docker kube-apiserver kube-controller-manager kube-scheduler kubelet kube-proxy
```

用命令查看k8s是否已经运行

```shell
[root@localhost parallels]# kubectl cluster-info
Kubernetes master is running at http://localhost:8080
```

至此一个单机版kubernetes集群就安装启动完成了

# docker，k8s服务启动命令

```shell
#关闭 
systemctl stop docker
#启动  
systemctl start docker  
#开机启动
systemctl enable docker

#查看运行状态
systemctl status kubelet
systemctl status docker
```



# 错误  

```java
Job for kube-apiserver.service failed because the control process exited with error code. See "systemctl status kube-apiserver.service" and "journalctl -xe" for details.
```

我这里的原因是：端口被占用，修改端口号

```shell
vi /etc/kubernetes/apiserver 
KUBE_API_PORT="--port=18080"
```

