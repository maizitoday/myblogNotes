---
title:       "docker-compose.yml常用命令"
subtitle:    ""
description: ""
date:        2019-04-26
author:      "麦子"
image:       "https://img.zhaohuabing.com/in-post/2018-05-23-service_2_service_auth/background.jpg"
tags:        ["docker", "docker-compose.yml"]
categories:  ["Tech" ]
---

[TOC]

转载： https://blog.csdn.net/Anumbrella/article/details/80877643

# Compose模板文件

模板文件是使用Compose的核心，涉及的指令关键字也比较多，大部分指令与`docker run`相关参数的含义都是类似的。默认的模板文件迷城为docker-compose.yml，格式为YAML格式。

比如一个Compose模板文件：

```yaml
version: "2"
services:
	web:
		images:	nginx
		ports:
			- "80:80"
		volumes:
			- "/data"
#volumes:

#networks:
```



## Compose的模板文件主要分为3个区域



### services

服务，在它下面可以定义应用需要的一些服务，每个服务都有自己的名字、使用的镜像、挂载的数据卷、所属的网络、依赖哪些其他服务等等。

### volumes

数据卷，在它下面可以定义的数据卷（名字等等），然后挂载到不同的服务下去使用。

### networks

应用的网络，在它下面可以定义应用的名字、使用的网络类型等等。

**注意：每个服务都必须通过image指令指定镜像或build指令（需要Dockerfile）等来自动构建生成镜像。如果使用build指令，在Dockefile中设置的选项（例如：CMD、EXPOSE、VOLUME、ENV等）将会自动被获取，无需在docker-compose.yml中再次设置。**

# 主要指令的用法



官网指令：https://docs.docker.com/compose/compose-file/#build



## Build

指定Dockerfile所在**文件夹**的路径（可以是绝对路径，或者相对docker-compose.yml文件的路径）。Compose将会利用它自动构建这个镜像，然后使用这个镜像：

```yaml
build: /path/to/build/dir
```



## Command

覆盖容器启动后默认执行的命令：

```yaml
command: echo "hello world"
```



## cgroup_parent

例如，创建了一个cgroup组名称为cgroups_1:

```yaml
cgroup_parent: cgroups_1
```



## container_name

指定容器名称。默认将会使用“项目名称_服务名称_序号”这样的格式。例如：

```yaml
container_name: docker-web-container
```



## devices

指定设备映射关系，例如：

```yaml
devices:
	- "/dev/ttyUSB1:/dev/ttyUSB0"
```



## dockerfile

如果需要，指定额外的编译镜像的Dockerfile文件，可以通过该指定龙指定，例如：

```yaml
dockerfile: Dockerfile-alternate
```

注意：该指令不能跟image同时使用，否则Compose将不知道根据哪个指令来生成最终的服务镜像。



## env_file

从文件中获取环境变量，可以为单独的文件路径或列表。如果通过**docker-compose -f FILE** 方式来指定Compose模板文件，则env_file中变量的路径会基于模板文件路径。

```
env_file: .env

env_file:
	- ./common.env
	- ./apps/web.env
	- /opt/secrets.env
```

环境变量文件中每一行必须符合格式，支持#开头的注释行：

```yaml
# common.ev Set development environment
PROG_ENV=development
```



## environment

设置环境变量，可以使用数组或字典两种格式。只给定名称的变量会自动获取运行Compose主机上对应变量的值，可以用来防止防泄露不必要的数据。例如：

```yaml
environment: 
	RACK_ENV: development
	SESSION_SECRET:
```

**注意**：如果变量名称或者值中用到true|false，yes|no等表达布尔含义的词汇，最好放到引号里，避免YAML自动解析某些内容为对应的布尔语义。



## expose

暴露端口，但不映射到宿主机，只允许能被连接的服务访问。仅可以指定内部端口为参数，如下所示：

```yaml
expose:
	- "3000"
	- "8000"
```



## extends

基于其他模板文件进行扩展。例如，我们已经有了一个webapp服务，定义一个基础模板文件为common.yml，如下所示：

```yaml
# common.yml

webapp:
	build: ./webapp
	environment:
		- DEBUG=false
		- SEND_EMAILS=false
```

再编写一个新的development.yml文件，使用common.yml中的webapp服务进行扩展：

```yaml
# development.yml

web:
	extends:
		file: common.yml
		service: webapp
	ports:
		- "8000:8000"
	links:
		- db
	environment:
		- DEBUG=true
db:
	image: mysql
```

后者会自动继承common.yml中的webapp服务及环境变量定义。

使用extends需要注意以下几点：

- 要避免出现循环依赖，例如A依赖B，B依赖C，C反过来依赖A的情况
- extends不会继承links和volumes_from中定义的容器和数据卷资源

一般情况下，推荐在基础模板中只定义一些可以共享的镜像和环境变量，在扩展模板中具体指定应用变量、链接、数据卷等信息



## external_links

链接到docker-compose.yml外部的容器，甚至可以是非Compose管理的外部容器。参数格式跟links类似：

```yaml
external_links:
	- redis_1
	- project_db_1:mysql
	- project_db_1:postgresql
```



## extra_hosts

类似于Docker中的–add-host参数，指定额外的host名称映射信息，例如：

