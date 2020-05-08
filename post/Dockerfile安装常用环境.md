---
title:       "Dockerfile安装常用环境"
subtitle:    ""
description: "Nginx,MySql，Oracle，Redis，MongoDB，ElasticSearch,Kibana,Logstash,RockerMQ,Centos7等环境安装"
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



# Centos7

安装Centos7，java，maven，SSH

```dockerfile
# install centos7
FROM centos:7
RUN yum -y install net-tools  

#常用命令下载
RUN yum -y install wget
RUN yum install -y curl


#配置ssh 
RUN yum -y  install  openssh-server
RUN sed -i 's/#PermitRootLogin yes/PermitRootLogin yes/g' /etc/ssh/sshd_config
RUN sed -i 's/#PubkeyAuthentication yes/PubkeyAuthentication yes/g' /etc/ssh/sshd_config
RUN cat /etc/ssh/sshd_config 


#创建一个新用户
RUN useradd  maizissh -d /home/maizissh
RUN mkdir /home/maizissh/.ssh
RUN chown -R maizissh. /home/maizissh/.ssh
COPY id_rsa.pub /usr/local/macrsa/
RUN cd /usr/local/macrsa/ && ls -l 
RUN cat /usr/local/macrsa/id_rsa.pub >> /home/maizissh/.ssh/authorized_keys
RUN cat /home/maizissh/.ssh/authorized_keys
RUN chmod 700 /home/maizissh/.ssh
RUN chmod 600 /home/maizissh/.ssh/authorized_keys


# install java
COPY jdk-8u11-linux-x64.tar.gz /usr/local/java/
COPY start.sh /usr/local/
RUN cd /usr/local/java/ && ls && tar -xvf jdk-8u11-linux-x64.tar.gz && ls
RUN echo 'export JAVA_HOME=/usr/local/java/jdk1.8.0_11' >> /etc/profile
RUN echo 'export CLASSPATH=$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar' >> /etc/profile
RUN echo 'export PATH=$PATH:$JAVA_HOME/bin' >> /etc/profile

RUN echo "查看-----------------"
RUN cat /etc/profile
RUN echo "java success"



# install maven
RUN cd /usr/local/ && mkdir maven
RUN wget http://apache-mirror.rbc.ru/pub/apache/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz
RUN mv apache-maven-3.3.9-bin.tar.gz  /usr/local/maven/  && ls 
RUN cd /usr/local/maven/ && tar -xzvf apache-maven-3.3.9-bin.tar.gz && ls
RUN echo 'export MAVEN_HOME=/usr/local/maven/apache-maven-3.3.9'>>/etc/profile
RUN echo 'export PATH=$JAVA_HOME/bin:$MAVEN_HOME/bin:$PATH'>>/etc/profile

RUN echo "查看-----------------"
RUN cat /etc/profile
RUN echo "maven success"



# COPY java_maven.sh /usr/bin/java_maven.sh
# RUN  chmod +x  /usr/bin/java_maven.sh

#安装java和maven环境
RUN chmod 777 /usr/local/start.sh
ENTRYPOINT [ "sh", "-c", "/usr/local/start.sh" ]
#CMD sh /usr/local/start.sh


# 编译 
# docker build -t mycentos7 .

#启动
#docker run -itd --name mycentos7 -p 6666:22 --privileged=true mycentos7 /usr/sbin/init
 
```

进入容器后，设置root密码

```
passwd  回车
然后设置root密码
```

 修改添加用户的密码

```
passwd   设置root密码
passwd   maizissh  修改用户密码， 这个命令只适合用户。 
```

**注意：这里需要通过 追加到 >> /etc/profile文件中，这样才会永久生效，不然只有这一次连接才有java和maven环境**

## start.sh

```shell

#!/usr/bin/env bash
 
#!/bin/bash

echo "开始执行容器启动脚本"
source /etc/profile
cat /etc/profile

#防止容器启动后退出
tail -f /dev/null
```

## 问题

1. CMD无法执行这个脚本， 还不知道什么原因， 所以运行后，手动执行这个脚本，让 /etc/profile文件生效就好。 

