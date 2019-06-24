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


Shift+Alt+O #java自动导包

Ctrl + P　　#快速打开最近打开的文件
Alt + up / down　　#移动上下行
Shift + Alt + up / down　　#在当前行上复制上下行
Home　　#光标跳转到当前行头
End　　#光标跳转到当前行尾

Ctr + Home　　#跳转到当前页头
Ctrl + End　　#跳转到当前页尾

Ctrl + B　　#侧边栏隐藏
Ctrl + F　　#查找

Ctrl + H　　#查找替换
Ctrl + /　　#添加单行注释

Shift + Alt + A　　#添加多行注释
```

# git颜色代表

红色，未加入版本控制; (刚clone到本地)
绿色，已经加入版本控制暂未提交; (新增部分)
蓝色，加入版本控制，已提交，有改动； (修改部分)
白色，加入版本控制，已提交，无改动；
灰色：版本控制已忽略文件。

## git文件标识

A: 增加的文件.
C: 文件的一个新拷贝.
D: 删除的一个文件.
M: 文件的内容或者mode被修改了.
R: 文件名被修改了。
T: 文件的类型被修改了。
U: 文件没有被合并(你需要完成合并才能进行提交)
X: 未知状态













