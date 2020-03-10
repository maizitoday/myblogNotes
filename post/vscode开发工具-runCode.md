---
title:       "runCode"
subtitle:    ""
description: ""
date:        2020-03-09
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["开发工具", "runCode"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：https://blog.csdn.net/zheng2263/article/details/83856709**

# 简介

Vscode runcode插件支持运行C, C++, Java, JS, PHP, Python, Perl, Ruby, Go, Lua, Groovy, PowerShell, CMD, BASH, F#, C#, VBScript, TypeScript, CoffeeScript, Scala, Swift, Julia, Crystal, OCaml, R, AppleScript, Elixir, VB.NET, Clojure, Haxe, Objective-C, Rust等。实际上就是帮你运行cmd上的指令，需要自行配置好本地环境，不然不起效。C/c++使用mingw中的gcc/g++执行，其中shell脚本，用git-bash中的bash命令来执行。

# 用途

作为编辑器相对编译器轻量，脚本部分允许选中部分代码进行执行，方便调试。 适合用来执行写简单代码，脚本，方便测试。

# settings.json配置

```json
"code-runner.executorMap": {
        "go": "cd $dir && go run .",
        "java": "cd $dir && javac $fileName && java $fileNameWithoutExt"
    },

"code-runner.executorMapByGlob": {
        "$dir\\*.go": "go"
    },
```



# java示例

![Xnip2020-03-09_16-52-15](/img/Xnip2020-03-09_16-52-15.png)

# GO示例

![Xnip2020-03-10_10-06-51](/img/Xnip2020-03-10_10-06-51.png)

