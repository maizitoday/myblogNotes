---
title:       "vscode-常用插件集合"
subtitle:    ""
description: "前端，注释, 插件"
date:        2020-04-20
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["开发工具"]
categories:  ["Tech" ]
---

[TOC]

# Debug Visualizer

Debug Visualizer 可视化调试， 这个用来看数据结构调试。安装此扩展程序后，使用命令Debug Visualizer: New View打开一个新的可视化器视图。在此视图中，您可以输入一个表达式，在逐步浏览应用程序时可以对其进行评估和可视化。

**这个才学习数据结构的时候，可以直观的看数据结构类型。** 

# Live server

启动具有实时重新加载功能的本地开发服务器，以处理静态和动态页面。实时的查看修改的前端页面效果。 



# Debugger for Chrome

调试前端脚本



# Browser Preview

VS Code的浏览器预览功能使您可以在编辑器中打开可以调试的真实浏览器预览。

# Jira and Bitbucket (Official)

将Jira和Bitbucket的功能引入VS Code-使用Atlassian for VS Code，您可以创建和查看问题，开始解决问题，创建请求请求，进行代码审查，开始构建，获取构建状态等等！

# EditorConfig for VS Code 

主要用于统一编辑代码时的风格，其中主要统一的是缩进的风格。

在当前项目根目录下添加`.editorconfig`文件

[editorconfig](http://editorconfig.org/)是**一种帮助开发者在不同的ide和编辑器之间，定义和维护编码格式的工具**

**示例**

```properties
# EditorConfig is awesome: http://EditorConfig.org

# 表明这是最顶层的配置文件，这样才会停止继续向上查找 .editorconfig 文件；
# 查找的 .editorconfig 文件是从顶层开始读取的，类似变量作用域的效果，内部
# 的 .editorconfig 文件属性优先级更高
root = true

# 指定作用文件格式
[*]

charset = utf-8

# 定义换行符 [lf | cr | crlf]
end_of_line = lf

# 文件是否以一个空白行结尾 [true | false]
insert_final_newline = true

# 是否除去换行行首的任意空白字符
trim_trailing_whitespace = true


# 编码格式。支持latin1、utf-8、utf-8-bom、utf-16be和utf-16le，不建议使用uft-8-bom。
# Set default charset
[*.{js,py,java,html,xml}]
charset = utf-8

# 4 space indentation
[*.{py,java}]

# 缩进的类型 [space | tab]
indent_style = space

# 缩进的大小
# tab_width: 设置整数用于指定替代tab的列数。默认值就是indent_size的值，一般无需指定。
indent_size = 4

# Tab indentation (no size specified)
[Makefile]
indent_style = tab

# Indentation override for all JS under lib directory
[**.{js,html,xml}]
indent_style = space
indent_size = 2

# Matches the exact files either package.json or .travis.yml
[{package.json,.travis.yml}]
indent_style = space
indent_size = 2

[*.md]
indent_size = 4

```

# Checkstyle for Java

代码格式检查。

直接命令

```
check 
```

可以看到默认自带的sun和google的check模板。 

# SonarLint

SonarLint是一个IDE扩展，可帮助您在编写代码时检测和修复质量问题



```
mac`：`ctrl+cmd+i
```

cmd+G 选中一个字符，然后就可以改这个字符的所有的地方了。 

ctrl+G 直接跳转到多少行。 

TODOS用来做什么的。 





