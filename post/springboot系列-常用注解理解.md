---
title:       "springboot系列-常用注解理解"
subtitle:    ""
description: "@Configuration,@bean,@Component,@ComponentScan,@Scope,@Autowired,@Resource,spring的容器理解"
date:        2019-06-11
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["springboot系列", "常用注解理解"]
categories:  ["Tech" ]
---

[TOC]

# @Configuration

**转载地址： https://www.cnblogs.com/duanxz/p/7493276.html**

@Configuration用于定义配置类，可替换xml配置文件，**被注解的类内部包含有一个或多个被@Bean注解的方法，这些方法将会被AnnotationConfigApplicationContext或AnnotationConfigWebApplicationContext类进行扫描，并用于构建bean定义，初始化Spring容器。**

相当于以前定义beans文件，我们以前一般都是在这里定义所有需要的容器加载后，就可以用的bean对象。比如数据库等这样的操作。 **spring容器启动就可以用的java实例对象**。 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context" xmlns:jdbc="http://www.springframework.org/schema/jdbc"  
    xmlns:jee="http://www.springframework.org/schema/jee" xmlns:tx="http://www.springframework.org/schema/tx"
    xmlns:util="http://www.springframework.org/schema/util" xmlns:task="http://www.springframework.org/schema/task" xsi:schemaLocation="
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
        http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc-4.0.xsd
        http://www.springframework.org/schema/jee http://www.springframework.org/schema/jee/spring-jee-4.0.xsd
        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
        http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.0.xsd
        http://www.springframework.org/schema/task http://www.springframework.org/schema/task/spring-task-4.0.xsd" default-lazy-init="false">


</beans>
```

 @Configuation等价于<Beans></Beans>

实例运行效果如下：

![Xnip2019-06-11_17-09-48](/img/Xnip2019-06-11_17-09-48.png)

@Configuration类里面一般都有@bena注解，这些bean就是直接被实例化，spring容器进行托管处理。 **并且保证对象在容器化中只实例化一次。https://www.cnblogs.com/nihaofenghao/p/12612437.html** 

# @bean 

**转载地址： https://www.cnblogs.com/feiyu127/p/7700090.html**

@Bean是一个方法级别上的注解，主要用在@Configuration注解的类里，也可以用在@Component注解的类里。**添加的bean的id为方法名**，相当于以前的下面这种情况。 @Bean却只能在配置类中默认是**单例**。

```xml
<beans>
    <bean id="transferService" class="com.acme.TransferServiceImpl"/>
</beans>
```

 @Bean等价于<Bean></Bean>

## 指定bean的scope

```java
@Configuration
public class MyConfiguration {

    @Bean
    @Scope("prototype")
    public Encryptor encryptor() {
        // ...
    }

}
```

## 自定义bean的命名

```java
@Configuration
public class AppConfig {

    @Bean(name = "myFoo")
    public Foo foo() {
        return new Foo();
    }

}
```

## bean的别名

```java
@Configuration
public class AppConfig {

    @Bean(name = { "dataSource", "subsystemA-dataSource", "subsystemB-dataSource" })
    public DataSource dataSource() {
        // instantiate, configure and return DataSource bean...
    }

}
```

## bean的描述

```java
@Configuration
public class AppConfig {

    @Bean
    @Description("Provides a basic example of a bean")
    public Foo foo() {
        return new Foo();
    }

}
```

## 实例效果如下

![Xnip2019-06-11_17-33-03](/img/Xnip2019-06-11_17-33-03.png)

## @Bean(initMethod = "init",destroyMethod = "destory")

好文：https://blog.csdn.net/liujun03/article/details/81671041?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.channel_param



## 什么时候调用

@Configuration用于定义配置类，可替换xml配置文件，被注解的类内部包含有一个或多个被@Bean注解的方法，这些方法将会被AnnotationConfigApplicationContext或AnnotationConfigWebApplicationContext类进行扫描，并用于构建bean定义，初始化Spring容器。

![Xnip2019-06-11_18-00-13](/img/Xnip2019-06-11_18-00-13.png)

从这里可以看到， spring启动后，两个bean已经自动加载。

# @Import

转载地址：https://blog.csdn.net/pange1991/article/details/81356594

在应用中，有时没有把某个类注入到IOC容器中，但在**运用的时候**需要获取该类对应的bean，此时就需要用到@Import注解。

![Xnip2019-06-11_18-02-13](/img/Xnip2019-06-11_18-02-13.png)

# @Component

把普通pojo实例化到spring容器中，相当于配置文件中的<bean id="" class=""/>



## @component和@Bean的区别

转载地址：https://blog.csdn.net/w605283073/article/details/89221522

@Component`（和`@Service`和`@Repository`）**用于自动检测和使用类路径扫描自动配置bean**。注释类和bean之间存在隐式的一对一映射（即每个类一个bean）。这种方法对需要进行逻辑处理的控制非常有限，因为它纯粹是声明性的。

