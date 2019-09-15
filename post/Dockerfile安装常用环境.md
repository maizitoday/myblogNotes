---
title:       "Dockerfile安装常用环境"
subtitle:    ""
description: ""
date:        2019-08-21
author:      "麦子"
image:       "https://c.pxhere.com/photos/50/b0/tree_road_archway_symmetry_and_black_white-99031.jpg!d"
tags:        ["开发工具", "Dockerfile安装常用环境"]
categories:  ["Tech" ]
---

[TOC]



# 修改容器，镜像名称

```yaml
# 修改容器名称
docker rename 0cf82999886a redis-5.0.5 

# 修改镜像名称
docker tag  XX镜像id   XX新的名字

```

# mysql

```dockerfile
 FROM mysql
 EXPOSE 3306
 
 # 编译 
 # docker build -t mysql-master .

 # 启动
 # docker run -p 3306:3306  --name mysql-master -e MYSQL_ROOT_PASSWORD=root -itd mysql-master
 
```

# oracle

```dockerfile
 FROM absolutapps/oracle-12c-ee
 EXPOSE 1521 


 # 编译 
 # docker build -t oracle12 .

 
 # 启动
 # docker run -itd --name oracle12  --privileged  -p 8081:8080 -p 1521:1521 absolutapps/oracle-12c-ee

 # 账号: SYSTEM   密码: oracle  SERVICE_NAME:ORCL 
```



# redis

<https://hub.docker.com/_/redis>

```shell
# 安装客户端
brew cask install another-redis-desktop-manager
```



```yml
 FROM redis
 EXPOSE  6379
 

 # 编译 
 # docker build -f /vscode/docker-software-dockerfile/redis/Dockerfile .

 
 # 启动
 docker run -p 6379:6379 \
          -v /vscode/docker-software-dockerfile/redis/6379/data:/data \
          -v /vscode/docker-software-dockerfile/redis/6379/redis.conf:/etc/redis/redis.conf \
            --privileged=true \
            --name redis \
            -itd redis  redis-server /etc/redis/redis.conf --appendonly yes
```

## redis.conf

```shell
#获取原始的redis.conf
wget http://download.redis.io/releases/redis-5.0.5.tar.gz

#新版本可能需要修改为 
bind 0.0.0.0

#不想给redis配置密码，需要修改配置文件：
protected-mode no

```

上面处理完成，本地客户端和docker容器可以进行连接了。 

## 服务启动和停止

```yml
apt-get update && apt-get install procps 

ps aux | grep redis

#关闭 
cd  /usr/local/bin 
root@369a39fe77bf:/usr/local/bin# redis-cli -h 127.0.0.1 -p 6379 shutdown

#启动
cd  /usr/local/bin 
root@369a39fe77bf:/usr/local/bin# redis-server /etc/redis/redis.conf
```

# mongodb

```yml
FROM mongo
EXPOSE 27017
 
#docker hub -- https://hub.docker.com/_/mongo 

#编译 
docker build -f /vscode/docker-software-dockerfile/mongodb/Dockerfile .

#启动
docker run -p 27017:27017 -v $PWD/db:/data/db -v $PWD/configdb:/data/configdb -d mongo

#用自己的配置文件进行启动
#docker run --name some-mongo -v /my/custom:/etc/mongo -d mongo --config /etc/mongo/mongod.conf
```

