---
title:       "mybatis系列-动态sql"
subtitle:    ""
description: ""
date:        2019-09-03
author:      "麦子"
image:       "https://c.pxhere.com/images/52/54/18c63f88716ecc0dd8dd729f47a3-1420031.jpg!d"
tags:        ["mybatis系列", "动态sql", "OGNL表达式"]
categories:  ["Tech" ]
---

[TOC]



**说明：以下文字总结来源尚硅谷视频《MyBatis》<https://www.bilibili.com/video/av34875242/?p=9>**

**官网文档：<http://www.mybatis.org/mybatis-3/zh/configuration.html#typeAliases>**

**OGNL文档：<https://commons.apache.org/proper/commons-ognl/language-guide.html>**

# if-标签

## 讨论 null  和 '' 是否有区别

```xml
<select id="findStudentInfoByID" resultMap="student">
      select * from student  
      <if test="id != '' ">
             where student.id = #{id} 
      </if>          
 </select>
: ==>  Preparing: select * from student where student.id = ? 
: ==> Parameters: null
: <==      Total: 0
```

```xml
<select id="findStudentInfoByID" resultMap="student">
    select * from student  
    <if test="id != null ">
           where student.id = #{id} 
    </if>
</select>
: ==>  Preparing: select * from student 
: ==> Parameters: 
: <==      Total: 9
```

对比发现在这个里面  null  和 '' 是有区别的。 但是他也提供了去除空格的方法。 

```
<select id="findStudentInfoByID" resultMap="student">
    select * from student  
        <if test="name.trim() != '' ">
                 where student.id = #{id} 
        </if>
</select>
```

**注意：这里也要保证name值不能为null，不然也就是Cause: java.lang.NullPointerException: target is null for method trim**

# where-标签

## 问题

```xml
<!-- 多出的and问题  -->
<select id="findStudentInfoByID" resultMap="student">
  select * from student  
                where 
                <if test="id != null ">
                  student.id = #{id} 
                </if>
                <if test="name != null ">
                  and student.name = #{name} 
                </if>
</select>
```

## 第一种解决方法

```xml
<select id="findStudentInfoByID" resultMap="student">
  select * from student  
                where  1=1
                <if test="id != null ">
                   and student.id = #{id} 
                </if>
                <if test="name != null ">
                   and student.name = #{name} 
                </if>
</select>
```

## 第二种解决办法

```xml
<select id="findStudentInfoByID" resultMap="student">
    select * from student  
                  <where>
                        <if test="id != null ">
                            and student.id = #{id} 
                        </if>
                        <if test="name != null ">
                            and student.name = #{name} 
                        </if>
                  </where> 
</select>
```

当我们的id是null，name不为null的时候打印sql如下：

```sql
select * from student WHERE student.name = ? 
```

## 注意

他只会去掉掉**前面**的and 和 or

# trim-标签

## 问题

```xml
<!-- 多出的and问题  -->
<select id="findStudentInfoByID" resultMap="student">
        select * from student  
            <where>
                <if test="id != null ">
                      student.id = #{id} and
                </if>
                <if test="name != null ">
                      student.name = #{name} 
                </if>
            </where> 
</select>
```

## 解决办法

```xml
<!-- 
        prefix : 给字符串加一个前缀
        prefixOverrides : 前缀覆盖，干掉前面多余的字符串
        suffix：  给字符串后缀加一个字符串
        suffixOverrides：后缀覆盖，干掉后缀多余的字符串
 -->
<select id="findStudentInfoByID" resultMap="student">
    select * from student  
    <trim prefix="where" suffixOverrides="and">
        <if test="id != null ">
              student.id = #{id} and
        </if>
        <if test="name != null ">
              student.name = #{name} 
        </if>
    </trim>
</select>
```

# choose-标签

```xml
<select id="findStudentInfoByID" resultMap="student">
      select * from student  
            <where>
                <choose>
                    <when test="id != null">
                          student.id = #{id} 
                    </when>
                    <when test="name != null">
                          student.name = #{name} 
                    </when>
                    <otherwise>
                           1 = 1
                    </otherwise>
                </choose>
            </where>
</select>
```

# set-标签

## 问题