```yaml
extra_hosts:
	- “googledns:8.8.8.8”
	- "dockerhub:52.1.157.61"
```

会在启动后的服务容器中/etc/hosts文件中添加如下两条条目：

```yaml
8.8.8.8  googledns
52.1.157.61 dockerhub
```



## image

指定为镜像名称或镜像ID。如果镜像本地不存在，Compose将会尝试拉取这个镜像，例如：

```yaml
image: ubuntu
image: mysql
```



## labels

为容器添加Docker元数据（metadata）信息。例如，可以为容器添加辅助说明信息：

```yaml
labels：
	com.startupteam.description: "webapp for a strtup team"
```



## links

链接到其他服务中的容器。使用服务名称（同时作为别名），或者“服务名称:服务别名”（如 SERVICE:ALIAS）,这样的格式都可以，例如：

```yaml
links:
	- db
	- db:database
	- redis
```

使用别名将会自动在服务容器中的/etc/hosts里创建。例如：

```yaml
172.17.2.186  db
172.17.2.186  database
172.17.2.187  redis
```



## net

设置网络模式。参数类似于docker client的–net参数：

```yaml
net: "bridge"
net: "none"
net: "host"
```



## ports

暴露端口信息。使用“宿主机:容器”（如 “HOST:CONTAINER”）格式，或者仅仅指定容器的端口（宿主机将会随机选择端口）：

```yaml
ports：
	- “3000”
	- “8000:8000”
	- "8080:22"
```



## volumes

数据卷所挂载路径设置。可以设置宿主机路径（HOST:CONTAINER）或加上访问模式（HOST:CONTAINER:ro）。该指令中路径支持相对路径。例如：

```yaml
volumes:
	- /var/lib/mysql
	- cache/:/tmp/cache
	- ~/configs:/etc/configs/:ro
```



## volumes_from

从另一个服务或容器挂载它的数据卷：

```yaml
volumes_from:
	- service_name
	- container_name
```



## entrypoint

指定服务容器启动后执行的命令：

```yaml
entrypoint: /code/entrypoint.sh
```



## user

指定容器中运行应用的应用名：

```yaml
user: nginx
```



## working_dir

指定容器中的工作目录：

```yaml
working_dir: /code
```



## privileged

允许容器中运行一些特权命令：

```yaml
privileged: true
```



## restart

指定容器退出后的重启策略为始终重启。该命令对保持服务始终运行十分有效，在生产环境中推荐配置为always或者unless-stopped:

```yaml
restart: always
```



## depends_on

这个标签解决了容器的依赖、启动先后的问题。



## depends_on 和 links

还记得上面的depends_on吧，那个标签解决的是启动顺序问题，这个标签解决的是容器连接问题，与Docker client的--link一样效果，会连接到其它服务中的容器。

在使用 Compose 时，最大的好处就是少打启动命令，但是一般项目容器启动的顺序是有要求的，如果直接从上到下启动容器，必然会因为容器依赖问题而启动失败。
例如在没启动数据库容器的时候启动了应用容器，这时候应用容器会因为找不到数据库而退出，为了避免这种情况我们需要加入一个标签，就是 depends_on，这个标签解决了容器的依赖、启动先后的问题。

# 读取环境变量

Compose模板文件支持动态读取主机的系统环境变量。

例如，下面的Compose文件将从运行它的环境变量中读取变量`${MONGO_VERSION}`的值，并将其写入执行的命令中：

```yaml
db:
	image: "mongo:${MONGO_VERSION}"
```



如果执行`MONGO_VERSION=3.0 docker-compoe up` 则会启动一个mongo:3.0镜像的容器；如果执行`MONGO_VERSION=2.8 docker-compoe up`则会启动一个mongo:2.8镜像的容器



# docker-compose.yml模板文件

```yaml
version: '2'
services:
  web1:
    image: nginx
    ports: 
      - "8080:80"
    container_name: "web1"
    networks:
      - dev
    volumes:
      - ntfs:/data
  web2:
    image: nginx
    ports: 
      - "8081:80"
    container_name: "web2"
    networks:
      - dev
      - pro
    volumes:
      - ./:/usr/share/nginx/html
  web3:
    image: nginx
    ports: 
      - "8082:80"
    container_name: "web3"
    networks:
      - pro
    volumes:
      - ./:/usr/share/nginx/html

networks:
  dev:
    driver: bridge
  pro:
    driver: bridge

volumes:
  ntfs:
    driver: local
```



## 启动服务

通过执行 docker-compose up -d  启动， 

## 查看服务

 docker-compose ps`查看服务，

## 停止服务

docker-compose stop [name]` 停止服务，`docker-compose start [name]`启动服务，

## 删除服务

`docker-compose rm [name]`删除服务，需要停止服务，否则使用-f参数，与`docker rm`命令类似

如果要删除数据卷、网络、应用则使用`docker-compose down`命令。

## 查看服务日志

`docker-compose logs -f [name]`查看具体服务的日志。

## 进入容器内部

通过`docker-compose exec [name] shell`可以进入容器内部，例如，进入web1容器内部，使用`docker-compose exec web1 /bin/bash`命令。

## 关闭服务

使用`docker-compose down`关闭先前的服务，重新启动服务。