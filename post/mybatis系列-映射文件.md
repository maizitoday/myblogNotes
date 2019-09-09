---
title:       "mybatis系列-映射文件"
subtitle:    ""
description: ""
date:        2019-09-02
author:      "麦子"
image:       "https://c.pxhere.com/images/52/54/18c63f88716ecc0dd8dd729f47a3-1420031.jpg!d"
tags:        ["mybatis系列", "insert操作返回主键", "#和$区别", "一对一模式数据", "一对多模式数据"]
categories:  ["Tech" ]
---

[TOC]



**说明：以下文字总结来源尚硅谷视频《MyBatis》<https://www.bilibili.com/video/av34875242/?p=9>**

**官网文档：<http://www.mybatis.org/mybatis-3/zh/configuration.html#typeAliases>**

# 默认返回 Integer，Long，Boolean类型

对于增加，删除，修改， mybatis框架默认返回上面三种类型， 只要在Mapper中，直接写好返回类型就OK了。 

```java
public interface MysqlStudentMapper {

    boolean insertStudent(Student student);

    int deleteStudent(Student student);
    
    int updateStudent(Student student);
}
```

```xml
<insert id="insertStudent">
          insert into student (id,name,sex,age,address_bus)  values
                               (#{id}, #{name}, #{sex}, #{age}, #{addressBus})
 </insert>

<update id="updateStudent">
     update student set name = #{name} where id = #{id}        
</update>

<delete id="deleteStudent">
       delete from student where id = #{id};
</delete>
```

## 小技巧

我们可以把参数出入给sql执行的数据格式都为Map就好了， 因为框架底层也是用map来封装所有的参数的。 

# insert操作返回主键

##  MySql

```xml
<!--
   useGeneratedKeys: 使用自增主键获取主键策略
   keyProperty:  指定对应的主键属性， 也就是Mybatis获取到主键的值后赋值给哪一个javaBean对象的属性
 -->  
<insert id="insertStudent" 
        parameterType="com.example.mybatisconfig.mysql.bean.Student"
        useGeneratedKeys="true"  
        keyProperty="id">
       insert into student (name,sex,age,address_bus)  values 
                           (#{name}, #{sex}, #{age}, #{addressBus})
    </insert>
```

## Oracle

```xml
<!-- 
keyProperty:  指定对应的主键属性， 查询出队列后，放入到这个javaBean里面的id中
resultType:   查出返回的数据类型， 
     order:   BEFORE  在执行insert语句之前执行这个查询序列的操作。
-->
<insert id="insertStudent">
   <selectKey keyProperty="id" order="BEFORE" resultType="INTEGER">
              select SEQ_NEWSID.nextval from dual
   </selectKey>
   insert into student (id, name,sex,age,addressBus) 
                values (#{id}, #{name}, #{sex}, #{age}, #{addressBus})
</insert>
```

```xml
<!-- 
keyProperty:  指定对应的主键属性， 查询出队列后，放入到这个javaBean里面的id中
resultType:   查出返回的数据类型， 
     order:   AFTER  在执行之后，直接把序列号返回给当前对象的id
-->
<insert id="insertStudent">
  <selectKey keyProperty="id" order="AFTER" resultType="INTEGER">
    select SEQ_NEWSID.currval from dual
  </selectKey>
  insert into student (id, name,sex,age,addressBus)  
                    values (SEQ_NEWSID.nextval, #{name}, #{sex}, #{age}, #{addressBus})
</insert>
```

## 注意

**如果在并发的时候， AFTER获取的总是最后一条的值， 是会有问题的， 一般我们用第一种方式。** 

# 参数处理

## 方法参数作为sql语句参数占位符

框架默认会把所有的参数都放入到**Map中**进行填充到sql语句中。 

```xml
<!-- 允许使用方法签名中的名称作为语句参数名称。 
     为了使用该特性，你的项目必须采用 Java 8 编译.   默认是true， 
     所以在mapper中方法中的参数，在sql的xml中可以直接使用。
 -->
<setting name="useActualParamName" value="true" />
```



## #和$区别

```xml
<select id="findStudentInfoByID" resultType="com.example.mybatisconfig.mysql.bean.Student">
       select * from student where id = ${id} and name = #{name}
</select>
```

