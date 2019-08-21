---
title:       "dockerfile常用命令"
subtitle:    ""
description: ""
date:        2019-03-27
author:      "麦子"
image:       "https://img.zhaohuabing.com/in-post/2018-05-23-service_2_service_auth/background.jpg"
tags:        ["docker","dockerfile"]
categories:  ["Tech" ]
---



[TOC]

### DockerFile作用

Dockerfile是由一系列命令和参数构成的脚本，这些命令应用于基础镜像并最终创建一个新的镜像。比用命令更加的灵活。 

### Dockerfile 语法

转载地址： https://www.cnblogs.com/boshen-hzb/p/6400272.html

#### ADD

ADD命令有两个参数，源和目标。它的基本作用是从源系统的文件系统上复制文件到目标容器的文件系统。如果源是一个URL，那该URL的内容将被下载并复制到容器中。

```yaml
#Usage: ADD [source directory or URL] [destination directory]
#ADD /my_app_folder /my_app_folder` 
```

#### CMD

和RUN命令相似，CMD可以用于执行特定的命令。和RUN不同的是，这些命令不是在镜像构建的过程中执行的，而是在用镜像构建容器后被调用。

CMD的作用是作为执行container时候的默认行为（容器默认的启动命令）,当运行container的时候声明了command，则不再用image中的CMD默认所定义的命令, 一个Dockerfile中只能有一个有效的CMD，当定义多个CMD的时候，只有最后一个才会起作用.

```yaml
支持三种格式
CMD ["executable","param1","param2"] 使用 exec 执行，推荐方式；
CMD command param1 param2 在 /bin/sh 中执行，提供给需要交互的应用；
CMD ["param1","param2"] 提供给 ENTRYPOINT 的默认参数；
指定启动容器时执行的命令，每个 Dockerfile 只能有一条 CMD 命令。如果指定了多条命令，只有最后一条会被执行。
如果用户启动容器时候指定了运行的命令，则会覆盖掉 CMD 指定的命令。
```



#### ENTRYPOINT

配置容器启动后执行的命令，并且不可被 docker run 提供的参数覆盖。每个 Dockerfile 中只能有一个 ENTRYPOINT，当指定多个时，只有最后一个起效。

```yaml
两种格式：
ENTRYPOINT ["executable", "param1", "param2"]
ENTRYPOINT command param1 param2（shell中执行）。
```

   

#### COPY

格式为 COPY <src> <dest>,复制本地主机的 <src>（为 Dockerfile 所在目录的相对路径）到容器中的 <dest>。当使用本地目录为源目录时，推荐使用 COPY。



#### ENV

ENV命令用于设置环境变量。这些变量以”key=value”的形式存在，并可以在容器内被脚本或者程序调用。这个机制给在容器中运行应用带来了极大的便利。

```yaml
# Usage: ENV key value
ENV JAVA_HOME /usr/local/jdk1.7.0_80
ENV CLASSPATH ${JAVA_HOME}/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV PATH $PATH:${JAVA_HOME}/bin
```



#### EXPOSE

EXPOSE用来指定端口，使容器内的应用可以通过端口和外界交互。

```yaml
# Usage: EXPOSE [port]
EXPOSE 8080
# EXPOSE命令只是声明了容器应该打开的端口并没有实际上将它打开!
# docker run -P 只有运行的时候，才会打开这个端口映射 

```



#### FROM

FROM命令可能是最重要的Dockerfile命令。改命令定义了使用哪个基础镜像启动构建流程。基础镜像可以为任意镜 像。如果基础镜像没有被发现，Docker将试图从Docker image index来查找该镜像。FROM命令必须是Dockerfile的首个命令。

```yaml
# Usage: FROM [image name]
FROM ubuntu 
```



#### MAINTAINER

我建议这个命令放在Dockerfile的起始部分，虽然理论上它可以放置于Dockerfile的任意位置。这个命令用于声明作者，并应该放在FROM的后面。

```yaml
# Usage: MAINTAINER [name]
MAINTAINER authors_name 
```



#### RUN

每一个RUN指令都会是在一个新的container里面运行,并提交为一个image作为下一个RUN的base

```yaml
RUN命令有两种格式

