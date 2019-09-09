---
title:       "mybatis系列-mybatis-config.xml全局配置文件设置"
subtitle:    ""
description: ""
date:        2019-09-02
author:      "麦子"
image:       "https://c.pxhere.com/images/52/54/18c63f88716ecc0dd8dd729f47a3-1420031.jpg!d"
tags:        ["mybatis系列", "mybatis-config.xml设置"]
categories:  ["Tech" ]
---

[TOC]

**说明：以下文字总结来源尚硅谷视频《MyBatis》<https://www.bilibili.com/video/av34875242/?p=9>**

**官网文档：<http://www.mybatis.org/mybatis-3/zh/configuration.html#typeAliases>**

# Settings标签

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
 
<configuration>

    <settings>
         <!-- 开启驼峰命名法 -->
        <setting name="mapUnderscoreToCamelCase" value="true" />
        <!-- 允许使用方法签名中的名称作为语句参数名称。 为了使用该特性，你的项目必须采用 Java 8 编译
             默认是true， 所以在mapper中方法中的参数，在sql的xml中可以直接使用。
         -->
        <setting name="useActualParamName" value="true" />
    </settings>
</configuration>
```

# typeAliases标签

```xml
<typeAliases>
      <!-- 
       type: 为某个java类型起别名，默认别名就是类的小写  
       alias: 为这个类取一个别名
       但是感觉这种方式，在实际开发中不利于查看代码
      -->
    <typeAlias type="com.example.mybatisconfig.mysql.Student"  alias="student"/></typeAliases>
```



## 内建的相应的类型别名

这是一些为常见的 **Java 类型内建的相应的类型别名**。它们都是不区分大小写的，注意对基本类型名称重复采取的特殊命名风格。**注意默认的别名都是小写**, 所以我们以后返回的时候**可以不写java.lang.String**

| 别名       | 映射的类型 |
| :--------- | :--------- |
| _byte      | byte       |
| _long      | long       |
| _short     | short      |
| _int       | int        |
| _integer   | int        |
| _double    | double     |
| _float     | float      |
| _boolean   | boolean    |
| string     | String     |
| byte       | Byte       |
| long       | Long       |
| short      | Short      |
| int        | Integer    |
| integer    | Integer    |
| double     | Double     |
| float      | Float      |
| boolean    | Boolean    |
| date       | Date       |
| decimal    | BigDecimal |
| bigdecimal | BigDecimal |
| object     | Object     |
| map        | Map        |
| hashmap    | HashMap    |
| list       | List       |
| arraylist  | ArrayList  |
| collection | Collection |
| iterator   | Iterator   |

# databaseIdProvider(动态切换数据库)

```xml
<!-- 支持多数据厂商，作用就是得到数据库厂商的标识，mybatis就能根据不同的数据库厂商标识执行不同的sql语句-->
<databaseIdProvider type="DB_VENDOR">
        <property name="MySql" value="mysql"/>
        <property name="Oracle" value="oracle" />
 </databaseIdProvider>
```

## xml

```xml
<mapper namespace="com.example.mybatisconfig.mysql.MysqlStudentMapper">

  <select id="getStudentInfoByID" resultType="com.example.mybatisconfig.mysql.bean.Student" databaseId="mysql">
         select sex from student where id = #{id}
  </select>

  <select id="getStudentInfoByID" resultType="com.example.mybatisconfig.mysql.bean.Student" databaseId="oracle">
        select name from student where id = #{id}
  </select>

  <select id="getStudentInfoByID"resultType="com.example.mybatisconfig.mysql.bean.Student">
         select * from student where id = #{id}
   </select>

</mapper>
```

## databaseId  

如果配置了数据库厂商标识（databaseIdProvider），MyBatis 会加载所有的不带 databaseId 或匹配当前 databaseId 的语句；如果带或者不带的语句都有，则不带的会被忽略。

**注意：但是我用了这种方式，在springboot中并没有成功， 报出的错误是，在指定的mysql或者oracle中，找不到这个getStudentInfoByID这个方法，后期在修改测试这个问题。**



# mappers(映射器)

```xml
<mappers>
  <!-- 使用相对于类路径的资源引用 -->
  <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
  <!-- 使用完全限定资源定位符（URL） -->
  <mapper url="file:///var/mappers/AuthorMapper.xml"/>
  <!-- 使用映射器接口实现类的完全限定类名 -->
  <mapper class="org.mybatis.builder.AuthorMapper"/>
</mappers>
```

**注意：class方式的话，有sql映射文件，映射文件名必须和接口同名，并且放在与接口同一目录下**

```xml
<!-- 将包内的映射器接口实现全部注册为映射器 -->
<mappers>
  <package name="org.mybatis.builder"/>
</mappers>
```

**还可以直接注解写sql语句，但是一般不用这种方式开发。** 