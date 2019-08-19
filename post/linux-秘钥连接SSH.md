---
title:       "秘钥连接SSH"
subtitle:    ""
description: ""
date:        2019-08-10
author:      "麦子"
image:       "https://get.pxhere.com/photo/snow-macro-snowflake-black-darkness-phenomenon-computer-wallpaper-screenshot-black-and-white-font-grass-night-midnight-1420032.jpg"
tags:        ["服务脚本", "开发工具", "秘钥连接SSH"]
categories:  ["Tech" ]
---



[TOC]



# 查看是否SSH运行

通过下面几种命令，查看SSH是否运行,  查看进程， 端口， 

```shell
systemctl status sshd

ps -elf | grep ssh

netstat -anpt | grep 22

```

还可以直接直接下面的查看是否对外有22端口。

```
nmap 127.0.0.1

Starting Nmap 6.40 ( http://nmap.org ) at 2019-08-19 21:08 CST
Nmap scan report for localhost (127.0.0.1)
Host is up (0.0000020s latency).
Not shown: 995 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
25/tcp   open  smtp
111/tcp  open  rpcbind
631/tcp  open  ipp
8080/tcp open  http-proxy


```



# 制作公钥和私钥

```shell
ssh-keygen` 即可，一路回车默认即可。
默认生成的公钥文件：`~/.ssh/id_rsa.pub`
私钥文件：`~/.ssh/id_rsa
```

# 将公钥注册到服务端

```
ssh-copy-id -i ~/.ssh/id_rsa.pub parallels@10.211.55.6
```

执行成功后会在服务端生成`~/.ssh/authorized_keys`文件，文件内容和客户端生成的`id_rsa.pub`内容完全一致。

# mac配置

在你当前用户， 也就是/Users/maizi/.ssh这个文件夹下面， 配置里面的一个config文件，具体如下：

```shell
Host mycentos7
    HostName 10.211.55.6
    User parallels
    port 22
    IdentityFile ~/.ssh/id_rsa
```

然后终端之间使用：

```shell
ssh  mycentos7 
```

就可以直接连接远程Linux了