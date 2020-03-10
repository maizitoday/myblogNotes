---
title:       "启动时候怎么设置JVM虚拟机与输出到调试控制台而不是终端"
subtitle:    ""
description: ""
date:        2020-03-09
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["开发工具", "启动时候怎么设置JVM虚拟机"]
categories:  ["Tech" ]
---

[TOC]

# JVM虚拟机配置

点击调试模式， 调出launch.json

```java
{
    "configurations": [
        {
            "type": "java",
            "name": "Spring Boot-DemoApplication<redisapi>",
            "request": "launch",
            "cwd": "${workspaceFolder}",
            "console": "internalConsole",
            "mainClass": "com.example.redisapi.DemoApplication",
            "projectName": "redisapi",
            "args": ""
        }
    ]
}
```

添加下面参数就好

```java
"vmArgs": "-Djava.net.preferIPv4Stack=true"
```

可以看出**vmArgs**就是修改虚拟机的参数。 



# 输出到调试控制台而不是终端

**转载地址：https://blog.csdn.net/LaineGates/article/details/88219585** 

visual studio code每次debug，默认会显示“终端窗口”，但终端窗口会添加很多附属信息，比如启动的程序、参数等等。
但visual studio code的“调试控制台”就很好，每次只显示本次调试的结果。
经过上网查找及尝试，最终发现了解决办法，在launch.json中对应配置中，添加一行：

```java
"console":"none"
```

“console”的值选项包括：

| 选项                 | 输出效果                               |
| -------------------- | -------------------------------------- |
| “none”               | 只输出到"调试控制台"                   |
| “integratedTerminal” | 同时输出到"调试控制台"和软件内置“终端” |
| “externalTerminal”   | 输出到外部“终端”                       |

**具体示例**

```java
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Python: Current File",
            "type": "python",
            "request": "launch",
            "console":"none",
            "stopOnEntry": false,
            "program": "${file}"
        },
        ... // 其他配置
   ]
}

```

**配置用电脑中的哪个终端显示数据**

![Xnip2020-03-09_13-33-42](/img/Xnip2020-03-09_13-33-42.png)

如此配置后， 程序启动的程序输出的数据都会在这个配置好的终端进行显示出来了。 