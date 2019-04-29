---
title:       "springCloud系列-简介"
subtitle:    ""
description: ""
date:        2019-04-17
author:      "麦子"
image:       "https://zhaohuabing.com/img/2018-12-27-the-obstacles-to-put-istio-into-production/background.jpg"
tags:        ["springCloud", "简介"]
categories:  ["Tech" ]
---

[TOC]



**说明： 本学习主要对尚硅谷周阳大神《SpringCloud视频》记录，一下文字来源这个视频PPT**

**说明： 版本springboot2.14 ** **<spring-cloud.version>Greenwich.SR1</spring-cloud.version>**

# 微服务说明

微服务化的核心就是将传统的一站式应用，根据业务拆分成一个一个的服务，彻底地去耦合,每一个微服务提供单个业务功能的服务，一个服务做一件事，从技术角度看就是一种小而独立的处理过程，类似进程概念，能够自行单独启动或销毁，拥有自己独立的数据库。

# springCloud和dubbo区别

SpringBoot专注于快速、方便的开发单个微服务个体，SpringCloud关注全局的服务治理框架。同时Dubbo是RPC通信，springCloud是通过HTTP协议通信。

![屏幕快照 2019-04-17 下午3.37.40](/img/17-1.png)

# SpringCloud优点和缺点

![屏幕快照 2019-04-15 下午7.22.14](/img/17-3.png)

![屏幕快照 2019-04-15 下午7.24.21](/img/17-4.png)

# 初步流程图

![屏幕快照 2019-04-15 下午7.55.50](/img/17-5.png)

# 搭建传统工程

![屏幕快照 2019-04-17 下午7.16.45](/img/17-2.png)

这是我们传统工程，  Mytools作为公共的部分， consumer项目通过RestTemplate来调用provide提供的接口，我们将在这个传统项目一层一层添加springCloud的相关服务。

# SpringBoot引用自己的parent

 创建Springboot的时候， 默认会引用

```xml
<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.0.3.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
```

但是我们在实际开发工作中，一般都是引用自己的父类，修改如下：

```xml
 <parent>
		<artifactId>mycloud</artifactId>
		<groupId>maizi.cloud</groupId>
		<version>1.0-SNAPSHOT</version>
	</parent>
	 #上面是自己的父类 

   #这个就相当于上面，引用Spring的父类一些东西
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-dependencies</artifactId>
				<version>2.1.4.RELEASE</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
```

**注意：我一开始父类创建的时候选择的是pom类型，不是选择jar， 但是后面在consumer工程引用tools里面的java类时候，一直无法引用成功，后面删除工程，重新创建父类工程为jar类型，引用成功。**