@Bean`用于**显式声明单个bean**，而不是让Spring像上面那样自动执行它。它将bean的声明与类定义分离，并允许您精确地创建和配置bean。

```java
@Component
public class Student {
 
    private String name = "lkm";
 
    public String getName() {
        return name;
    }
 
    public void setName(String name) {
        this.name = name;
    }
}
```

而@Bean则常和@Configuration注解搭配使用

```java
@Configuration
public class WebSocketConfig {
    @Bean
    public Student student(){
        return new Student();
    }
 
}
```

上面可以看出一个是相当于实例化了对象，一个只是声明了一个bean对象。 



## 为什么有了@Compent,还需要@Bean呢

如果想将第三方的类变成组件，你又没有没有源代码，也就没办法使用`@Component`进行自动配置，这种时候使用`@Bean`就比较合适了。不过同样的也可以通过xml方式来定义。另外`@Bean注解的方法返回值是对象，可以在方法中为对象设置属性。比如我们使用**第三方的配置Druid的监控**。

```java
@Configuration
public class DruidConfig {
    /***
     * 
     * @return  配置Druid的监控,配置一个管理后台的Servlet
     */
    @Bean
    public ServletRegistrationBean statViewServlet() {
        ServletRegistrationBean bean = new ServletRegistrationBean(new StatViewServlet(), "/druid/*");
        Map<String, String> initParams = new HashMap<>();

        initParams.put("loginUsername", "admin");
        initParams.put("loginPassword", "admin");
        initParams.put("allow", "");// IP白名单 (没有配置或者为空，则允许所有访问)
        initParams.put("deny", "192.168.15.21"); //IP黑名单 (存在共同时，deny优先于allow)

        bean.setInitParameters(initParams);
        return bean;
    }

    /***
     *  
     *   配置一个web监控的filter
     * */
    @Bean
    public FilterRegistrationBean webStatFilter() {
        FilterRegistrationBean bean = new FilterRegistrationBean();
        bean.setFilter(new WebStatFilter());
        Map<String, String> initParams = new HashMap<>();
        initParams.put("exclusions", "*.js,*.gif,*.jpg,*.bmp,*.png,*.css,*.ico,/druid/*");
        bean.setInitParameters(initParams);
        bean.setUrlPatterns(Arrays.asList("/*"));
        return bean;
    }
}
```

这样我们就直接可以是对应的功能了。

# @ComponentScan

**转载地址：https://blog.51cto.com/4247649/2118342**

@ComponentScan等价于<context:component-scan base-package="com.dxz.demo"/>，@ComponentScan主要就是定义**扫描的路径**从中找出标识了**需要装配**的类自动装配到spring的bean容器中。@ComponentScan注解默认就会装配标识了@Controller，@Service，@Repository，@Component注解的类到spring容器中。

## 常用方式

- **自定扫描路径下边带有@Controller，@Service，@Repository，@Component注解加入spring容器**

- **通过includeFilters加入扫描路径下没有以上注解的类加入spring容器**
- **通过excludeFilters过滤出不用加入spring容器的类**
- **自定义增加了@Component注解的注解方式**

## 实例证明

![Xnip2019-06-11_18-54-26](/img/Xnip2019-06-11_18-54-26.png)

包扫描的方式比以前介绍的通过@Bean注解的方式是不是方便很多。

# @Repository，@Service，@Controller

Spring的注解形式：@Repository、@Service、@Controller，它们分别对应存储层Bean，业务层Bean，和展示层Bean。

**这4种注解是没什么本质区别,都是声明作用,取不同的名字只是为了更好区分各自的功能**。

## 为什么有时候不用@Repostitory注解

**转载地址：https://blog.csdn.net/f45056231p/article/details/81676039**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.databasedemo.mybatis.SchoolXmlMapper">
    <select id="getSchoolById" resultType="com.example.databasedemo.mybatis.School">
         select * from school where id=#{id}
    </select>