2. 改用ENTRYPOINT命令执行， 可以执行脚本， 但是  source  /etc/profile依然无法运行，所以还是要手动进行执行这个命令，同时怀疑是权限导致，后期设置root权限启动查看是否可以，在进行测试。 

   

# Nginx

```dockerfile
FROM nginx
EXPOSE  8081

# 编译 
# docker build -f /vscode/docker-software-dockerfile/nginx/Dockerfile .


#拷贝容器内 Nginx 默认配置文件到本地当前目录下的 conf 目录
#docker run -itd -p 8081:80 -d nginx
#docker cp fd2fb5879cd5:/etc/nginx/nginx.conf /vscode/docker-software-dockerfile/nginx/conf
#docker cp fd2fb5879cd5:/etc/nginx/conf.d/default.conf /vscode/docker-software-#dockerfile/nginx/conf.d


# 启动
#docker run -itd  --name nginx  -p 8081:80  -v /vscode/docker-software-dockerfile/nginx/conf/nginx.conf:/etc/nginx/nginx.conf  -v /vscode/docker-software-dockerfile/nginx/defaultconf/default.conf:/etc/nginx/conf.d/default.conf  -v /vscode/docker-software-dockerfile/nginx/logs:/var/log/nginx    nginx

# 注意  
# 发现里面的/usr/share/nginx/html/index.html 无法替换。 $PWD如果不行就用绝对路径 -v 


```



# Mysql

```dockerfile
 FROM mysql
 EXPOSE 3306
 
 # 编译 
 # docker build -t mysql-master .

 # 启动
 # docker run -p 3306:3306  --name mysql-master -e MYSQL_ROOT_PASSWORD=root -itd mysql-master
 
```

# Oracle

```dockerfile
 FROM absolutapps/oracle-12c-ee
 EXPOSE 1521 


 # 编译 
 # docker build -t oracle12 .

 
 # 启动
 # docker run -itd --name oracle12  --privileged  -p 8081:8080 -p 1521:1521 absolutapps/oracle-12c-ee

 # 账号: SYSTEM   密码: oracle  SERVICE_NAME:ORCL 
```



# Redis

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

# MongoDB

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

 **集群模式搭建请搜索查看《mongodb-集群和分片》文章，一般我们都是直接集群模式处理**

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



## Kibana

```yml
FROM kibana:6.8.3
EXPOSE  5601
 

# 编译 
docker build -f /vscode/docker-software-dockerfile/kibana/Dockerfile .


# 启动
docker run -d --name kibana6 --net esk -p 5601:5601 kibana:6.8.3

#配置文件地址
/usr/share/elasticsearch/config
```



## Logstash

### dockerfile

```dockerfile
FROM logstash:6.8.3

# 编译 
# docker build -f /vscode/docker-software-dockerfile/Logstash/Dockerfile .

# 启动   
# docker run --name logstash -itd -p 6666:6666 -v  $PWD/config/logstash.yml:/usr/share/logstash/config/logstash.yml -v $PWD/config/pipelines.yml:/usr/share/logstash/config/pipelines.yml -v $PWD/config/logstash-sample.conf:/usr/share/logstash/config/logstash-sample.conf logstash:6.8.3 

```

### logstash.yml

这个我直接给屏蔽了

```yml
# http.host: "0.0.0.0"
# xpack.monitoring.elasticsearch.hosts: [ "http://elasticsearch:9200" ]
```

### pipelines.yml

```yml
- pipeline.id: sample
  path.config: "/usr/share/logstash/config/logstash-sample.conf"
```

### logstash-sample.conf

模拟用tcp插件来获取数据到ES中。 

```json
# input{
#     file{
#         path => ["/rmc.log"]
#         start_position => beginning
#         sincedb_path => "/dev/null"
#     }
# }

input{
  tcp {
        port => "6666"
        mode => "server"
    }
}

filter{


}
  
output {
  elasticsearch {
    hosts => ["http://192.168.1.100:9200"]
    id => "my_plugin_id"
    index => "viewdata-%{+YYYY.MM.dd}"
    #user => "elastic"
    #password => "changeme"
  }
  stdout {
        codec => rubydebug
    }
}
##nc 192.168.1.100 6666 < /var/log/nginx/access.log
用nc命令来模拟tcp数据
```



