---
title:       "vccode"
subtitle:    ""
description: ""
date:        2019-04-27
author:      ""
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["开发工具", "vccode"]
categories:  ["Tech" ]
---

[TOC]

# 理解工作区

比如你常用vscode来进行java和go语言的开发， 这个时候你肯定安装了很多的插件，这个时候在配置的时候，系统默认设置（不可修改）-用户设置-工作区设置-文件夹设置，如果你只是在工作区设置的话，那么加载不同的环境的时候，只是加载java或者go语言需要的插件，这样的话，vscode占用的性能也就少了。 

**注意：如果打开工作区，会把工作区里面的所有的项目都打开，这样的话，一般开发，可以把所有需要相关的项目都放到工作区，然后直接打开。** 

如下打开toolsOpe.code-workspace工作空间，这个工作空间就打开了3个项目。

```shell
➜  toolsOperating cat toolsOpe.code-workspace 
{
    "folders": [
                {
                        "path": "HelloDemo"
                },
                {
                        "path": "test"
                },
                {
                        "path": "maventest"
                }
        ]
}%                                                                                                                                        
➜  toolsOperating 
```

**注意：我们在创建项目的时候，注意终端显示的路径， 因为在创建项目的时候，都是已这个路径用命令来执行的。**



# launch.json 

程序运行时候的相关配置，运行时候可以添加相关参数。

# 配置java,maven环境 

```json
{
    "java.home": "/Library/Java/JavaVirtualMachines/jdk1.8.0_101.jdk/Contents/Home",
    "java.configuration.maven.userSettings": "/Users/maizi/apache-maven-3.3.9/conf/settings.xml",
    "maven.terminal.useJavaHome": true,
    "maven.terminal.customEnv": [
        {
            "environmentVariable": "JAVA_HOME",
            "value": "/Library/Java/JavaVirtualMachines/jdk1.8.0_101.jdk/Contents/Home"
        }
    ]
}
```

# 快捷键

```yaml
#针对mac系统

ctrl+cmd+t  #生成方法注释
ctrl+cmd+i  #生成类注释
⌘+  #方法展开
⌘-  #方法缩起
⌘+shift+o #查找文件
⌘+g  #在一个类里面找字符串

```



# 常用插件

## Remote VSCode  

