</mapper>
```

是因为我们在mybatis的xml文件配置了这个bean,它会去将dao这个层中的mapper(也就是我们的接口)都生成实现类,然后交给spring管理(因为mybatis.xml文件我们最终还是导入了spring容器中),所以我们这里不对这些接口用@repository注解,也是一样可以用它的实现类,(这也是我们写项目时,有时感觉完全是没用到@repository注解的原因,因为没有什么必要)

# @Autowired, @Resource

@Resource和@Autowired注解都是用来实现依赖注入的。只是@AutoWried按by type自动注入(实例对象类型比较)，而@Resource默认按byName自动注入(对象名字)。



## 依赖注入，控制反转

**转载地址：https://blog.csdn.net/taijianyu/article/details/2338311 作者写的非常好**

依赖注入(Dependency Injection)和控制反转(Inversion of Control)是同一个概念。具体含义是:当某个角色(可能是一个Java实例，调用者)需要另一个角色(另一个Java实例，被调用者)的协助时，在 传统的程序设计过程中，通常由调用者来创建被调用者的实例。**但在Spring里，创建被调用者的工作不再由调用者来完成，因此称为控制反转;创建被调用者 实例的工作通常由Spring容器来完成，然后注入调用者，因此也称为依赖注入。**

**不管是依赖注入，还是控制反转，都说明Spring采用动态、灵活的方式来管理各种对象**。对象与对象之间的具体实现互相透明。在理解**依赖****注入**之前，看如下这个问题在各种社会形态里如何解决:一个人(Java实例，调用者)需要一把斧子(Java实例，被调用者)。

(1)原始社会里，几乎没有社会分工。需要斧子的人(调用者)只能自己去磨一把斧子(被调用者)。对应的情形为:Java程序里的调用者自己创建被调用者。

(2)进入工业社会，工厂出现。斧子不再由普通人完成，而在工厂里被生产出来，此时需要斧子的人(调用者)找到工厂，购买斧子，无须关心斧子的制造过程。对应Java程序的简单工厂的设计模式。

(3)进入“按需分配”社会，需要斧子的人不需要找到工厂，坐在家里发出一个简单指令:需要斧子。斧子就自然出现在他面前。对应Spring的**依赖****注入**。

第一种情况下，Java实例的调用者创建被调用的Java实例，必然要求被调用的Java类出现在调用者的代码里。无法实现二者之间的松耦合。

第二种情况下，调用者无须关心被调用者具体实现过程，只需要找到符合某种标准(接口)的实例，即可使用。此时调用的代码面向接口编程，可以让调用者和被调用者解耦，这也是工厂模式大量使用的原因。但调用者需要自己定位工厂，调用者与特定工厂耦合在一起。

第三种情况下，调用者无须自己定位工厂，程序运行到需要被调用者时，系统自动提供被调用者实例。事实上，调用者和被调用者都处于Spring的管理下，二者之间的**依赖**关系由Spring提供。

所谓**依赖注入**，**是**指程序运行过程中，如果需要调用另一个对象协助时，无须在代码中创建被调用者，而是**依赖**于外部的**注入**。Spring的**依赖注入**对调用者和被调用者几乎没有任何要求，完全支持对POJO之间**依赖**关系的管理。**依赖注入**通常有两种: **设值注入**，**设值注入**

**设值注入**：指通过setter方法传入被调用者的实例。这种**注入**方式简单、直观，因而在Spring的**依赖****注入**里大量使用

```xml
＜BEANS＞
    ＜!—定义第一bean，该bean的id是chinese, class指定该bean实例的实现类 --＞
    ＜BEAN class=lee.Chinese id=chinese＞
    ＜!-- property元素用来指定需要容器注入的属性，axe属性需要容器注入此处是设值注入，因此Chinese类必须拥有setAxe方法 --＞
    ＜property name="axe"＞
    ＜!-- 此处将另一个bean的引用注入给chinese bean --＞
    ＜REF local="”stoneAxe”/"＞
    ＜/property＞
    ＜/BEAN＞
    ＜!-- 定义stoneAxe bean --＞
    ＜BEAN class=lee.StoneAxe id=stoneAxe /＞
