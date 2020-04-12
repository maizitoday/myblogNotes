---
title:       "HomeBrew"
subtitle:    ""
description: ""
date:        2020-03-10
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["开发工具", "HomeBrew"]
categories:  ["Tech" ]
---

[TOC]

转载地址：https://www.jianshu.com/p/ba22aea460c6

# 简介

什么是Homebrew呢？Homebrew is the easiest and most flexible way to install the UNIX tools Apple didn’t include with OS X. 官方的解释非常明了，Homebrew是一个包管理器，用于在Mac上安装一些OS X没有的UNIX工具（比如著名的wget）。
Homebrew将这些工具统统安装到了 /usr/local/Cellar 目录中，并在 /usr/local/bin 中创建符号链接。

# 目录

Homebrew安装成功后，会自动创建目录 /usr/local/Cellar 来存放Homebrew安装的程序。

# 使用

这是你在命令行状态下面就可以使用 brew 命令了.通过 brew install就可以安装软件了,通过 brew search 就可以搜索程序，例如 brew search vim ,就可以搜索名称包括vim的程序,

通过 brew [update](http://www.osxtoy.com/tag/update/) 就可以把包信息更新到最新，不过包更新是通过git命令，所以要先通过 brew install git 命令安装git。

# 命令

## 显示已经安装软件列表

```shell
brew list 
```

## 查看brew的帮助

```shell
brew –help
```

## 安装软件

```shell
brew install git
```

## 卸载软件

```shell
brew uninstall git
```

## 搜索软件

```shell
brew search git
```

## 更新软件

```shell
更新软件，把所有的Formula目录更新，并且会对本机已经安装并有更新的软件用*标明。
brew update
更新某具体软件
brew upgrade git
```

## 查看软件信息

```shell
brew [info | home] [FORMULA...]
```

## 删除程序

```shell
brew cleanup git   #单个软件删除
brew cleanup       #所有程序老版删除
```

## 查看那些已安装的程序需要更新

```shell
brew outdated
```

# 其它Homebrew指令

```shell
brew list   —列出已安装的软件
brew update   —更新Homebrew
brew home  *—用浏览器打开
brew info   *—显示软件内容信息
brew deps * — 显示包依赖
brew server *  —启动web服务器，可以通过浏览器访问http://localhost:4567/ 来同网页来管理包
brew -h brew   —帮助
```



# brew install XXX一直卡在Updating Homebrew

出现这个的时候， 先Ctrl+C , 然后在执行，就不会卡了， 就是这么囧的。