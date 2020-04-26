---
title:       "tasks.json"
subtitle:    ""
description: "任务模式的脚本"
date:        2020-03-09
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["开发工具", "tasks.json"]
categories:  ["Tech" ]
---

[TOC]

**官方文档：**

**https://code.visualstudio.com/docs/editor/debugging   **

**https://code.visualstudio.com/docs/editor/tasks**

# 简介

通过命令， 进行一键处理相关程序处理。 

# 文件模板

转载地址：https://blog.csdn.net/ustczhng2012/article/details/102580968?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task

```java
{
    "version": "2.0.0",
    //每次执行都启动一个新的控制台
    "presentation": {
        "reveal": "always",
        "panel": "new",
        "echo": true
    },
    //设置环境变量
    "options": {
        "env": {
            "LINUX_SRC_HOME": "/home/user/system/packages/services/Car/evs",
            "LOCAL_SRC_HOME": "${workspaceRoot}"
        }
    },
    "type": "shell",
    "problemMatcher": {
        "owner": "vs_code",
        "fileLocation": [
            "relative",
            "${workspaceRoot}"
        ],
        "pattern": {
            "regexp": ".*(app/.*|project/.*):(\\d+):(\\d+):\\s+(warning|error):\\s+(.*)$",
            "file": 1,
            "line": 2,
            "column": 3,
            "severity": 4,
            "message": 5
        }
    },
    //任务列表
    "tasks": [
        {
            "label": "01.[同步代码]本地代码->Linux远程服务器", // 取个名字
            "command": "${workspaceRoot}\\.vscode\\sync_code.cmd",
            "args": [
                "native",
                "False"
            ],
            "identifier": "CodeSync",
            "taskClassify": "同步代码"
        },
		
        {
            "label": "02.[同步代码并获取修改文件列表]本地代码-->Linux远程服务器",
            "command": "${workspaceRoot}\\.vscode\\sync_code.cmd",
            "args": [
                "native",
                "True"
            ],
            "identifier": "CodeSyncDiff",
            "taskClassify": "同步代码"
        },	
		
        {
            "label": "03.[编译IT]在Linux远程服务器上编译IT工程",
            "dependsOn": "CodeSync",
            "command": "${workspaceRoot}\\.vscode\\build_obj.cmd",
            "args": [
                "test",
                "DTCenter.out",
                "it_cfg",
                "Debug",
                "-j8",
                "cache"
            ],
            "taskClassify": "编译IT工程"
        },
        {
            "label": "04.[同步+编译+IT]在Linux远程服务器上构建IT工程并运行",
            "dependsOn": "CodeSync",
            "command": "${workspaceRoot}\\.vscode\\build_and_run_IT.cmd ratmng.nrom.cfgslave",
            "taskClassify": "同步+编译+IT工程"
        },     
        {
            "label": "05.[静态检查]代码静态检查",
            "dependsOn": "CodeSyncDiff",
            "command": "${workspaceRoot}\\.vscode\\inc_build_flint.cmd",
            "taskClassify": "flint"
        },
        {
            "label": "06.[增量构建] 代码增量compile",
            "dependsOn": "CodeSyncDiff",
            "command": "${workspaceRoot}\\.vscode\\inc_build_compile.cmd",
            "taskClassify": "增量编译"
        }
    ]
}
```

# 好处

熟悉这一套后， 自定义Maven 或者是 Shell的模板， 自定义一套启动程序或者其他的命令。 更好的感觉是在哪里写一套shell脚本， 然后自动一键运行处理。 

# 运行

![Xnip2020-03-09_14-57-15](/img/Xnip2020-03-09_14-57-15.png)

![Xnip2020-03-09_14-57-44](/img/Xnip2020-03-09_14-57-44.png)