＜/BEANS＞
```

从配置文件中，可以看到Spring管理bean的灵巧性。bean与bean之间的**依赖**关系放在配置文件里组织，而不是写在代码里。通过配置文件的 指定，Spring能精确地为每个bean**注入**属性。因此，配置文件里的bean的class元素，不能仅仅**是**接口，而必须**是**真正的实现类。

Spring会自动接管每个bean定义里的property元素定义。Spring会在执行无参数的构造器后、创建默认的bean实例后，调用对应 的setter方法为程序**注入**属性值。property定义的属性值将不再由该bean来主动创建、管理，而改为被动接收Spring的**注入**。

每个bean的id属性**是**该bean的惟一标识，程序通过id属性访问bean，bean与bean的**依赖**关系也通过id属性完成。

## @Autowired

**转载地址：https://www.cnblogs.com/caoyc/p/5626365.html**

@Autowired 注释，**它可以对类成员变量、方法及构造函数进行标注**，**完成自动装配的工作**。 通过 @Autowired的使用来消除 set ，get方法。在使用@Autowired之前，我们对一个bean配置起属性时，是这用用的

```xml
<property name="属性名" value=" 属性值"/>    
```

通过这种方式来，配置比较繁琐，而且代码比较多。在Spring 2.5 引入了 @Autowired 注释。

### 那么使用@Autowired的原理是什么？

其实在启动spring IoC时，容器自动装载了一个AutowiredAnnotationBeanPostProcessor后置处理器，当容器扫描到@Autowied、@Resource或@Inject时，**就会在IoC容器自动查找需要的bean，并装配给该对象的属性**。

### **注意事项**

在使用@Autowired时，首先在容器中查询对应类型的bean，

1.如果查询结果刚好为一个，就将该bean装配给@Autowired指定的数据

2.如果查询的结果不止一个，那么@Autowired会根据名称来查找。 **如果我们想使用按照名称（byName）来装配，可以结合@Qualifier注解一起使用**， 因为Autowired默认是按照类型来查找的。 

```java
@Autowired  
@Qualifier("office")   
private Office office; 

@Qualifier 只能和 @Autowired 结合使用，是对 @Autowired 有益的补充。
@Qualifier("office") 中的 office 是 Bean 的名称。

```

3.如果查询的结果为空，那么会抛出异常。解决方法时，使用required=false.相当于@Autowired(required = **false**)，这等于告诉 Spring：在找不到匹配 Bean 时也不报错。

在默认情况下使用 @Autowired 注释进行自动注入时，**Spring 容器中匹配的候选 Bean 数目必须有且仅有一个**。

## @Resource

@Resource依赖注入时查找bean的规则。

既不指定name属性，也不指定type属性，则自动按byName方式进行查找。如果没有找到符合的bean，则回退为一个原始类型进行查找，如果找到就注入。

# @Transactional

**转载地址：https://www.cnblogs.com/jpfss/p/9151308.html**

原理采用AOP和动态代理，实现是了事物处理。 

## 使用方式

1. spring boot 添加事物使用 @Transactional注解

2. 简单使用 在启动类上方添加 @EnableTransactionManagement注解

3. 使用时直接在**类或者方法**上使用 @Transactional注解

## 需要注意地方

1. 不要在接口上声明 @Transactional ，而要在具体类的方法上使用 @Transactional 注解，否则注解可能无效。 

2. 将 @Transactional  放置在类级的声明中 放在类声明 会使得所有方法都有事务 故 @Transactional应该放在方法级别 不需要使用事务的方法，就不要放置事务，比如查询方法。否则对性能是有影响的。  
3. 使用了 @Transactional的方法，对同一个类里面的方法调用， @Transactional无效。比如有一个类Test，它的一个方法A，A再调用Test本类的方法B（不管B是否**public**还是**private**），但A没有声明注解事务，而B有。则外部调用A之后，B的事务是不会起作用的。（经常在这里出错）
4. 使用了 @Transactional 的方法， 只能是**public**， @Transactional注解的方法都是被外部其他类调用才有效，故只能是**public**。道理和上面的有关联。故在 **protected**、**private** 或者 **package**-visible 的方法上使用 @Transactional 注解，它也不会报错，但事务无效。  
5. 在service中加上 @Transactional，如果是action直接调该方法，会回滚，如果是间接调，不会回滚。
6. 在service中的**private**加上 @Transactional，事务不会回滚。  

# @Scope

**转载地址：https://blog.csdn.net/weixin_32916879/article/details/81350968**

spring中scope是一个非常关键的概念，**简单说就是对象在spring容器（IOC容器）中的生命周期**，也可以理解为对象在spring容器中的创建方式。

## singleton

此取值时表明容器中创建时只存在一个实例，所有引用此bean都是单一实例。

## prototype

也就是说，容器每次返回请求方该对象的一个新的实例之后，就由这个对象“自生自灭”，

## request

再次说明request，session和global session类型只实用于web程序，当同时有100个HTTP请求进来的时候，容器会分别针对这10个请求创建10个全新的RequestPrecessor实例，且他们相互之间互不干扰，简单来讲，request可以看做prototype的一种特例，除了场景更加具体之外，语意上差不多。

## session

Spring容器会为每个独立的session创建属于自己的全新的UserPreferences实例，比request scope的bean会存活更长的时间，其他的方面没区别，**如果java web中session的生命周期**。

## global session

global session只有应用在基于porlet的web应用程序中才有意义，它映射到porlet的global范围的session，如果普通的servlet的web 应用中使用了这个scope，容器会把它作为普通的session的scope对待。 

# @RequestParam，@PathVariable

@RequestParam` 和 `@PathVariable` 注解是用于从request中接收请求的，两个都可以接收参数，关键点不同的是`@RequestParam` 是从request里面拿取值，而 `@PathVariable` 是从一个URI模板里面来填充。

