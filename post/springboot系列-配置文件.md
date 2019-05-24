---
title:       "springboot系列-配置文件"
subtitle:    ""
description: ""
date:        2019-05-24
author:      "麦子"
image:       "https://c.pxhere.com/images/51/0a/8a6110798028de393339f17f9c51-1433591.jpg!d"
tags:        ["springboot系列", "yaml", "配置文件优先级", "配置文件激活方式", "修改默认配置路径"]
categories:  ["Tech" ]
---

[TOC]

**说明： 下列文章主要对尚硅谷《SpringBoot视频教程》的总结，下列文字描述多来源他们的课件。**

**视频地址： https://www.bilibili.com/video/av38657363/?p=9**

# YAML语法



## 基本语法

k:(空格)v：表示一对键值对（空格必须有）；

以**空格**的缩进来控制层级关系；只要是左对齐的一列数据，都是同一个层级的

```yaml
server:
   port: 8081
   path: /hello
```

属性和值也是大小写敏感；



## 值的写法

### 字面量：数字，字符串，布尔）

k: v：字面直接来写；

​		字符串默认不用加上单引号或者双引号；

​		""：双引号；不会转义字符串里面的特殊字符；特殊字符会作为本身想表示的意思

​				name:   "zhangsan \n lisi"：输出；zhangsan 换行  lisi

​		''：单引号；会转义特殊字符，特殊字符最终只是一个普通的字符串数据

​				name:   ‘zhangsan \n lisi’：输出；zhangsan \n  lisi

### 对象、Map（属性和值） 

k: v：在下一行来写对象的属性和值的关系；注意缩进

​		对象还是k: v的方式

```yaml
friends:
		lastName: zhangsan
		age: 20
```

行内写法：

```yaml
friends: {lastName: zhangsan,age: 18}
```



### 数组（List、Set）

用- 值表示数组中的一个元素

```yaml
pets:
 - cat
 - dog
 - pig
```

行内写法

```yaml
pets: [cat,dog,pig]
```



## 配置文件值注入

### 配置文件

```yaml
person:
  last-name: 麦子
  age: 18
  boss: false
  birth: 2017/12/12
  maps: {k1: v1,k2: 12}
  lists:
     - 小七
     - 小强
  dog:
     name: 石头
     age: 2
    
```



### javaBean

```java
/****
 *  将配置文件中配置的每一个属性的值，映射到这个组件中
 *  @ConfigurationProperties：告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定；
                              prefix = "person"：配置文件中哪个下面的所有属性进行一一映射
 *  只有这个组件是容器中的组件，才能容器提供的@ConfigurationProperties功能；
 * 
 *  @Component: 注入到容器中
 * 
 */
@Component
@ConfigurationProperties(prefix = "person")
@Getter
@Setter
@ToString
public class Person {
  private String lastName; // @ConfigurationProperties 在这里last-name就是lastName和@Value不一样
  private int age;   
  private boolean boss;
  private Date birth;
  private Map<String,Object> maps;
  private List<String> lists;
  private Dog dog;
}
```



### 自动提示

我们可以导入配置文件处理器，以后编写配置就有提示了。

```xml
<!--导入配置文件处理器，配置文件进行绑定就会有提示-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-configuration-processor</artifactId>
			<optional>true</optional>
	 </dependency>
```



## @Value和@ConfigurationProperties区别

如果说，我们只是在某个业务逻辑中需要获取一下配置文件中的某项值，使用@Value；他也不支持复杂类型，比如map

如果说，我们专门编写了一个javaBean来和配置文件进行映射，我们就直接使用@ConfigurationProperties；

```java
@Value("${person.last-name}")   #这种形式， person.last-name 这个需要和yml一模一样
private String lastName;
```



### ConfigurationProperties可以添加校验

```java
@Validated
public class Person {
     @Email
     private String lastName;
     private int age;
     private boolean boss;
     private Date birth;
     private Map<String,Object> maps;
     private List<String> lists;
     private Dog dog;
}
```

**以上都是从全局默认配置文件中。**

# 加载指定配置文件

@**PropertySource**：加载指定的配置文件；

 **注意：这里需要是 properties 格式文件， 同时需要设置编码格式，不然中文乱码，**![Xnip2019-05-24_22-06-49](/img/Xnip2019-05-24_22-06-49.png)

# 加载配置文件类

SpringBoot推荐给容器中添加组件的方式；推荐使用全注解的方式

1、配置类**@Configuration**------>Spring配置文件，标明这是一个配置类

2、使用**@Bean**给容器中添加组件

