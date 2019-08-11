---
title:       "远程开发和调试"
subtitle:    ""
description: ""
date:        2019-08-10
author:      "麦子"
image:       "https://get.pxhere.com/photo/fishing-fisherman-lake-hobby-nature-sports-man-sunset-action-active-activity-casting-catch-catching-evening-freshwater-landscape-leisure-lure-male-dusk-silhouette-water-sky-sunrise-reflection-horizon-calm-morning-sea-dawn-atmosphere-afterglow-cloud-sun-night-darkness-1437559.jpg"
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

