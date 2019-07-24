---
title:       "使用Maven来构建Docker镜像并上传远程服务器"
subtitle:    ""
description: ""
date:        2019-07-21
author:      "麦子"
image:       "https://c.pxhere.com/images/aa/a6/2c425d9c6f81e52b34d87e5c3989-1424141.jpg!d"
tags:        ["docker", "maven", "Maven来构建Docker镜像", "docker开启远程服务", "vscode远程连接docker"]
categories:  ["Tech" ]
---

# 目的

本地通过maven命令， 进行打包， 把项目打包成镜像，然后上传到Linux上的docker上面，作为镜像。

# pom.xml

加载docker-maven-plugin插件

```xml
<plugin>
				<groupId>com.spotify</groupId>
				<artifactId>docker-maven-plugin</artifactId>
				<version>0.4.13</version>
				<executions>
					<execution>
						<id>build-image</id>
						<phase>package</phase>
						<goals>
							<goal>build</goal>
						</goals>
					</execution>
				</executions>
				<configuration>
					<!-- 远程docker服务地址 -->
					<dockerHost>http://10.211.55.8:2375</dockerHost>
					<imageName>${docker.hub.domain}/${docker.image.prefix}/${project.artifactId}:${docker.image.tag}</imageName>
					<forceTags>true</forceTags>
          <!-- Dockerfile 文件位置-->
					<dockerDirectory>${project.basedir}/src/main/docker</dockerDirectory>
					<resources>
						<resource>
							<targetPath>/</targetPath>
                <!-- maven 编译后，会在target下面生成 Dockerfile 和 jar文件-->
							<directory>${project.build.directory}</directory>
							<include>${project.build.finalName}.jar</include>
						</resource>
					</resources>	
				</configuration>
			</plugin>
```

## 将插件绑定在某个phase执行

很多场景下，我们有这样的需求，执行例如`mvn clean package` 时，插件就自动为我们构建Docker镜像。要想实现这点，我们只需将插件的goal绑定在某个phase即可。

phase和goal可以这样理解：maven命令格式是：`mvn phase:goal` ，例如`mvn package docker:build` 。那么，package和docker都是phase，build则是goal 。

```xml
<executions>
  <execution>
    <id>build-image</id>
    <phase>package</phase>
    <goals>
      <goal>build</goal>
    </goals>
  </execution>
</executions>
```

就可将插件绑定在package这个phase上。也就是说，用户只需执行`mvn package` ，就会自动执行`mvn docker:build` 。当然，读者也可按照需求，将插件绑定到其他的phase。

**相关属性标签的解释：https://yq.aliyun.com/articles/622330**

# Dockerfile，sh 文件

```dockerfile
FROM openjdk:8-jre
RUN  mkdir maizi
COPY *.jar /maizi/app.jar
COPY start.sh /maizi/
RUN  cd /maizi/ && chmod a+x start.sh 
CMD [ "sh","/maizi/start.sh" ]
EXPOSE 8888
```



```shell
#!/bin/sh
apt-get update && apt-get install -y procps
cd /maizi/ && touch springboot.log
nohup java -Djava.security.egd=file:/dev/./urandom -jar  app.jar >>springboot.log 2>&1
cat springboot.log
```

# 配置远端docker开启远程访问

**原文地址**：**https://blog.csdn.net/qq_19674905/article/details/80267821**

## 配置远程访问

打开/usr/lib/systemd/system/docker.service如下：

