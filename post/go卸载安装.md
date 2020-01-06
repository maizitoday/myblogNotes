---
title:       "go环境配置"
subtitle:    ""
description: ""
date:        2020-01-01
author:      "麦子"
image:       "https://get.pxhere.com/photo/nature-wilderness-ground-prairie-cute-looking-wildlife-wild-fur-portrait-small-usa-mammal-squirrel-rodent-fauna-vertebrate-herbivorous-marmot-north-dakota-prairie-dog-black-tailed-prairie-dog-theodore-roosevelt-national-park-1280413.jpg"
tags:        ["go语言基础"]
categories:  ["Tech" ]
---

[TOC]

转载地址：https://www.jianshu.com/p/ad57228c6e6a

# 卸载

查看路径在哪里

```shell
> which go   
```

**获取root权限**

删除/usr/local下的go目录。 备注: 这个目录是安装go的时候自动生成的. 如果删除完, 使用 go version, 会报找不到go命令

删除mac os x的安装包安装的文件, 删除 /etc/paths.d/go 文件

```shell
> rm -rf /usr/local/go
> rm -rf /etc/paths.d/go
```

删除go的环境配置文件

在/etc/profile 或者 $HOME/.profile 或者 $HOME/.bahs_profile中删除bin的设置

```shell
> vim ~/.bash_profile
> source .bash_profile
```

# 安装

下载安装包https://studygolang.com/dl，按步骤安装即可。

```go
➜  ~ go version
go version go1.13.5 darwin/amd64

➜  ~ go env
GO111MODULE=""
GOARCH="amd64"
GOBIN="/Users/maizi/go/bin"
GOCACHE="/Users/maizi/Library/Caches/go-build"
GOENV="/Users/maizi/Library/Application Support/go/env"
GOOS="darwin"
GOPATH="/Users/maizi/go"
GOROOT="/usr/local/go"
```

# 配置环境变量

```shell
export GOPATH=/Users/maizi/go
export GOROOT=/usr/local/go
export PATH=$PATH:$GOROOT/bin
```

# 结构说明

- [ ] `GOROOT`：安装目录（go语言安装目录）
- [ ] `GOPATH`：工程目录（自己工程项目目录）
- [ ] `GOBIN`：可执行文件目录
- [ ] `PATH`：将go可执行文件加入PATH中，使GO命令与我们编写的GO应用可以全局调用。

`GOPATH`包含三个目录：`bin`、`pkg`、`src`。

- [ ] `src`目录：源文件
- [ ] `pkg`目录：编译好的库文件，主要是*.a文件。
- [ ] `bin`目录：可执行文件。

# 注意

千万不要把`GOPATH`设置成go的安装路径。

# VSCODE配置

先安装Go的插件， 然后双击一个写好的go文件， 按照提示安装所有的插件提示，安装成功后，目录结构如下：

![Xnip2020-01-02_10-03-27](/img/Xnip2020-01-02_10-03-27.png)

 然后配置setting文件

```json
//-----------------------------------------
    "go.buildOnSave": "workspace",
    "go.lintOnSave": "package",
    "go.vetOnSave": "package",
    "go.buildTags": "",
    "go.buildFlags": [],
    "go.lintFlags": [],
    "go.vetFlags": [],
    "go.coverOnSave": false,
    "go.useCodeSnippetsOnFunctionSuggest": false,
    "go.formatOnSave": true,
    "go.formatTool": "goreturns",
    "go.goroot": "/usr/local/go",
    "go.gopath": "/Users/maizi/go",
    "go.gocodeAutoBuild": false,
//-----------------------------------------
```

然后点击 Run Code运行即可。  