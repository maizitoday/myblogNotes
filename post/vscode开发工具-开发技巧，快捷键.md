---
title:       "vscode开发技巧，快捷键"
subtitle:    ""
description: "java快捷键，Maven快捷键，调试，适应无鼠标操作"
date:        2019-07-25
author:      "麦子"
image:       "https://get.pxhere.com/photo/photography-sea-fog-horizon-water-calm-shore-black-and-white-morning-sky-pier-mist-ocean-haze-fixed-link-1445001.jpg"
tags:        ["开发工具"]
categories:  ["Tech" ]
---

[TOC]



**注意：有些快捷键已经无法实现或者冲突，或者是版本升级后有修改。** 

# 文档资料

https://geek-docs.com/vscode/vscode-tutorials/vscode-create-terminal.html

# 快捷键

**官方具体快捷键：<https://code.visualstudio.com/docs/getstarted/keybindings#_customizing-shortcuts>**

- Command（或 Cmd）⌘
- Shift ⇧
- Option（或 Alt）⌥
- Control（或 Ctrl）⌃
- Caps Lock ⇪
- Fn
- ⌃ （Control 键）option+Shift+i



## 文件快捷键



### 快速最近打开文件

键盘快捷键：⌘P，同时这个也是可以搜索项目里面的文件和类的。



### 查找项目具体类和文件

查找项目中的一个java类：⇧⌘O



### 查看源码包里面的类

比如查看java.lang.String的类： ⌘O



### 并排编辑

键盘快捷键：⌘\    ,  把文件分割成多个。 



### 在编辑之间切换

键盘快捷键：⌘1，，2，⌘3



### 关闭当前打开的文件夹

键盘快捷键：⌘W



### 切换边栏

键盘快捷键：⌘B





### 关闭当前打开的文件夹

键盘快捷键：⌘W， 快速关闭你打开的java类或者其他。 



### 代码折叠

键盘快捷键：⌥⌘[  和  ⌥⌘]

折叠全部（⌘K⌘0）折叠编辑器中的所有区域。

展开全部（⌘K⌘J）展开编辑器中的所有区域。



### 删除一行

ctrl + K 



### 选中一行

⌘+ > ,然后shift+向上箭头



### 复制一行

shift + alt + 向下箭头



### 切换集成终端

在快捷键中搜索终端，然后设置快捷键就好。 



### 跳转多少行

ctrl+G   直接跳转到多少行，直接输入就好， 

### 打开Dash工具

这个直接就打开Dash工具， 查看当前的这个值的API



### 多个字符统一修改

选中一个字符，cmd+G   ，然后就可以改这个字符的所有的地方了。  shift + ⌘ + L 



### 自动添加方法注释

ctrl + alt + F  快捷键



### 搜索和替换

对所有字符都可以进行搜索和替换： ⇧⌘R



### 文件类搜索

选中一个字符串， 然后按下 ：⌘G 可以显示出这个文件的所有的这个字符出来， 比 ⌘F要方便一点



查看源码固定类

⌘+O，然后输入你要查看的类，比如 String，这样就可以查看String的源码类了。



### 导航到特定行

键盘快捷键：⌘L



### 终端





### 快速跳转到项目中的错误和警告

快速跳转到项目中的错误和警告： ⇧⌘M



### 缩进

修改设置，默认设置4个空格。

```
    "editor.insertSpaces": true,
    "editor.tabSize": 4,
```

## 

## 切换左边工具栏

键盘快捷键：⌘B ， 就是隐藏和显示你左边的那个项目栏那一块的显示。**注意你的鼠标要在左侧那一块聚焦。**

## 快速打开vscode设置

快捷键：⌘，



## 查看Git输出

 ⇧⌘U， 并在下拉列表中选择**Git**。

## 

## 同时打开多个文件

![grid-layout](/img/grid-layout.gif)



## java快捷键

**官网地址：<https://code.visualstudio.com/docs/java/java-editing>**

### 快速寻找java启动类和Controller入口

要在当前文件中搜索符号，请使用**快速打开**（⌘P），然后输入“@”命令，然后输入您要查找的符号的名称。这个也就是显示出你项目的启动的入口的类。 但是测试时候，需要打开这个项目的一个文件才可以。 如果输入"#"可以查看这个项目的任何类，包括源码包，如java.lang.String的源码类。

 

### 使用Spring Boot导航代码

