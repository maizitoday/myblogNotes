---
title:       "maven多项目依赖"
subtitle:    ""
description: ""
date:        2019-10-21
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["开发工具", "maven"]
categories:  ["Tech" ]
---

[TOC]

# 简述

简单查看项目和项目依赖处理， 业务模块进行modules的分开。抽离公用模块代码。 

# jsite-root

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.org.jsite</groupId>
    <artifactId>jsite-root</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>pom</packaging>

    <modules>
        <module>jsite-common</module>
        <module>jsite-framework</module>
        <module>jsite-flowable</module>
        <module>jsite-web</module>
    </modules>

    <name>JSite</name>
    <description>JSite for Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.4.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
```

# jsite-common

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>cn.org.jsite</groupId>
        <artifactId>jsite-root</artifactId>
        <version>0.0.1-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>

    <artifactId>jsite-common</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>
```

# jsite-framework

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>cn.org.jsite</groupId>
        <artifactId>jsite-root</artifactId>
        <version>0.0.1-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>

    <artifactId>jsite-framework</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>JSite Framework</name>
    <description>JSite Framework for Spring Boot</description>

    <dependencies>

        <dependency>
            <groupId>cn.org.jsite</groupId>
            <artifactId>jsite-common</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
```

# jsite-web

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>cn.org.jsite</groupId>
        <artifactId>jsite-root</artifactId>
        <version>0.0.1-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>

    <artifactId>jsite-web</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>war</packaging>
 

    <dependencies>
        <dependency>
            <groupId>cn.org.jsite</groupId>
            <artifactId>jsite-framework</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>cn.org.jsite</groupId>
            <artifactId>jsite-flowable</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
    </dependencies>
```

