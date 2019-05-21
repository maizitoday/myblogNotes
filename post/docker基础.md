---
title:       "docker"
subtitle:    ""
description: ""
date:        2019-04-09
author:      "麦子"
image:       "https://img.zhaohuabing.com/in-post/2018-04-16-using-helm-to-deploy-to-kubernetes/buffalo.jpg"
tags:        ["docker", "镜像"]
categories:  ["Tech" ]
---

[TOC]

# 镜像管理



镜像就是一个只读模板， 里面可以有很多东西。    启动起来后， 就是一个容器了。   仓库就是存放镜像的地方。阿里云的加速器。  具体步骤查看阿里云官方网站显示。 常用的    docker search  XXX ,   docker pull  XXX, 给镜像打标签， docker tag  centos  tomcat_centos:tomcat  这样可以用多个镜像了。

# 容器启动



docker ps 显示启动中的容器， docker ps -a  启动所有状态的容器，停止的或者是运行的。 

```yaml
#启动容器， 
docker run -itd  xxx   -i:表示把容器的标准输出打开  
                       -t：表示分配一个为终端，
                       -d:表示后台启动   xxx：表示镜像。
                       --privileged（使用该参数，container内的root拥有真正的root权限）

docker run -itd -p 10001:22 --privileged=true base_centos:latest /usr/sbin/init
```
刚开始接触Docker的朋友，可能会遇到这么一个问题，使用centos7镜像创建容器后，在里面使用systemctl启动服务报错。**Failed to get D-Bus connection: Operation not permitted**

报这个错误的原因： Docker的设计理念是在容器里面不运行后台服务，容器本身就是宿主机上的一个独立的主进程，也可以间接的理解为就是容器里运行服务的应用进程。一个容器的生命周期是围绕这个主进程存在的，所以正确的使用容器方法是将里面的服务运行在前台。

再说到systemd，这个套件已经成为主流Linux发行版（比如CentOS7、Ubuntu14+）默认的服务管理，取代了传统的SystemV风格服务管理。systemd维护系统服务程序，它需要特权去会访问Linux内核。而容器并不是一个完整的操作系统，只有一个文件系统，而且默认启动只是普通用户这样的权限访问Linux内核，也就是没有特权，所以自然就用不了！

因此，请遵守容器设计原则，一个容器里运行一个前台服务！



# 保存镜像到本地

```yaml
docker save spring-boot-docker  -o  /home/wzh/docker/spring-boot-docker.tar
```



# 加载本地镜像

```yaml
docker load -i spring-boot-docker.tar  
```



# Docker命令



## CP

```yaml
docker cp RS-MapReduce 30026605dcfe:/home/cloudera

将容器30026605dcfe的/home/cloudera/RS-MapReduce目录拷贝到主机的/tmp目录中。

docker cp  30026605dcfe:/home/cloudera/RS-MapReduce /tmp/
```



## run

- **-i:** 以交互模式运行容器，通常与 -t 同时使用；
- **-t:** 为容器重新分配一个伪输入终端，通常与 -i 同时使用；
- **-d:** 后台运行容器，并返回容器ID；
- **-p:** 端口映射，格式为：**主机(宿主)端口:容器端口**
- **-e username="ritchie":** 设置环境变量；
- **--link=[]:** 添加链接到另一个容器；
- **--expose=[]:** 开放一个端口或一组端口；
- **--name="nginx-lb":** 为容器指定一个名称；
- **-h "mars":** 指定容器的hostname；Linux中直接 hostname 查看主机名称

```dockerfile
--restart具体参数值详细信息：
no -  容器退出时，不重启容器；
on-failure - 只有在非0状态退出时才从新启动容器；
always - 无论退出状态是如何，都重启容器；
如果创建时未指定 --restart=always ,可通过update 命令设置
docker update --restart=always xxx     
```

### --rm

在Docker容器退出时，默认容器内部的文件系统仍然被保留，以方便调试并保留用户数据。但是，对于foreground容器，由于其只是在开发调试过程中短期运行，其用户数据并无保留的必要，因而可以在容器启动时设置--rm选项，这样在容器退出时就能够自动清理容器内部的文件系统。显然，--rm选项不能与-d同时使用，即只能自动清理foreground容器，不能自动清理detached容器。



