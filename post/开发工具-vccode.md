---

title:       "vccode-java环境和git操作"
subtitle:    ""
description: ""
date:        2019-07-24
author:      ""
image:       "https://c.pxhere.com/images/aa/a6/2c425d9c6f81e52b34d87e5c3989-1424141.jpg!d"
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



# git操作

## git属性基本配置

### 查看全局配置

```shell
#这个文件也同时在你当前用户的
cat ~/.gitconfig
 
#查看你的git配置
git config --global --list
```

### 修改当前项目配置

```shell
cd  /Users/maizi/Desktop/demo/.git/config  #进入到当前项目目录的.git文件夹下面，修改config文件

#添加如下基本信息
[user]
	name = yubo
	email = maizi_today@sina.com
[commit]
	template = /Users/maizi/Desktop/demo/src/main/resources/commitMsg
```

### 查看当前项目配置

```
 git config --local --list
```



### git设置远程地址

```shell
git remote add origin https://github.com/maizitoday/vscodegitdemo.git
```

查看插件效果：

![Xnip2019-07-22_17-28-44](/img/Xnip2019-07-22_17-28-44.png)

这里就可以看到这个远程服务地址， 同时， **打圆圈的地方可以点击，直接跳转到git服务web页面。**



### 远程分支同步到当前本地分支

```shell
git pull origin master  #将指定远程分支同步到当前本地分支

From https://github.com/maizitoday/vscodegitdemo
 * branch            master     -> FETCH_HEAD
```



## 本地项目提交远程服务

```shell
➜ content git init
➜ content git:(master) ✗ git remote add origin https://github.com/maizitoday/myblogNotes.git
➜ gitdemo git:(master) git pull origin master
➜ content git:(master) ✗ git add .
➜ content git:(master) ✗ git commit -am '第一次提交'
➜ content git:(master) git push -u origin master

// 然后在修改具体项目的git配置如下：
[user]
	name = yubo
	email = maizi_today@sina.com
[commit]
	template = /Users/maizi/Desktop/demo/src/main/resources/commitMsg
```

## 

## 远程项目下载到本地

```shell
git clone https://github.com/maizitoday/vscodegitdemo
```



## git颜色代表

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



## GitLens插件基本操作



### FILE HISTORY

**文件历史记录**视图以可视化，导航和浏览当前文件的修订历史记录。可以查看这个文件的所有的修改记录。如图：

![Xnip2019-07-24_11-45-45](/img/Xnip2019-07-24_11-45-45.png)



### LINE HISTORY

行历史记录视图以可视化，导航和浏览当前文件的**选定行**的修订历史记录。他主要看的是具体哪一行的git记录。如图：

![Xnip2019-07-24_11-47-38](/img/Xnip2019-07-24_11-47-38.png)

可以看到这一行，有两次的修改， 如果鼠标点击到上面一行的话效果如下：

![Xnip2019-07-24_11-48-47](/img/Xnip2019-07-24_11-48-47.png)

可以明显看到， 这一行没有只有一条git记录。



### SEARCH COMMITS

查询出**提交注释**的关键词查询，如下图：

![Xnip2019-07-24_11-56-23](/img/Xnip2019-07-24_11-56-23.png)

这里可以看到， 有两个个文件的提交的时候有**测试**这两个关键字，方便查询提交的时候的关键字，然后查询到这个具体的文件。



### 选定版本进行对比

先选择你需要的版本

![Xnip2019-07-24_12-32-48](/img/Xnip2019-07-24_12-32-48.png)

在选中另外一个版本

![Xnip2019-07-24_12-33-08](/img/Xnip2019-07-24_12-33-08.png)

然后就可以看到两个版本直接的比对了。 



### 导出固定版本的项目

看上图有一个**Explore Repository from Here** 这个选项可以导出这个版本的项目代码



### REPOSITORIES

这里可以看到你现在项目的代码和本机的git直接的代码对比， 也可以和远程git仓库地址对比，如下：

![Xnip2019-07-24_14-55-20](/img/Xnip2019-07-24_14-55-20.png)

上面就是本地git和远程git仓库的对比， 可以在COMPARE栏看到差异， 下面是和本地项目和本地git服务之间的对比：

 ![Xnip2019-07-24_15-01-12](/img/Xnip2019-07-24_15-01-12.png)

同时可以看到， 这里还可以看到这个git的所有的用户有哪一些， 这些用户提交了哪一些， 然后也可以进行代码的对比。同时，他的对比是和当前的工作的这个分支进行对比。 也就是HEAD分支。 



### 单个文件快速查看版本信息

![Xnip2019-07-24_15-28-54](/img/Xnip2019-07-24_15-28-54.png)

这些地方的标致都可以查看这个文件的所有git信息， 同时右上角的那个鱼标的图像，可以看上一个版本， 或者上上一个版本。

###  创建分支

下面直接点击当前分支，然后选择创建分支， 创建完成后，在终端会出现：git branch XXX master然后回车，本地分支已经创建完成。

![Xnip2019-07-24_16-02-26](/img/Xnip2019-07-24_16-02-26.png)

本地创建完成后， 发布到远端服务，点击一下按钮：

![Xnip2019-07-24_16-04-51](/img/Xnip2019-07-24_16-04-51.png)

publish branch（发布分支）将本地分支发布到远端并提交你的代码 。这样就可以看到**remotes栏**会显示出本地上传的远端分支。



### 分支比较

![Xnip2019-07-24_16-07-51](/img/Xnip2019-07-24_16-07-51.png)

通过上面图，可以看到分支和分支之间进行比较。



### 如何选择当前分支

上面创建了很多个分支， 如何确定当前项目用哪一个分支

![Xnip2019-07-24_16-13-13](/img/Xnip2019-07-24_16-13-13.png)

同时也可以指定当前分支和一个固定分支的区别

![Xnip2019-07-24_16-14-05](/img/Xnip2019-07-24_16-14-05.png)

点击中间之后，就可以进行选择一个分支， 然后时刻进行比较。



### 主干和分支合并

![Xnip2019-07-24_16-24-26](/img/Xnip2019-07-24_16-24-26.png)

 从上面图可以看到，你可以多种选择。

 



