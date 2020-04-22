---
title:       "vscode-常用插件集合"
subtitle:    ""
description: "runcode，koroFileHeader，Jira，Checkstyle，注释, 代码检查，前端调试插件集合"
date:        2020-04-20
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["开发工具"]
categories:  ["Tech" ]
---

[TOC]

# runcode

## 简介

Vscode runcode插件支持运行C, C++, Java, JS, PHP, Python, Perl, Ruby, Go, Lua, Groovy, PowerShell, CMD, BASH, F#, C#, VBScript, TypeScript, CoffeeScript, Scala, Swift, Julia, Crystal, OCaml, R, AppleScript, Elixir, VB.NET, Clojure, Haxe, Objective-C, Rust等。实际上就是帮你运行cmd上的指令，需要自行配置好本地环境，不然不起效。C/c++使用mingw中的gcc/g++执行，其中shell脚本，用git-bash中的bash命令来执行。

## 用途

作为编辑器相对编译器轻量，脚本部分允许选中部分代码进行执行，方便调试。 适合用来执行写简单代码，脚本，方便测试。

## settings.json配置

```json
"code-runner.executorMap": {
        "go": "cd $dir && go run .",
        "java": "cd $dir && javac $fileName && java $fileNameWithoutExt"
    },

"code-runner.executorMapByGlob": {
        "$dir\\*.go": "go"
    },
```



## java示例

![Xnip2020-03-09_16-52-15](../../../../img/Xnip2020-03-09_16-52-15.png)

## GO示例

![Xnip2020-03-10_10-06-51](../../../../img/Xnip2020-03-10_10-06-51.png)



# 代码注释插件koroFileHeader

插件地址：https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE

## 常规配置

```json
  //--------------------koroFileHeader插件设置---------------------
  "fileheader.customMade": {
    // 头部注释
    "Description": "请输入....",
    "Author": "麦子",
    "Date": "Do not edit", // 文件创建时间(不变)
    "FilePath": "Do not edit", // 文件在项目中的相对路径 自动更新
    "LastEditTime": "Do not edit", // 文件最后编辑时间
    "LastEditors": "Do not edit", // 文件最后编辑者
  },
  "fileheader.cursorMode": {
    // 函数注释
    "Description": "方法说明....",
    "Date": "Do not edit", // 文件创建时间(不变)
    "param": "",
    "return": "",
    "LastEditors": "Do not edit" // 文件最后编辑者
  },
  "fileheader.configObj": {
    "createHeader": true, // 默认打开
    "wideSame": true, // 设置为true开启  头部注释等宽设置
    "autoAdd": true, // 自动添加头部注释开启才能自动添加
    "autoAlready": true, // 默认开启
    "CheckFileChange": true, // 默认关闭  单个文件保存时进行diff检查
    "showErrorMessage": true, // 默认不显示错误通知 用于debugger
    "autoAddLine": 100,// 默认文件超过100行就不再自动添加头部注释
    "moveCursor": true, // 移动光标到`Description :`所在行
    "prohibitAutoAdd": [
      "json",
      "html",
      "js",
      "css",
      "md"
    ], // 禁止.json文件，自动添加头部注释
    "headInsertLine": {
      "java": 2 // java 后缀的文件，在第二行插入文件头部注释
    },
    "annotationStr": {
      "head": "/*", // 自定义注释头部
      "middle": " * @", // 自定义注释中间部分(注意空格,这也是最终生成注释的一部分)
      "end": " */", // 自定义注释尾部
      "use": true, // 是否使用自定义注释符号
   }
},
```

## 文件头部注释快捷键

```
ctrl+cmd+i
```

## 函数注释快捷键

```
ctrl+alt+f #原来这个快捷键
修改为
//        #这个快捷键
```



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

可以看到默认自带的sun和google的check模板。 可以设置为自动检测

```properties
"java.checkstyle.autocheck": false,
```



# Debug Visualizer

Debug Visualizer 可视化调试， 这个用来看数据结构调试。安装此扩展程序后，使用命令Debug Visualizer: New View打开一个新的可视化器视图。在此视图中，您可以输入一个表达式，在逐步浏览应用程序时可以对其进行评估和可视化。

**这个才学习数据结构的时候，可以直观的看数据结构类型。** 

# Live server

启动具有实时重新加载功能的本地开发服务器，以处理静态和动态页面。实时的查看修改的前端页面效果。 



# Debugger for Chrome

调试前端脚本

# Browser Preview

VS Code的浏览器预览功能使您可以在编辑器中打开可以调试的真实浏览器预览。

# SonarLint

SonarLint是一个IDE扩展，可帮助您在编写代码时检测和修复质量问题，目前直接用的是自带的checkStyle插件。   