## logs

- **-f :** 跟踪日志输出
- **--since :**显示某个开始时间的所有日志
- **-t :** 显示时间戳
- **--tail :**仅列出最新N条容器日志



## inspect

```dockerfile
docker inspect 容器id：#查看到容器的相关信息，这里可以看到这个容器所有信息，网络，数据卷，端口等

docker volume ls
docker inspect mysshvolume:  #查看数据卷的相关信息
```



#   删除镜像 



在使用Docker的时候遇到删不掉image的情况，如下： docker  rmi  XXX（镜像名称）：YYY（镜像tag）

```yaml
1. 先查询记录 docker ps -a
2. 把该镜像的记录全部删除掉，如果删除所有镜像的记录，
     可以使用:docker ps -a|awk '{print $1}'|xargs docker rm
3. docker rmi 5e4f2da203e2就可以了
删除tag就是删除这个tag的镜像 ， 如果是删除imageId的话，就是这个镜像的id，所有imageId的都可以删除。镜像里面的  tag有不同的， 但是imageId是相同的。
```
# 通过容器创建镜像   



![4-9-20](/img/4-9-20.png)

# 使用模板创建镜像

![4-9-21](/img/4-9-21.png)

# docker容器管理

![4-9-22](/img/4-9-22.png)

启动时候，修改名字， 然后进入容器的时候，可以直接用名字进入容器中。 docker exec -it   xxxx(容器名字)  bash

![4-9-23](/img/4-9-23.png)



# 上传到自己的私有仓库



![4-9-24](/img/4-9-24.png)

上传到docker库里面： 上传的时候如果出现denied: requested access to the resource is denied 这个错误， 你需要的是把镜像名字前面改成maizi1989/mz_base_centos7，

```yaml
  其中maizi1989这个是docker hub上面的账户。 

  REPOSITORY： 是这个名字前面需要加入maizi1989

  docker tag 6824c2a39c01  maizi1989/base_nginx   这个可以直接修改。
```


# 数据卷

数据管理:   用户在使用 Docker 的过程中，往往需要能查看容器内应用产生的数据，或者需要把容器内的数据进行备份，甚至多  个容器之间进行数据的共享，这必然涉及容器的数据管理操作。容器中管理数据主要有两种方式：数据卷(Data Volumes)，数据卷容器(Data Volume Containers)。

 docker run -itd -p 10001:22 -v /docker_data:/opt/webdata  --privileged=true base_centos:latest /usr/sbin/init 

 其中 /docker_data 表示宿主的目录，  /opt/webdata表示容器里面的目录。  这两个目录是一种数据共享目录。

 docker run -itd -p 10001:22 -v /docker_data:/opt/webdata**:ro**  --privileged=true base_centos:latest /usr/sbin/init 

加入ro就表示这个数据卷只读的。

# 数据卷容器

相当于， 把一个镜像作为一个容器， 然后其他的容器可以直接挂载这个容器， 然后所有挂载的地方对这个数据源进行同步，任何一方进行了改变，其他地方都是可以用的。数据可以进行备份和还原。

# 网络映射处理

 docker run创建Docker容器时，可以用–net选项指定容器的网络模式，Docker有以下4种网络模式： 

```dockerfile
 bridge模式： 使用–net =bridge指定，默认设置；     
             在宿主机上作为一块虚拟网卡使用 ， 默认和宿主机是想通的。 

  host模式：  使用–net =host指定；   
             容器将不会虚拟出自己的网卡，配置自己的IP等，而是使用宿主机的IP和端口。

 none模式：   使用–net =none指定；  
             需要我们自己为Docker容器添加网卡、配置IP等。 

container模式： 使用–net =container:NAMEorID指定。
               这个模式指定新创建的容器和已经存在的一个容器共享一个Network Namespace，而不是和宿主机共享
```


# dockerFile

Docker 镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。 就相当于，把所有的操作的一些命令直接写在这个文本中， 不用每次去敲写。  这个后期在继续学习这个话题。



# docker-compose

  这个就相当于shell脚本， 可以一次性启动很多容器



# 多个容器通信

 解决方法 在运行容器的时候如果想连接其他容器 在运行的时候加上 --link +容器名称

