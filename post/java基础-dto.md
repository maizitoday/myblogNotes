---
title:       "dto和Entity转换"
subtitle:    ""
description: ""
date:        2019-06-29
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["java基础", "dto和Entity转换"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：<https://blog.csdn.net/liuweixiao520/article/details/78174368>**

# pom.xml

```xml
<dependency>
			<groupId>net.sf.dozer</groupId>
			<artifactId>dozer</artifactId>
			<version>5.4.0</version>
</dependency>
```

如果使用的事springboot集成的**spring-boot-starter-dozer**，需要去加载固定的xml。这里demo就使用上面的，不使用集成的。

# DozerBeanMapperConfigure

注入bean

```java
@Configuration
public class DozerBeanMapperConfigure {
    @Bean
    public DozerBeanMapper mapper() {
        // 这里没有加载"classpath:dozer-mappings/*.xml"
        DozerBeanMapper mapper = new DozerBeanMapper(); 
        return mapper;
    }
}
```

# SourceBean

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@ToString
public class SourceBean {
    @Mapping("pk")
    private int id;

    private String name;

    @Mapping("binaryData")
    private String data;
}
```

# TargetBean

```java
@Data
@ToString
@AllArgsConstructor
@NoArgsConstructor
public class TargetBean {
    private String pk;

    private String name;

    private String binaryData;
}
```



# DtoController

```java
@RestController
public class DtoController {

    @Autowired
    private Mapper mapper;
 
    @GetMapping(value="dto")
    public void dtoHanler() {
        TargetBean targetBean = new TargetBean();
		SourceBean sourceBean = new SourceBean(1, "小强", "1989");
		mapper.map(sourceBean, targetBean);
        System.out.println(targetBean);
        System.out.println(sourceBean);
    }

}
```

# 运行结果

```java
TargetBean(pk=1, name=小强, binaryData=1989)
SourceBean(id=1, name=小强, data=1989)
```



# 复杂用xml方式

此种方式最简便，但是官方中只有一个注释@Mapping，因此只能应对简便的映射关系，意思是如果想要使用更复杂的映射，就只能使用xml配置，或者API方式，**官方极力推荐使用XML方式**

```java
@Configuration
public class DozerBeanMapperConfigure {

    @Bean
    public DozerBeanMapper mapper() {
        DozerBeanMapper mapper = new DozerBeanMapper();
        mapper.setMappingFiles(Arrays.asList("mapping/UserMapping.xml"));
        return mapper;
    }

}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<mappings xmlns="http://dozer.sourceforge.net"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://dozer.sourceforge.net
          http://dozer.sourceforge.net/schema/beanmapping.xsd">
    <mapping>
        <class-a map-null="false">com.example.springboot.Entity.User</class-a>
        <class-b map-null="false">com.example.springboot.EntityVo.UserVo</class-b>
        <field>
            <a>teacher.name</a>
            <b>teacherName</b>
        </field>
    </mapping>
 </mappings>
```

# 常用开发

我们在mybatis中的时候， 可以用DTO的这个类直接继承数据库的对应的实体类， 然后如果有需要多的字段或者是需要改变字段来对应前端的话，我们就可以使用@Mapper这个注解来对应前台显示字段，同时又屏蔽了数据库结构。