```java
@RequestMapping("/zyh/{type}")
public String zyh(@PathVariable(value = "type") int type)
```



# @Inject

转载地址：https://www.cnblogs.com/pjfmeng/p/7551340.html

1、@Inject是JSR330 (Dependency Injection for Java)中的规范，需要导入javax.inject.Inject;实现注入。 **他是java注解**

2、@Inject是根据**类型**进行自动装配的，如果需要按名称进行装配，则需要配合@Named；

3、@Inject可以作用在变量、setter方法、构造函数上。

**a、**将@Inject可以作用在变量、setter方法、构造函数上，和@Autowired一样

![1050601-20170919114448087-472523137](/img/1050601-20170919114448087-472523137.png)

**b、@Named**

**@Named("XXX") 中的 XX是 Bean 的名称，所以 @Inject和 @Named结合使用时，自动注入的策略就从 byType 转变成 byName 了。**

![1050601-20170919114552321-1115699816](/img/1050601-20170919114552321-1115699816.png)

# @Qualifier

@Qualifier("XXX") 中的 XX是 Bean 的名称，所以 @Autowired 和 @Qualifier 结合使用时，自动注入的策略就从 byType 转变成 byName 了。不过需要注意的是@Autowired 可以对成员变量、方法以及构造函数进行注释，而 @Qualifier 的标注对象是成员变量、方法**入参**、构造函数**入参**。

```java
@Autowired
public void setDataSource(@Qualifier("myDataSource") DataSource dataSouce){
     //这里就是直接把 bean  myDataSource对象直接使用了， 
     dataSouce.get....
}
```

# @Aspect

转载地址：https://www.cnblogs.com/lc0605/p/10694489.html

要想把一个类变成切面类，需要两步，

① 在类上使用 @Component 注解 把切面类加入到IOC容器中 
② 在类上使用 @Aspect 注解 使之成为切面类

