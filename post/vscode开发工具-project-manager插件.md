---

title:       "project-manager插件"
subtitle:    ""
description: ""
date:        2019-07-24
author:      "麦子"
image:       "https://c.pxhere.com/images/d4/ac/449725f029ba24f1c132e540b906-1542243.jpg!d"
tags:        ["开发工具","project-manager插件"]
categories:  ["Tech" ]
---

**转载地址： <http://www.ibloger.net/article/3289.html>**

# 作用

在项目开发的时候，我们经常需要同时操作多个项目，经常需要切换项目。

以前的方式

- 在工具栏中点击文件，打开，选择本地项目的目录 / 新建窗口
- 如果有最近打开的项目，点击打开最近的文件

这两种方式对于需要经常切换项目时，比较耗时，为解决这个问题，VSCode 提供了 `Project Manager` 插件管理，开发时常用的项目

# 管理项目命令

- `Project Manager: Save Project` 将当前文件夹另存为新项目
- `Project Manager: Edit Project` 手动编辑项目（projects.json）
- `Project Manager: List Projects to Open` 列出所有已保存/检测到的项目并选择一个
- `Project Manager: List Projects to Open in New Window` 列出所有已保存/检测到的项目，然后选择一个在新窗口中打开
- `Project Manager: Refresh Projects` 刷新缓存的项目

如图：

![Xnip2019-07-25_16-54-52](/img/Xnip2019-07-25_16-54-52.png)



# 保存项目

可以随时将当前项目保存在管理器中。使用如下：

```
Project Manager: Save Project 
```

就可以了。 



# 直接设置JSON文件

直接点击，打开查看JSON格式文件， 直接修改其中的name和rootPath

![Xnip2019-07-25_17-00-17](/img/Xnip2019-07-25_17-00-17.png)

设置后， 可以看到在左下方看到当前项目的名称， 点击这个地方，可以进行方便进行项目的切换。 

![Xnip2019-07-25_17-00-35](/img/Xnip2019-07-25_17-00-35.png)

# 项目显示排序

```java
projectManager.sortList

// 显示的时候进行排序
Saved：您保存项目的顺序
Name：您为项目键入的名称
Path：项目的完整路径
Recent：最近使用的项目
```

![Xnip2019-07-25_17-02-24](/img/Xnip2019-07-25_17-02-24.png)



# 添加另外项目到工作区

![Xnip2019-07-25_21-54-36](/img/Xnip2019-07-25_21-54-36.png)

如下我就添加了另外一个项目进来， 进来后，项目就都带上了圆圈。

![Xnip2019-07-25_21-55-50](/img/Xnip2019-07-25_21-55-50.png)

