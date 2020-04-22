---
title:       "远程开发和调试，SFTP,remote-ssh"
subtitle:    ""
description: "remote-ssh，远程连接Linux服务器，远程直接上传jar或者class文件,SFTP"
date:        2019-08-10
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["开发工具", "远程开发"]
categories:  ["Tech" ]
---

[TOC]

# remote-ssh

远程连接远端Linux， 然后可以用vscode的工具进行打开和编辑远端Linux的文件。

![Xnip2019-08-10_21-04-31](/img/Xnip2019-08-10_21-04-31.png)

安装好插件后， 直接连接和配置需要远程的Linux服务器， 前提要确保你的机子已经可以和远端Linux，秘钥登录了。找到你自己机子的配置SSH的地方， 进行下面配置：

```shell
Host mycentos7
    HostName 10.211.55.6
    User parallels
    port 22
    IdentityFile ~/.ssh/id_rsa
```

然后点击连接，可以出现下面图：

![Xnip2019-08-10_21-07-31](/img/Xnip2019-08-10_21-07-31.png)

![Xnip2019-08-10_21-08-06](/img/Xnip2019-08-10_21-08-06.png)

可以看到 ，这个时候， 远程的服务器就像在本地一样了。 



# SFTP

通过下面的命令，配置和远端Linux服务器通信，达到可以对文件进行上传和下载，对比，有时候我们在修改代码的时候，方便替换一两个文件。

```
>SFTP:config
```



sftp.jso 格式如下：更多配置看官网地址：<https://marketplace.visualstudio.com/items?itemName=liximomo.sftp>

```
{
    "name": "sftp测试",
    "host": "10.211.55.6",
    "protocol": "sftp",
    "port": 22,
    "username": "parallels",
    "remotePath": "/home/parallels/Music",
    "uploadOnSave": true
}
```



# 远程调试代码

## launch配置

```json
{
    "configurations": [
        {
            "type": "java",
            "name": "Debug (Attach)",
            "request": "attach",
            "hostName": "10.211.55.10",
            "port": 5006
        }
    ]
}
```

## springboot启动

```shell

java  -Xnoagent -Djava.compiler=NONE -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=10.211.55.10:5006 -jar remotedemo-0.0.1-SNAPSHOT.jar 
```

注意：这里的address一定需要是ip+端口形式启动，否则无法开启远程调试。

## 命令分析

**转载地址：<https://blog.51cto.com/a4boy/1889940>**

### transport

这里通常使用套接字传输。但是在 Windows 平台上也可以使用共享内存传输。

### server

如果值为 y，目标应用程序监听将要连接的调试器应用程序。否则，它将连接到特定地址上的调试器应用程序。

### address

这是连接的传输地址。如果服务器为 n，将尝试连接到该地址上的调试器应用程序。否则，将在这个端口监听连接。

### suspend

 如果值为 y，目标 VM 将暂停，直到调试器应用程序进行连接。