```java
@Aspect
@Component
public class AspectTest {
    /**
     * 前置通知：目标方法执行之前执行以下方法体的内容
     * @param jp
     */
    @Before("execution(* com.springboot.aop.controller.*.*(..))")
    public void beforeMethod(JoinPoint jp){
        String methodName = jp.getSignature().getName();
        System.out.println("【前置通知】the method 【" + methodName + "】 begins with " + Arrays.asList(jp.getArgs()));
    }

    /**
     * 返回通知：目标方法正常执行完毕时执行以下代码
     * @param jp
     * @param result
     */
    @AfterReturning(value="execution(* com.springboot.aop.controller.*.*(..))",returning="result")
    public void afterReturningMethod(JoinPoint jp, Object result){
        String methodName = jp.getSignature().getName();
        System.out.println("【返回通知】the method 【" + methodName + "】 ends with 【" + result + "】");
    }

    /**
     * 后置通知：目标方法执行之后执行以下方法体的内容，不管是否发生异常。
     * @param jp
     */
    @After("execution(* com.springboot.aop.controller.*.*(..))")
    public void afterMethod(JoinPoint jp){
        System.out.println("【后置通知】this is a afterMethod advice...");
    }

    /**
     * 异常通知：目标方法发生异常的时候执行以下代码
     */
    @AfterThrowing(value="execution(* com.springboot.aop.controller.*.*(..))",throwing="e")
    public void afterThorwingMethod(JoinPoint jp, NullPointerException e){
        String methodName = jp.getSignature().getName();
        System.out.println("【异常通知】the method 【" + methodName + "】 occurs exception: " + e);
    }

  /**
   * 环绕通知：目标方法执行前后分别执行一些代码，发生异常的时候执行另外一些代码
   * @return
   */
  @Around(value="execution(* com.springboot.aop.controller.*.*(..))")
  public Object aroundMethod(ProceedingJoinPoint jp){
      String methodName = jp.getSignature().getName();
      Object result = null;
      try {
          System.out.println("【环绕通知中的--->前置通知】：the method 【" + methodName + "】 begins with " + Arrays.asList(jp.getArgs()));
          //执行目标方法
          result = jp.proceed();
          System.out.println("【环绕通知中的--->返回通知】：the method 【" + methodName + "】 ends with " + result);
      } catch (Throwable e) {
          System.out.println("【环绕通知中的--->异常通知】：the method 【" + methodName + "】 occurs exception " + e);
      }

      System.out.println("【环绕通知中的--->后置通知】：-----------------end.----------------------");
      return result;
  }
}
```

# 动态注入bean



## @ConditionalOnResource

转载地址：https://www.jianshu.com/p/d05f5c146452

当**指定的资源文件**出现在classpath中生效

**定义**

```java
@Target({ ElementType.TYPE, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(OnResourceCondition.class)
public @interface ConditionalOnResource {

    /**
     * 资源文件必须指定
     */
    String[] resources() default {};

}
```

**示例**

```java
org.apache.shiro.spring.boot.autoconfigure.ShiroAutoConfiguration类中

@Bean
@ConditionalOnResource(resources="classpath:shiro.ini")
protected Realm iniClasspathRealm()
```



## @Conditional

转载地址：https://blog.csdn.net/xcy1193068639/article/details/81491071

@Conditional是Spring4新提供的注解，它的作用是按照一定的条件进行判断，满足条件给容器注册bean。根据判断是否实例化bean对象。动态注入Bean的处理。 



## @ConditionalOnMissingBean

转载地址：https://www.cnblogs.com/YuyuanNo1/p/12511121.html

Spring4推出了@Conditional注解，方便程序根据当前环境或者容器情况来动态注入bean， 继@Conditional注解后，又基于此注解推出了很多派生注解，比如@ConditionalOnBean、@ConditionalOnMissingBean  @ConditionalOnExpression、@ConditionalOnClass......动态注入bean变得更方便了。 

说明： 配置类中有两个Computer类的bean，一个是笔记本电脑，一个是备用电脑。如果当前容器中已经有电脑bean了，就不注入备用电脑，如果没有，则注入备用电脑，这里需要使用到@ConditionalOnMissingBean。

# @ControllerAdvice 

`@ControllerAdvice`，是Spring3.2提供的新注解，从名字上可以看出大体意思是控制器增强

该注解使用`@Component`注解，这样的话当我们使用`<context:component-scan>`扫描时也能扫描到。

一般都是用于异常的处理， 配合  @ExceptionHandler 进行全局异常处理。 

# @MapperScan

在SpringBoot中集成MyBatis，可以在mapper接口上添加@Mapper注解，将mapper注入到Spring,但是如果每一给mapper都添加@mapper注解会很麻烦，这时可以使用@MapperScan注解来扫描包。



# 如何理解spring的容器的概念

实际上，容器里面什么都没有，决定容器里面放什么对象的是我们自己，决定对象之间的依赖关系的，也是我们自己，容器只是给我们提供一个管理对象的空间而已。

好文：https://zhuanlan.zhihu.com/p/69010848

# 总结

 Spring框架包含Ioc和Aop，其中很多注解都是为了方便创建对象，以及什么时候创建bean的判断，我们一个请求过来，为什么进入@Controller中，也是因为在扫描这个@Controller注解，看是否有方法和他的value值相等，然后找到特定的方法中来，**所以也就是一个路牌，为了就是为了找到对应的方法**，找到对应的类中，找到了就执行方法处理起来。 