```java
: ==>  Preparing: select * from student where id = 1 and name = ? 
: ==>  Parameters: 小强(String)
: <==  Total: 1
```

可以看到$符号直接就把数据给填充进入了， 而#只是一个占位符号，占位符的预编译更加的安全。 大多数的时候我们都是使用#的， 但是我们在分表的时候， 我们需要查询表的时候，表名是无法预编译的，所以只能使用这个美元了。

## 参数属性

```properties
#{height,javaType=double,jdbcType=NUMERIC,numericScale=2}
```

尽管所有这些选项很强大，但大多时候你只须简单地指定属性名，其他的事情 MyBatis 会自己去推断，顶多要为可能为空的列指定 `jdbcType`。

jdbcType特定环境下面设置， 在我们的数据为NULL的时候， 有些数据可能不能识别mybatis对null的默认处理，比如Oracle的报错， 因为Mybatis对所有null映射的事原生Jdbc的OTHER类型， Oracle不能正确处理，他是没有这个类型的。所以这个时候需要加上这个属性。 

```properties
#{height,jdbcType=NULL}
```



# select标签

## 1.  返回一个对象和集合， 都是返回集合中的元素类型

```xml
<select id="findStudentInfoByID" resultType="com.example.mybatisconfig.mysql.bean.Student">
         select * from student where id = ${id} and name = #{name}
</select>
```

## 2.  返回Map对象

```xml
<select id="findStudentInfoByID" resultType="map">
       select * from student where id = ${id} and name = #{name}
</select>
```

## 3.  返回Map<Integer,Student>

```xml
<select id="findStudentInfoByID" resultType="com.example.mybatisconfig.mysql.bean.Student">
    select * from student where id = ${id} and name = #{name}
</select>
```

```java
  @MapKey("id") // 告诉Mybatis以这个值为key，一般都是主键。
  Map<Integer, Student> findStudentInfoByID(Map<String, Object> map);
```

### 显示效果为一个key对应这个对象。 



## 4.  resultMap

```xml
<resultMap id="student" type="com.example.mybatisconfig.mysql.bean.Student">
    <!-- id 定义主键底层有优化 -->
    <id column="id" property="id"/>
    <result column="name" property="name"/>
    <result column="sex" property="sex"/>
    <result column="age" property="age"/>
    <result column="address_bus" property="addressBus"/>
</resultMap>

<select id="findStudentInfoByID" resultMap="student">
		select * from student where id = ${id} and name = #{name}
</select>
```

### 一对一模式数据

```xml
<resultMap id="student" type="com.example.mybatisconfig.mysql.bean.Student">
      <!-- id 定义主键底层有优化 -->
      <id column="id" property="id"/>
      <result column="name" property="name"/>
      <result column="sex" property="sex"/>
      <result column="age" property="age"/>
      <result column="address_bus" property="addressBus"/>
      <!-- 一个学生对应一个爱好-->
      <association property="play" javaType="com.example.mybatisconfig.mysql.bean.Play">
            <id column="id" property="id"/>
            <result column="name" property="name"/>
            <result column="sid" property="sid"/>
      </association>
    </resultMap>
   
    <select id="findStudentInfoByID" resultMap="student">
    select * from student  
             left join  play  on student.pid = play.id 
             where student.id = #{id}  and student.name = #{name} 
    </select>
```

### 一对多模式数据

```xml
<resultMap id="play" type="com.example.mybatisconfig.mysql.bean.Play">
    <id column="id" property="id"/>
    <result column="name" property="name"/>
    <result column="sid" property="sid"/>
    <collection property="studentList" ofType="com.example.mybatisconfig.mysql.bean.Student">
            <id column="id" property="id"/>
            <result column="name" property="name"/>
            <result column="sex" property="sex"/>
            <result column="age" property="age"/>
            <result column="address_bus" property="addressBus"/>
       </collection>
  </resultMap>
```

```java
Play(id=1, name=篮球, sid=1, 
studentList=[Student(id=1, name=篮球, sex=男, age=8, addressBus=湖南-17路, play=null)])
```

## 5.  对于多表查询，可以进行分布查询

相当于，多表查询，可以拆分为几次sql查询，还可以进行开启延迟加载模式，相当于用到的时候才去执行sql语句。 但是感觉这也增加了和数据库进行交互的次数。所以用的比较的少。 

