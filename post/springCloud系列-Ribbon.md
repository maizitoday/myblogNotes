---
title:       "springCloud系列-Ribbon"
subtitle:    ""
description: ""
date:        2019-04-18
author:      "麦子"
image:       "https://zhaohuabing.com/img/2018-12-27-the-obstacles-to-put-istio-into-production/background.jpg"
tags:        ["springCloud", "Ribbon", "负载均衡"]
categories:  ["Tech" ]
---

[TOC]

**说明： 本学习主要对尚硅谷周阳大神《SpringCloud视频》记录，以下文字来源这个视频PPT**

**说明： 版本springboot2.14**   **<spring-cloud.version>Greenwich.SR1</spring-cloud.version>**

# Ribbon是什么

Spring Cloud Ribbon是基于Netflix Ribbon实现的一套***客户端*** 负载均衡的工具。

简单的说，Ribbon是Netflix发布的开源项目，主要功能是提供客户端的软件负载均衡算法，将Netflix的中间层服务连接在一起。Ribbon客户端组件提供一系列完善的配置项如连接超时，重试等。简单的说，就是在配置文件中列出Load Balancer（简称LB）后面所有的机器，Ribbon会自动的帮助你基于某种规则（如简单轮询，随机连接等）去连接这些机器。我们也很容易使用Ribbon实现自定义的负载均衡算法。

# 配置Ribbon服务

总结：Ribbon其实就是一个软**负载均衡的客户端组件**

他可以和其他所需请求的客户端结合使用，和eureka结合只是其中的一个实例。

## pom.xml文件修改

在需要调用的微服务的工程加入以下：

```xml
<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
			<version>2.1.1.RELEASE</version>
		</dependency>

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
			<version>2.1.1.RELEASE</version>
</dependency>
```



## application.yml

```yaml
server:
  port: 9001

eureka:
  client:
    register-with-eureka: false#false表示不向注册中心注册自己,这个也是和提供者的一个区别，这里只是做消费
    service-url:
      defaultZone: http://eureka5001.com:5001/eureka,http://eureka5002.com:5002/eureka,http://eureka5003.com:5003/eureka
```





## 加入@LoadBalanced注解

```java
/***
 * boot -->spring   applicationContext.xml --- @Configuration配置   ConfigBean = applicationContext.xml
 */
@Configuration
public class RestTemplateConfig {  
	@Bean
	@LoadBalanced
	public RestTemplate getRestTemplate() {
		return new RestTemplate();
	}

	@Bean
	public IRule myRule()
	{
		//return new RoundRobinRule(); //达到的目的，用我们重新选择的随机算法替代默认的轮询。
		return new RandomRule();//随机
	}
}
```



## 集群provide

![19-1](/img/19-1.png)



## applicationName调用provide

```java
@RestController
public class ConsumerController {

    @Autowired
    private RestTemplate restTemplate;

    private static final String  URL = "http://PROVIDE-DATA/";

    @RequestMapping("getData")
    private MaiziUser getData()
    {
        return restTemplate.getForObject(URL+ "provide",MaiziUser.class);
    }
}
```



# 自定义负载均衡算法

```java
@SpringBootApplication
//在启动该微服务的时候就能去加载我们的自定义Ribbon配置类，从而使配置生效
@RibbonClient(name="PROVIDE-DATA",configuration= MySelfRule.class)
public class ConsumerApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConsumerApplication.class, args);
	}
}
```

需要注意 MySelfRule 这个类的路径不能和@ComponentScan这个注解在同一路径。(注：没有测试这个问题)

```java
@Configuration
public class MySelfRule {
    @Bean
    public IRule myRule()
    {
        //return new RandomRule();// Ribbon默认是轮询，我自定义为随机
        return new RoundRobinRule();//  轮询
        //return new RandomRule_ZY();// 我自定义为每台机器5次
    }
}
// 如何自定义可以参考RoundRobinRule这个类
```



# 已经提供的负载均衡算法

![19-2](/img/19-2.png)