```java
/**
 * @Configuration：指明当前类是一个配置类；就是来替代之前的Spring配置文件
 *
 * 在配置文件中用<bean><bean/>标签添加组件
 *
 */
@Configuration
public class MyAppConfig {

    //将方法的返回值添加到容器中；容器中这个组件默认的id就是方法名
    @Bean
    public HelloService helloService02(){
        System.out.println("配置类@Bean给容器中添加组件了...");
        return new HelloService();
    }
}
```

# 配置文件占位符

## 随机数

```yaml
${random.value}、${random.int}、${random.long}
${random.int(10)}、${random.int[1024,65536]}
```

## 占位符获取之前配置的值

```properties
person.last-name=张三${random.uuid}
person.age=${random.int}
person.birth=2017/12/15
person.boss=false
person.maps.k1=v1
person.maps.k2=14
person.lists=a,b,c
person.dog.name=${person.hello:hello}_dog  #如果person.hello没有值就用默认值
person.dog.age=15
```

```yaml
person:
  last-name: 原始文件
  age: 18
  boss: false
  birth: 2017/12/12
  maps: {k1: v1,k2: 12}
  lists:
     - 小七
     - 小强
  dog:
     name: ${person.last-name} #这里可以直接用上面定义好的。
     age: 2
```

# Profile

## 多Profile文件

我们在主配置文件编写的时候，文件名可以是   application-{profile}.properties/yml

默认使用application.properties的配置；

## yml支持多文档块方式

```yaml

spring:
  profiles:
    active:
    - test
---             # 三个横线表示文档快
spring:
  profiles: dev 
    
person:
  last-name: 开发文件
  lists:
     - 小七
     - 小强
  dog:
     name: ${person.last-name}
     age: 2

---
spring:
  profiles: test
    
person:
  last-name: 测试文件
  lists:
     - 小七002
     - 小强002
  dog:
     name: ${person.last-name}
     age: 4
```



## 激活指定profile

1、在配置文件中指定  spring.profiles.active=dev

2、命令行：

​	java -jar spring-boot-02-config-0.0.1-SNAPSHOT.jar --spring.profiles.active=dev；

​	可以直接在测试的时候，配置传入命令行参数

​        **这个更加的灵活， 配合shell脚本执行**

3、虚拟机参数；

​	-Dspring.profiles.active=dev

# 配置文件加载位置

springboot 启动会扫描以下位置的application.properties或者application.yml文件作为Spring boot的默认配置文件

–file:./config/

–file:./

–classpath:/config/  

–classpath:/

优先级由高到底，高优先级的配置会覆盖低优先级的配置；

SpringBoot会从这四个位置全部加载主配置文件；**互补配置**；

## 改变默认的配置文件位置

我们还可以通过spring.config.location来改变默认的配置文件位置，**注意写到配置文件中是没有用的**

```shell
➜  target git:(master) ✗ java -jar helloword-0.0.1-SNAPSHOT.jar --spring.config.location=/Users/maizi/Desktop/application.yml
```

**这个方式一般在运维的时候使用，比如运维需要修改不同的数据库，修改后，就可以启动写好的shell脚本，这样就和代码层分开了**。

**注意：根据maven项目的定义的规则，外面的 –file:./config/    –file:./  不会打包到jar中，当然你也可以用maven命令，把这两个的文件直接覆盖classpath里面的文件。**

# 外部配置加载顺序

**==SpringBoot也可以从以下位置加载配置； 优先级从高到低；高优先级的配置覆盖低优先级的配置，所有的配置会形成互补配置==**

**1.命令行参数**

所有的配置都可以在命令行上进行指定

**java -jar spring-boot-02-config-02-0.0.1-SNAPSHOT.jar --server.port=8087  --server.context-path=/abc**

多个配置用空格分开； --配置项=值

2.来自java:comp/env的JNDI属性

3.Java系统属性（System.getProperties()）

4.操作系统环境变量

## 由jar包外向jar包内进行寻找

就是和jar包同级目录的配置文件也可以加载。

![Xnip2019-05-24_23-12-10](/img/Xnip2019-05-24_23-12-10.png)

==**优先加载带profile**==

6.jar包外部的application-{profile}.properties或application.yml(带spring.profile)配置文件

7.jar包内部的application-{profile}.properties或application.yml(带spring.profile)配置文件

==**再来加载不带profile**==

8.jar包外部的application.properties或application.yml(不带spring.profile)配置文件

9.jar包内部的application.properties或application.yml(不带spring.profile)配置文件

**注意：好处就是有利于运维根据需要改变配置文件数据，执行外面的配置文件。** 