```xml
<!-- 多出的, 问题  -->
<update id="updateStudent">
      update student set 
          <if test="name != null">
               name = #{name},
          </if>
          <if test="age != 0">
              age = #{age},
          </if>
          <if test="sex != null">
              sex = #{sex}
          </if>
      where id = #{id}        
</update>
```

## 解决办法

```xml
<update id="updateStudent">
      update student 
        <set> 
            <if test="name != null">
                  name = #{name},
            </if>
            <if test="age != 0">
                  age = #{age},
            </if>
            <if test="sex != null">
                  sex = #{sex}
            </if>
        </set>        
      where id = #{id}   
</update>
```



# foreach-标签

```xml
<select id="findStudentInfoByID" resultMap="student">
    select * from student  where id in (1,2,3)
</select>

<!-- 
  collection:指定要遍历的集合
  item：将当前变量出来集合的元素赋值给指定的变量
  separator：买个元素之间的分隔符
  open：遍历出所有结果的第一个字符
  close：变量出所有结果最后的一个字符
  index: 遍历的是list的时候这个是索引。
  遍历map的时候，index表示map的key， item表示map的val。
-->
<select id="findStudentInfoByID" resultMap="student">
  select * from student  where id in 
                                  <foreach collection="selectparamMap" 
                                           item="paramMapVal" separator="," 
                                           open="(" close=")" index="i">
                                           
                                     #{paramMapVal}
                                     
                                  </foreach>
</select>
```

## 批量保存

### mysql

```xml
<insert id="insertStudentList">
  insert into student (name,sex,age,address_bus)  values 
      <foreach collection="studentlist" item="student" separator="," index="i">
          (#{student.name}, #{student.sex}, #{student.age}, #{student.addressBus})
      </foreach>
</insert>
```

### Oracle

```xml
<insert id="insertStudentList">
  begin
      <foreach collection="studentlist" item="student">
      INSERT INTO SYSTEM.STUDENT (ID, NAME, SEX, AGE, ADDRESSBUS) 
                                VALUES 
                                 (SEQ_NEWSID.nextval, #{student.name}, 
                                 #{student.sex}, #{student.age}, 
                                 #{student.addressBus});
      </foreach>
  end;
</insert>
```

#### 注意

Oracle中后面的end的分号不能丢失，不然执行不成功。 



# 两个内置参数

## _parameter

代表整个参数。

单个参数： _parameter就是这个参数

多个参数： 参数会被封装为一个map，他就代表这个map



## _databaseId

如果配置了databaseIdProvider标签，他就代表当前的数据库的别名。



## 实际用途演示

可以处理不同数据库的语句放到一起。 

```xml
<select id="XXXX" resultMap="XXXX">
    <if test="_databaseId == 'mysql'" >
        select * from a
          <if test="_parameter != null">
             where name = #{name}
    			</if>
    </if>    
    <if test="_databaseId == 'oracle'" >
        select * from b
    </if>    
</select>
```



# bind-标签

可以将OGNL表达式的值绑定到一个变量中，方便后来引用这个变量的值。 可以看到他可以数据进行处理加工，同时，OGNL表达式是可以调用java方法的，这样的话对数据的处理也可以有这个方法。 

```xml
<insert id="insertStudent" 
        parameterType="com.example.mybatisconfig.mysql.bean.Student" 
        useGeneratedKeys="true" keyProperty="id">
        <bind name="_buildName" value=" '110_' + name + '_119' " />
        insert into student (name,sex,age,address_bus)  
                            values
                            (#{_buildName}, #{sex}, #{age}, #{addressBus})
</insert>
```



# sql，include 标签

sql用于抽取可重用的sql标签。 include引用外部定义的sql片段。 

```xml
<insert id="insertStudent" 
        parameterType="com.example.mybatisconfig.mysql.bean.Student">
        <bind name="_buildName" value=" '110_' + name + '_119' " />
        insert into student (<include refid="student_sql"></include>)  
                     values (#{_buildName}, #{sex}, #{age}, #{addressBus})
</insert>

<sql id="student_sql">
      name,sex,age,address_bus
</sql>

```

注意：在sql片段中可以也可以加入<if>等标签。 