[Spring Boot Tools](https://marketplace.visualstudio.com/items?itemName=Pivotal.vscode-spring-boot)扩展为Spring Boot项目提供增强的导航和代码完成支持。 

- `@/` 显示所有已定义的请求映射（映射路径，请求方法，源位置）
- `@+` 显示所有已定义的bean（bean名称，bean类型，源位置）
- `@>` 显示所有功能（原型实现）
- `@` 显示代码中的所有Spring注释

![Xnip2019-07-25_22-21-41](/img/Xnip2019-07-25_22-21-41.png)

### 重构

在extract to variable/constant/method （提取到变量/常数/方法）之后触发器的重命名

![3cc43e5d1f304e2e9677b3a7522a7e14](/img/3cc43e5d1f304e2e9677b3a7522a7e14.gif)

### 将局部变量转换为字段

选中一个变量， 然后可以看到旁边有一个小黄灯， 然后点击小黄灯， 你就可以看到下面的两个选项了。

 ![26bd4bf1822044678b7d59b2b932f9f1](/img/26bd4bf1822044678b7d59b2b932f9f1.gif)



同时我们在写变量时候可以偷懒， 如下：

```
 Arrays.asList(1,2,3);
```

我们写上面这个， 然后快捷键出小黄灯，然后就直接创建出变量的完整形式就可以了。小黄灯可以补充完整形式。



### 自动唤醒灯泡

 快捷键： ⌘。

### java自动导包

Shift+Alt+O， 如果是一个对象有多个类的话，会不进行导入，需要你自己去选择处理。 



### 选中一行

 快捷键：⌥↑ 



### 自动保存就格式化java代码

打开设置，找到  Format On Save 选中，搜索 Default Formatter ， 找到你需要的java格式化的模板。 



### 多光标选择

使用⇧⌘L向当前选择的所有实例添加其他游标。**在java中可以一次性选择多个变量，进行修改**。



### 导航到文件的开头和结尾

键盘快捷键：⌘↑和⌘↓, 右边箭头和左边箭头，分别就到这一行的头部和尾部。



### 并排编辑

键盘快捷键：⌘\



### 设置Checkstyle配置文件   

![Xnip2019-07-27_18-23-56](/img/Xnip2019-07-27_18-23-56.png) 



### 上下移动线 

键盘快捷键：⇧⌥↑ 或 ⇧⌥↓



### 自定义代码片段 

https://code.visualstudio.com/docs/editor/userdefinedsnippets 官方文档

```json
{
	/***
	 prefix      :代码片段名字，即输入此名字就可以调用代码片段。
     body        :这个是代码段的主体.需要编写的代码放在这里,　　　　　 
     $1          :生成代码后光标的初始位置.
     $2          :生成代码后光标的第二个位置,按tab键可进行快速切换,还可以有$3,$4,$5.....
     ${1:字符}    :生成代码后光标的初始位置(其中1表示光标开始的序号，字符表示生成代码后光标会直接选中字符。)
     description  :代码段描述,输入名字后编辑器显示的提示信息。
	
	 换行：\r或者\n

	 tab键制表符：\t
	 
	*/
	"controller test": {
		"prefix": "myController",
		"body": [
			"@GetMapping(value = \"showMsg\")",
			"public String showMsg(@RequestParam String str) {",
			"\tSystem.out.println(\"欢迎你\");",
			"\treturn ${1:\"hello controller\"};",
			"}",
			"$2"
		],
		"description": "用于测试Controller的流程"
	}
}
```

显示效果如下：

![Xnip2019-07-27_21-32-10](/img/Xnip2019-07-27_21-32-10.png)



### 查看target里面的class文件   

查看第三方的包或者是java源码的class是OK的， 但是， 查看maven package中target下面的class文件，确打不开， 有时候我们需要查看这个地方的class文件，现在直接用，打开终端， 然后用下面这个命令打开。 因为我的class文件默认是用JD-GUI的软件打开的。

```shell
demo open /Users/maizi/Desktop/demo/target/classes/com/example/demo/DemoApplication.class
```

现在不知道有什么好的方法，如果后面发现，在进行更新处理。



## maven快捷键



### 自动搜索maven库里面包

![Xnip2019-07-25_23-06-55](/img/Xnip2019-07-25_23-06-55.png)

 

### 运行历史命令

1. 命令选项板>选择**Maven：历史** >选择项目>从历史记录中选择命令。
2. 右键单击项目>单击**历史记录** >从历史**记录中**选择命令。



### 运行自定义maven命令

设置工作区配置文件如下：

![Xnip2019-07-25_23-18-37](/img/Xnip2019-07-25_23-18-37.png)

点击项目， 运行 favorites 这个命令就可以了。



## 调试

**官网详细具体操作：<https://code.visualstudio.com/docs/java/java-debugging>**

### 面板详细查看变量数据

![debug_data_inspection](/img/debug_data_inspection.gif)

# 前端

## Emmet使用

https://code.visualstudio.com/docs/editor/emmet 官方文档

他可以让一些大量的HTML标签批量的生成显示出来。 

