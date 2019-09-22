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

#命令地址
#/usr/bin/mongo  #手动启动客户端地址

 
#/usr/bin/  ./mongofiles -d records put /macfile/110.pdf  #GridFS操作


#安装自己配置文件启动
docker run -p 27000:27000 \
              -v $PWD/27000/db:/data/db \
              -v $PWD/27000/configdb:/data/configdb \
              -itd mongo4.2.0 \
              --config /data/configdb/27000_mongod.conf

```

## 

# ELK

## ElasticSearch

```yml
FROM elasticsearch:6.8.3
EXPOSE  9300
EXPOSE  9200

# 编译 
docker build -f /vscode/docker-software-dockerfile/ElasticSearch/Dockerfile .

# 创建网段 
docker network create esk

# macOS with Docker for Mac  设置vm
$ screen ~/Library/Containers/com.docker.docker/Data/vms/0/tty
sysctl -w vm.max_map_count=262144

# 启动
docker run -itd --name elasticsearch --net esk -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:6.8.3

# 注意
这个 --name 名字不能修改， 因为kibana默认的配置文件 kibana.yml 就是这个 elasticsearch.hosts: [ "http://elasticsearch:9200" ]

# docker
9300端口是使用tcp客户端连接使用的端口；
9200端口是通过http协议连接es使用的端口；

#单节点发现编辑  discovery.type 
我们认识到某些用户需要将传输绑定到外部接口以测试其对传输客户端的使用。对于这种情况，我们提供发现类型single-node（通过设置discovery.type为来 配置single-node）；在这种情况下，节点将选举自己为主节点，并且不会与任何其他节点一起加入群集。

#配置文件地址
/usr/share/elasticsearch/config

```



## kibana

```yml
FROM kibana:6.8.3
EXPOSE  5601
 

# 编译 
docker build -f /vscode/docker-software-dockerfile/kibana/Dockerfile .


# 启动
docker run -d --name kibana6 --net esk -p 5601:5601 kibana:6.8.3

#配置文件地址
/usr/share/kibana/config
```



## $ docker network create somenetwork 

默认网络docker0：网络中所有主机间只能用IP相互访问。通过--link选项创建的容器可以对链接的容器名(container-name)作为hostname进行直接访问。
自定义网络(bridge)：网络中所有主机除ip访问外，还可以直接用容器名(container-name)作为hostname相互访问。