## $ docker network create somenetwork 

默认网络docker0：网络中所有主机间只能用IP相互访问。通过--link选项创建的容器可以对链接的容器名(container-name)作为hostname进行直接访问。
自定义网络(bridge)：网络中所有主机除ip访问外，还可以直接用容器名(container-name)作为hostname相互访问。

## 问题记录

我的ES和logstash, 一个是Networks：esk  一个是Networks：bridge，但是依然可以OK， 不知道为什么。 我的电脑是mac系统。 后期在来解答。 

# Rocketmq

## 单主机模式

### docker-compose.yml

```json
version: '2'
services:
  namesrv:
    image: rocketmqinc/rocketmq 
    container_name: rmqnamesrv
    ports:
      - 9876:9876 #namesrv只使用了一个9876端口。
    volumes:
      - $PWD/opt/rocketmq/logs:/home/rocketmq/logs
      - $PWD/opt/rocketmq/store:/home/rocketmq/store
    command: sh mqnamesrv
  broker:
    image: rocketmqinc/rocketmq
    container_name: rmqbroker
    ports:
      - 10909:10909 #主要是fastRemotingServer服务使用
      - 10911:10911 #这个主要是broker的服务端口号，作为对producer和consumer使用服务的端口号，默认为10911，可以通过配置文件中修改。
      - 10912:10912 #slave连接使用
    volumes:
      - $PWD/opt/rocketmq/logs:/home/rocketmq/logs
      - $PWD/opt/rocketmq/store:/home/rocketmq/store
      - $PWD/opt/rocketmq/conf/broker.conf:/opt/rocketmq-4.4.0/conf/broker.conf
    #command: sh mqbroker -n namesrv:9876
    command: sh mqbroker -n namesrv:9876 -c ../conf/broker.conf
    depends_on:
      - namesrv
    environment:
      - JAVA_HOME=/usr/lib/jvm/jre
  console:
    image: styletang/rocketmq-console-ng
    container_name: rocketmq-console-ng
    ports:
      - 8087:8080
    depends_on:
      - namesrv
    environment:
      - JAVA_OPTS= -Dlogging.level.root=info   -Drocketmq.namesrv.addr=rmqnamesrv:9876 
      - Dcom.rocketmq.sendMessageWithVIPChannel=falseclear
```

### broker.conf

```properties
# 所属集群名字
brokerClusterName = DefaultCluster
# broker 名字，注意此处不同的配置文件填写的不一样，如果在 broker-a.properties 使用: broker-a,
# 在 broker-b.properties 使用: broker-b
brokerName = broker-a
# 0 表示 Master，> 0 表示 Slave
brokerId = 0
# 删除文件时间点，默认凌晨4点
deleteWhen = 04
# 文件保留时间，默认48小时
fileReservedTime = 48
# Broker 的角色
# - ASYNC_MASTER 异步复制Master
# - SYNC_MASTER 同步双写Master
# - SLAVE
brokerRole = ASYNC_MASTER
# 刷盘方式
# - ASYNC_FLUSH 异步刷盘
# - SYNC_FLUSH 同步刷盘
flushDiskType = ASYNC_FLUSH
#set `brokerIP1` if you want to set physical IP as broker IP.
brokerIP1=这样ip是你映射到主机的IP地址， 但是一定要放开上面的端口
```

### 启动

```yaml
docker-compose up -d

#在broker中，执行，
sh mqadmin clusterList -n rmqnamesrv:9876 

#出现如下数据，证明连接nameserver已经正常。
OpenJDK 64-Bit Server VM warning: ignoring option MaxPermSize=128m; support was removed in 8.0
#Cluster Name     #Broker Name            #BID  #Addr                  #Version                #InTPS(LOAD)       #OutTPS(LOAD) #PCWait(ms) #Hour #SPACE
DefaultCluster    broker-a                0     172.20.0.4 :10911      V4_4_0                   0.00(0,0ms)         0.00(0,0ms)          0 436044.47 -1.0000
```









