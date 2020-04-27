---
title:       "centos7卸载和安装jdk"
subtitle:    ""
description: ""
date:        2019-08-11
author:      "麦子"
image:       "https://get.pxhere.com/photo/sea-light-black-and-white-photography-underwater-reflection-darkness-black-monochrome-outer-space-reef-diver-scuba-screenshot-macro-photography-monochrome-photography-atmosphere-of-earth-computer-wallpaper-99827.jpg"
tags:        ["开发工具","shell","jdk安装和卸载"]
categories:  ["Tech" ]
---

[TOC]

**转载地址**：<https://blog.csdn.net/zitong_ccnu/article/details/40041533>**

# 查看JDK安装版本

```shell
[root@localhost ~]#java -version
java version *1.7.0_51*
OpenJDK Runtime Environment ( rhel-2.4.5.5.el7-x86_64 u51-b31)
OpenJDK 64-Bit Server VM (build 24.51-b03, mixed mode)
```

# 查找OpenJDK安装包

```shell
rpm -qa | grep openjdk
java-1.7.0-openjdk-headless-1.7.0.51-2.4.5.5.el7.x86_64
java-1.7.0-openjdk-1.7.0.51-2.4.5.5.el7.x86_64
```

# 卸载OpenJDK安装包

```shell
[root@localhost ~]#yum -y remove java-1.7.0-openjdk-headless.x86_64
[root@localhost ~]#yum -y remove java-1.7.0-openjdk.x86_64
```

# 安装jdk环境

```shell
tar -xvf jdk-8u11-linux-x64.tar.gz
echo 'export JAVA_HOME=/usr/local/java/jdk1.8.0_11'>>/etc/profile
echo 'export CLASSPATH=$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar'>>/etc/profile
echo 'export PATH=$PATH:$JAVA_HOME/bin'>>/etc/profile
source /etc/profile
java -version
```