```properties
Service]
Type=notify
NotifyAccess=main
EnvironmentFile=-/run/containers/registries.conf
EnvironmentFile=-/etc/sysconfig/docker
EnvironmentFile=-/etc/sysconfig/docker-storage
EnvironmentFile=-/etc/sysconfig/docker-network
Environment=GOTRACEBACK=crash
Environment=DOCKER_HTTP_HOST_COMPAT=1
Environment=PATH=/usr/libexec/docker:/usr/bin:/usr/sbin
ExecStart=/usr/bin/dockerd-current -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock \
          --add-runtime docker-runc=/usr/libexec/docker/docker-runc-current \
          --default-runtime=docker-runc \
          --exec-opt native.cgroupdriver=systemd \
          --userland-proxy-path=/usr/libexec/docker/docker-proxy-current \
          --init-path=/usr/libexec/docker/docker-init-current \
          --seccomp-profile=/etc/docker/seccomp.json \
          $OPTIONS \
          $DOCKER_STORAGE_OPTIONS \
          $DOCKER_NETWORK_OPTIONS \
          $ADD_REGISTRY \
          $BLOCK_REGISTRY \
          $INSECURE_REGISTRY \
	  $REGISTRIES 
ExecReload=/bin/kill -s HUP $MAINPID
LimitNOFILE=1048576
LimitNPROC=1048576
LimitCORE=infinity
TimeoutStartSec=0
Restart=on-abnormal
KillMode=process

[Install]
WantedBy=multi-user.target

```

 **-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock** , 这个地方就是配置开启远程访问的端口和服务。



## docker重新读取配置文件，重新启动docker服务

```linux
systemctl daemon-reload
systemctl restart docker
```



## 验证docker进程和远程端口

```linux
ps -elf | grep docker

[root@localhost docker]# netstat -tulp  | grep  2375
tcp6       0      0 [::]:2375               [::]:*                  LISTEN      5741/dockerd-curren 

```



## 测试访问centos7的docker服务

在本地用docker 命令查看远程的docker服务镜像，用于测试远程是否OK。

```linux
➜  blog docker -H 10.211.55.8  images
REPOSITORY                            TAG                 IMAGE ID            CREATED             SIZE
maizi1989/maizi-today/maventodocker   20190718            6d4e4c41bc92        2 days ago          283MB
openjdk                               8-jre               f44b742ffc37        3 days ago          246MB
```

从这里可以看到， 本地已经可以查看到了远程docker的所有镜像了，证明远程服务已经开启。 

# maven 打包上传

vscode手动进行clean ，package打包或者在项目目录下进行如下命令进行打包：

```maven
  mvn clean package 
```

操作时候查看远端docker， 进行docker imgaes就可以查看到刚上传过来的镜像。 

# Docker报错 WARNING: IPv4 

**原文：https://blog.csdn.net/yjk13703623757/article/details/68939183**

启动容器

```dockerfile
docker run --rm -it -p 8888:8888/tcp maizi1989/maizi-today/maventodocker:20190718
```

如果直接进行启动，创建容器的时候， 报如下错误：

```linux
WARNING: IPv4 forwarding is disabled. Networking will not work.
```



## 解决办法

```
vim  /usr/lib/sysctl.d/00-system.conf
net.ipv4.ip_forward=1
systemctl restart network #重启network服务
```

完成以后，删除错误的容器，再次创建新容器，就不再报错了。

## 注意

这个问题一定要解决， 不然你在本地无法访问远端的docker启动的服务。还有导致你无法通过客户端去连接远端docker服务。 

# vscode工具连接远程docker服务

可以看到我们把镜像发布到远端docker的时候， 操作docker服务的时候，不方便， 所以找一个客户端直接远程连接docker服务， 方便操作。

点击vscode的 [设置],然后搜索 docker:host ， 然后配置如下：

```linux
tcp://10.211.55.8:2375

注意：这个地方不要写http, 如果是写http的话，我用vscode工具，进行run的时候， 他的dockerfile里面的端口无法映射到机器端口。
```

这样， 就可以远程操作docker服务了，这个插件有很多docker的操作， 很方便。效果如下：

 ![Xnip2019-07-21_15-13-53](/img/Xnip2019-07-21_15-13-53.png)

 可以看到这里可以配置选择工作区， 如果默认不配置的话，就显示你本机的docker服务。