1. RUN <command>
2. RUN ["executable", "param1", "param2"]
第一种后边直接跟shell命令

# 如果-c 选项存在，命令就从字符串中读取。如果字符串后有参数，他们将会被分配到参数的位置上，从$0开始。
在linux操作系统上默认 /bin/sh -c 

在windows操作系统上默认 cmd /S /C

第二种是类似于函数调用。

可将executable理解成为可执行文件，后面就是两个参数。
```



#### USER

USER命令用于设置运行容器的UID。

```yaml
# Usage: USER [UID]
USER 751
```



#### VOLUME

通过dockerfile的 VOLUME 指令可以在镜像中创建挂载点，这样只要通过该镜像创建的容器都有了挂载点。还有一个区别是，通过 VOLUME 指令创建的挂载点，无法指定主机上对应的目录，是自动生成的。

```yaml
#多种写法
VOLUME ["/data"]
VOLUME /var/log
VOLUME /var/log /var/db

我们通过docker inspect 查看通过该dockerfile创建的镜像生成的容器，可以看到映射到本地的一个地址
```



#### WORKDIR

设置工作目录,为后续的RUN、CMD、ENTRYPOINT指令配置工作目录。可以使用多个WORKDIR指令，后续命令如果参数时相对路径，则会基于之前命令指定的路径.



### 如何使用Dockerfiles

使用Dockerfiles和手工使用Docker Daemon运行命令一样简单。脚本运行后输出为新的镜像ID。

```yaml
# Build an image using the Dockerfile at current location
# Example: sudo docker build -t [name] .
sudo docker build -t my_mongodb . 
```



#### Dockerfile RUN，CMD，ENTRYPOINT命令区别

转载地址：https://www.jianshu.com/p/f0a0f6a43907

Dockerfile中RUN，CMD和ENTRYPOINT都能够用于执行命令，下面是三者的主要用途：

- RUN命令执行命令并创建新的镜像层，通常用于安装软件包
- CMD命令设置容器启动后默认执行的命令及其参数，但CMD设置的命令能够被`docker run`命令后面的命令行参数替换
- ENTRYPOINT配置容器启动时的执行命令（不会被忽略，一定会被执行，即使运行 `docker run`时指定了其他命令）

### DockerFile构建mycat

 转载地址： https://hub.docker.com/r/longhronshens/mycat-docker/dockerfile

创建Dockerfile文件(文件名称要大写)

```yaml
FROM centos
RUN echo "root:root" | chpasswd
RUN yum -y install net-tools

# install java
ADD http://mirrors.linuxeye.com/jdk/jdk-7u80-linux-x64.tar.gz /usr/local/
RUN cd /usr/local && tar -zxvf jdk-7u80-linux-x64.tar.gz && ls -lna

ENV JAVA_HOME /usr/local/jdk1.7.0_80
ENV CLASSPATH ${JAVA_HOME}/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV PATH $PATH:${JAVA_HOME}/bin

#install mycat
ADD http://dl.mycat.io/1.6-RELEASE/Mycat-server-1.6-RELEASE-20161028204710-linux.tar.gz /usr/local
RUN cd /usr/local && tar -zxvf Mycat-server-1.6-RELEASE-20161028204710-linux.tar.gz && ls -lna

VOLUME /usr/local/mycat/conf

EXPOSE 8066 9066
#EXPOSE 7066

CMD /usr/local/mycat/bin/mycat console
```

切换到这个文件的目录执行

```yaml
#编译
docker build -t mycat .

#build --tag, -t:** 镜像的名字及标签，
#通常 name:tag 或者 name 格式；可以在一次构建中为一个镜像设置多个标签

#运行
docker run -P     #-p 大写， 会默认有一个为容器的对应端口出来


```





