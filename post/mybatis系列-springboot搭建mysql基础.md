---
title:       "mybatis系列-springboot搭建mysql，oracle"
subtitle:    ""
description: ""
date:        2019-09-02
author:      "麦子"
image:       "https://c.pxhere.com/images/52/54/18c63f88716ecc0dd8dd729f47a3-1420031.jpg!d"
tags:        ["mybatis系列", "springboot搭建mysql，oracle", "springboot系列"]
categories:  ["Tech" ]
---

[TOC]

**mybatis官方文档：<http://www.mybatis.org/mybatis-3/zh/configuration.html#settings>**

# pom.xml

```xml
    <dependency>
      <groupId>org.mybatis.spring.boot</groupId>
      <artifactId>mybatis-spring-boot-starter</artifactId>
      <version>2.1.0</version>
      <scope>compile</scope>
    </dependency>

    <!--mysql-->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <scope>runtime</scope>
    </dependency>
    <!--oracle-->
    <dependency>
			<groupId>com.oracle</groupId>
			<artifactId>ojdbc6</artifactId>
			<version>11.2.0.4.0</version>
		</dependency>

    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid-spring-boot-starter</artifactId>
      <version>1.1.17</version>
    </dependency>
```



# application.yml

```yaml
server:
  port: 8088

#mysql
spring:
  datasource:
    username: root
    password: root
    url: jdbc:mysql://localhost:32769/school?useUnicode=true&characterEncoding=utf-8&noDatetimeStringSync=true
    driver-class-name: com.mysql.cj.jdbc.Driver  #最新的驱动用这个了
    sql-script-encoding: utf-8
    type: com.alibaba.druid.pool.DruidDataSource   
    
#oracle  
spring:
  datasource:
    username: SYSTEM
    password: oracle
    url: jdbc:oracle:thin:@localhost:1521:ORCL
    driver-class-name: oracle.jdbc.driver.OracleDriver
    sql-script-encoding: utf-8
    type: com.alibaba.druid.pool.DruidDataSource   
    
    

#配置mybatis基础设置
mybatis:
  config-location: classpath:mybatis/mybatis-config.xml
  mapper-locations:
  - classpath:mybatis/mapper/*.xml


#控制台显示sql日志, 对应mapp的包路径
logging:
  level:
    com: 
      example: 
        mybatisconfig:
          mysql: debug
```



# mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">

 
<configuration>
    <settings>
        <!--实体和数据库支持驼峰命名的映射-->
        <setting name="mapUnderscoreToCamelCase" value="true" />
    </settings>

</configuration>
```

## java实体

```java
import lombok.Data;

@Data
public class Student {
    private int id;
    private String name;
    private String sex;
    private int  age;
    private String addressBus;
}
```

## 数据库字段

```sql

CREATE TABLE `student` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(25) DEFAULT NULL,
  `sex` varchar(25) DEFAULT NULL,
  `age` int(200) DEFAULT NULL,
  `address_bus` varchar(255) NOT NULL,
  PRIMARY KEY (`id`),
  KEY `age_index` (`age`)
) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci


```

## 注意

可以看到 实体的 **addressBus** 和数据库的 **address_bus**， 这种对应进行映射的时候，需要开启驼峰命名的方式。 



# 开启扫描Mapper的注入

```java
@SpringBootApplication
@MapperScan(value = "com.example.mybatisconfig.mysql") // 扫描你mapper所在的文件夹
public class DemoApplication {
	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}
}
```



# 项目目录结构

![Xnip2019-09-02_16-13-15](/img/Xnip2019-09-02_16-13-15.png)

