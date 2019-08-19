---
title:       "linux下利用nohup后台运行jar文件包程序"
subtitle:    ""
description: ""
date:        2019-08-19
author:      "麦子"
image:       "https://get.pxhere.com/photo/Dodge-Daytona-low-key-toy-car-white-black-motor-vehicle-black-and-white-automotive-design-mode-of-transport-darkness-sports-car-monochrome-photography-photography-light-vehicle-performance-car-automotive-lighting-automotive-exterior-computer-wallpaper-monochrome-night-sky-bumper-city-car-wheel-driving-midnight-compact-car-executive-car-full-size-car-headlamp-1423465.jpg"
tags:        ["服务脚本", "nohup后台运行jar文件", "java基础"]
categories:  ["Tech" ]
---

[TOC]

转载地址：<https://blog.csdn.net/qian9140/article/details/84629762>**

# java -jar XXX.jar

特点：当前ssh窗口被锁定，可按CTRL + C打断程序运行，或直接关闭窗口，程序退出。 **但是发现centos7的时候， 关掉窗口， 程序依旧在运行。** 

# java -jar XXX.jar &

&代表在后台运行。

特定：当前ssh窗口不被锁定，但是当窗口关闭时，程序中止运行。

# nohup java -jar XXX.jar &

nohup 意思是不挂断运行命令,当账户退出或终端关闭时,程序仍然运行

当用 nohup 命令执行作业时，缺省情况下该作业的所有输出被重定向到nohup.out的文件中，除非另外指定了输出文件。

# nohup java -jar XXX.jar >temp.txt &

解释下>temp.txt

command >out.file

command >out.file是将command的输出重定向到out.file文件，即输出内容不打印到屏幕上，而是输出到out.file文件中。

# jobs

那么就会列出所有后台执行的作业，并且每个作业前面都有个编号。
如果想将某个作业调回前台控制，只需要 fg + 编号即可。