---
title:       "SpringCloud组件流程代码演示"
subtitle:    ""
description: "Eureka，Ribbon, Hystrix,Feign,Zuul,Config,Bus,Stream,Sleuth,Shiro,spring-boot-actuator,Spring Boot Batch, 一个简单的SpringCloud例子"
date:        2020-07-02
author:      "麦子"
image:       "https://img.zhaohuabing.com/in-post/istio-install_and_example/post-bg.jpg"
tags:        ["springboot", "springCloud"]
categories:  ["Tech" ]
---

[TOC]



# maven模块化基础结构

![Xnip2020-07-04_09-37-48](/img/Xnip2020-07-04_09-37-48.png)

通过构建maven模块化， 达到后期改动其他业务模块的时候， 只需要在这个maven工程进行 clear， install， package操作就可以了。具体pom文件如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>maizi.cloud</groupId>
    <artifactId>mycloud</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>


    <dependencies>
        <!-- 当前Module需要用到的jar包，按自己需求添加，如果父类已经包含了，可以不用写版本号 -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
            <version>1.18.4</version>
        </dependency>
    </dependencies>


    <modules>
        <module>MyTools</module>
        <module>MyProvide-8001</module>
        <module>MyProvide-8002</module>
        <module>MyConsumer-9001</module>
    </modules>
</project>
```

当然一些常用的jar也可以在父类中直接引入。 



# 工具类MyTool模块

![Xnip2020-07-04_09-43-03](/img/Xnip2020-07-04_09-43-03.png)

在这个类里面，我们抽离了公共的实体类，当然也可以添加更多的基础工具类操作。

# Eureka（服务治理）

**设置host文件**

```properties
127.0.0.1      eureka5001.com
127.0.0.1      eureka5002.com
127.0.0.1      eureka5003.com
```

**集群搭建 application.yml文件**

```yml
server:
  port: 5001
eureka:
  instance:
    hostname: eureka5001.com #eureka服务端的实例名称, 我这边做了host映射
  client:  #声明自己是服务端
    register-with-eureka: false  #false表示不向注册中心注册自己。
    fetch-registry: false
    service-url:
     defaultZone: http://eureka5002.com:5002/eureka/,http://eureka5003.com:5003/eureka/
```

```yml
server:
  port: 5002
eureka:
  instance:
    hostname: eureka5002.com #eureka服务端的实例名称, 我这边做了host映射
  client: #声明自己是服务端
    register-with-eureka: false  #false表示不向注册中心注册自己。
    fetch-registry: false
    service-url:
     defaultZone: http://eureka5001.com:5001/eureka/,http://eureka5003.com:5003/eureka/
```

```yml
server:
  port: 5003
eureka:
  instance:
    hostname: eureka5003.com #eureka服务端的实例名称, 我这边做了host映射
  client: #声明自己是服务端
    register-with-eureka: false  #false表示不向注册中心注册自己。
    fetch-registry: false
    service-url:
     defaultZone: http://eureka5001.com:5001/eureka/,http://eureka5002.com:5002/eureka/
```

**pom文件**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.4.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.eureka</groupId>
	<artifactId>eurekademo</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>eurekademo</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
		<spring-cloud.version>Greenwich.SR1</spring-cloud.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>

```



## 显示效果

![Xnip2020-07-02_09-46-29](/img/Xnip2020-07-02_09-46-29.png)

如上图，我们搭建了3个Eureka的服务，



# 服务提供方

### 

## MyProvide-8001

### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

	<parent>
		<artifactId>mycloud</artifactId>
		<groupId>maizi.cloud</groupId>
		<version>1.0-SNAPSHOT</version>
		<relativePath>../pom.xml</relativePath>
	</parent>
	<modelVersion>4.0.0</modelVersion>
	<artifactId>provide-8001</artifactId>
	<version>0.0.1-SNAPSHOT</version>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<maven.compiler.encoding>UTF-8</maven.compiler.encoding>
		<java.version>1.8</java.version>
		<maven.compiler.source>1.8</maven.compiler.source>
		<maven.compiler.target>1.8</maven.compiler.target>
		<spring-cloud.version>Greenwich.SR1</spring-cloud.version>
	</properties>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<!-- Import dependency management from Spring Boot -->
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-dependencies</artifactId>
				<version>2.1.4.RELEASE</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>

			<!--<dependency>-->
			<!--<groupId>org.springframework.cloud</groupId>-->
			<!--<artifactId>spring-cloud-dependencies</artifactId>-->
			<!--<version>${spring-cloud.version}</version>-->
			<!--<type>pom</type>-->
			<!--<scope>import</scope>-->
			<!--</dependency>-->

		</dependencies>
	</dependencyManagement>


	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
			<version>2.1.1.RELEASE</version>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>

		<dependency>
			<groupId>maizi.cloud</groupId>
			<artifactId>mytools</artifactId>
			<version>1.0-SNAPSHOT</version>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<configuration>
					<source>1.8</source>
					<target>1.8</target>
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
				<executions>
					<execution>
						<goals>
							<goal>repackage</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>

</project>

```

### application.xml

```yml
server:
  port: 8001

spring:
  application:
    name: provide-data #入驻eureka的服务名称

eureka:
  client:
    service-url:
      defaultZone: http://eureka5001.com:5001/eureka,http://eureka5002.com:5002/eureka,http://eureka5003.com:5003/eureka
```

### 具体代码

```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
public class MyprovideApplication {

	public static void main(String[] args) {
		SpringApplication.run(MyprovideApplication.class, args);
	}

}
```

```java
package com.example.demo;

import entity.MaiziUser;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @version V2.0
 * @Description:
 * @date 2019/4/17 下午6:17
 * @Company: cecsm.com
 * @Copyright Copyright (c) 2017
 */
@RestController
public class ProvideController {

    @Autowired
    private ProvideService service;

    @RequestMapping("provide")
    public MaiziUser provide(String foo) {
        return service.buildData(foo);
    }
}

```

```java
package com.example.demo;

import entity.MaiziUser;
import org.springframework.stereotype.Service;

/**
 * @version V2.0
 * @Description:
 * @date 2019/4/17 下午6:18
 * @Company: cecsm.com
 * @Copyright Copyright (c) 2017
 */
@Service
public class ProvideService {

    public MaiziUser buildData(String foo)
    {
         MaiziUser user = new MaiziUser();
         user.setName("麦子->"+foo).setAge(28).setSex("男");
         return user;
    }
}

```



## MyProvide-8002

与上面的8001的服务有区别的地方

```java
package com.provide;

import entity.MaiziUser;
import org.springframework.stereotype.Service;

/**
 * @version V2.0
 * @Description:
 * @date 2019/4/18 上午5:31
 * @Company: cecsm.com
 * @Copyright Copyright (c) 2017
 */
@Service
public class ProvideService {

	public MaiziUser buildData(String foo) {
		MaiziUser user = new MaiziUser();
		user.setName("小强++++" + foo).setAge(28).setSex("女");
		return user;
	}
}

```

可以看到，返回的值不同， 这是为了区别到时候消费端调用的时候，进行负载均衡的测试用。 



## 效果

启动上面两个接口提供方后，查看Eueka的服务页面如下：

![Xnip2020-07-04_10-07-20](/img/Xnip2020-07-04_10-07-20.png)

可以看到， 服务提供方已经注册到了Eueka服务中。 



# 消费处理方

## pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

	<parent>
		<artifactId>mycloud</artifactId>
		<groupId>maizi.cloud</groupId>
		<version>1.0-SNAPSHOT</version>
		<relativePath>../pom.xml</relativePath>
	</parent>
	<modelVersion>4.0.0</modelVersion>
	<artifactId>consumer-9001</artifactId>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<maven.compiler.encoding>UTF-8</maven.compiler.encoding>
		<java.version>1.8</java.version>
		<maven.compiler.source>1.8</maven.compiler.source>
		<maven.compiler.target>1.8</maven.compiler.target>
		<spring-cloud.version>Greenwich.SR1</spring-cloud.version>
	</properties>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<!-- Import dependency management from Spring Boot -->
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-dependencies</artifactId>
				<version>2.1.4.RELEASE</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>


	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
			<version>2.1.1.RELEASE</version>
		</dependency>

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-openfeign</artifactId>
			<version>2.1.1.RELEASE</version>
		</dependency>

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
			<version>2.1.1.RELEASE</version>
		</dependency>

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
			<version>2.1.1.RELEASE</version>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
			<version>2.1.1.RELEASE</version>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>maizi.cloud</groupId>
			<artifactId>mytools</artifactId>
			<version>1.0-SNAPSHOT</version>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<configuration>
					<source>1.8</source>
					<target>1.8</target>
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
				<executions>
					<execution>
						<goals>
							<goal>repackage</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>
</project>
```

## application.yml

```yml
server:
  port: 9001

eureka:
  client:
    register-with-eureka: false  #false表示不向注册中心注册自己,这个也是和提供者的一个区别，这里只是做消费
    service-url:
      defaultZone: http://eureka5001.com:5001/eureka
feign:
  hystrix:
    enabled: true

```

## 具体代码

```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.openfeign.EnableFeignClients;
import org.springframework.context.annotation.Bean;

import com.netflix.hystrix.contrib.metrics.eventstream.HystrixMetricsStreamServlet;

@SpringBootApplication
//在启动该微服务的时候就能去加载我们的自定义Ribbon配置类，从而使配置生效
//@RibbonClient(name="PROVIDE-DATA",configuration= MySelfRule.class)
@EnableFeignClients
@EnableEurekaClient // 配置本应用将使用服务注册和服务发现
@EnableCircuitBreaker
public class ConsumerApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConsumerApplication.class, args);
	}

	@Bean
	public ServletRegistrationBean getServlet(){
		HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
		ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
		registrationBean.setLoadOnStartup(1);
		registrationBean.addUrlMappings("/actuator/hystrix.stream");
		registrationBean.setName("HystrixMetricsStreamServlet");
		return registrationBean;
	}
}

```

```java
package com.example.demo;

import entity.MaiziUser;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import com.example.demo.feign.DataService;

/**
 * @version V2.0
 * @Description:
 * @date 2019/4/17 下午6:44
 * @Company: cecsm.com
 * @Copyright Copyright (c) 2017
 */
@RestController
public class ConsumerController {

    @Autowired
    private DataService dataService; 

    @RequestMapping("getDataByFeign")
    private MaiziUser getDataByFeign()
    {
        return dataService.getDataByFeign();
    }
}

```

```java
package com.example.demo.feign;

import entity.MaiziUser;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.stereotype.Service;
import org.springframework.web.bind.annotation.RequestMapping;

/**
 * @version V2.0
 * @Description:
 * @date 2019/4/18 下午2:28
 * @Company: cecsm.com
 * @Copyright Copyright (c) 2017
 */

/**
@FeignClient(value = "PROVIDE-DATA",fallback = MyHystrixClientFallbackFactory.class)
这里就是从Eureka的服务注册中获取需要的服务提供方接口，下面的  @RequestMapping(value = "/provide") 这个就是具体的哪个接口，同时 fallback = MyHystrixClientFallbackFactory.class 这里处置，如果提供方服务挂了， 那么这个地方就进行相关处理，防止雪崩的出现。 
*/
@Service
@FeignClient(value = "PROVIDE-DATA",fallback = MyHystrixClientFallbackFactory.class)
public interface DataService {

    @RequestMapping(value = "/provide")
    MaiziUser getDataByFeign();
}

```

```java
package com.example.demo.feign;

import entity.MaiziUser;
import org.springframework.stereotype.Component;

/**
 * @version V2.0
 * @Description:
 * @date 2019/4/18 下午10:56
 * @Company: cecsm.com
 * @Copyright Copyright (c) 2017
 */
@Component
public class MyHystrixClientFallbackFactory implements DataService{
    public MaiziUser getDataByFeign() {
         return new MaiziUser().setName("服务已经停止").setAge(11).setSex("男");
    }
}

```

# 运行结果

![Xnip2020-07-04_10-21-16](/img/Xnip2020-07-04_10-21-16.png)

可以看到相同的访问，出现了三个不同的结果，最后一个提示服务器已经停止，就是停止服务提供方的时候， 防止雪崩处理方式。 



# Hystrix Dashboard

用于监控服务状况，监控一些接口，不过我们现在最新用的事 Spring boot admin， 因为Hystrix Dashboard页面丑，而且不更新了。

## pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.4.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.myhystrixdashboard</groupId>
	<artifactId>myhystrixdashboard</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>myhystrixdashboard</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
		<spring-cloud.version>Greenwich.SR1</spring-cloud.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>

```



## application.yml

```yml
server:
  port: 9999
```



## 具体代码

```java
package com.myhystrixdashboard;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.hystrix.dashboard.EnableHystrixDashboard;

@SpringBootApplication
@EnableHystrixDashboard
public class MyhystrixdashboardApplication {

	public static void main(String[] args) {
		SpringApplication.run(MyhystrixdashboardApplication.class, args);
	}
}

```



## 注意

我们需要监控的接口都是在启动时候有 @EnableCircuitBreaker 这个注解的。 

# gateway网关

## pom.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.4.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.example</groupId>
	<artifactId>demo</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>demo</name>
	<description>Demo project for Spring Boot</description>
	<properties>
		<java.version>1.8</java.version>
		<spring-cloud.version>Greenwich.SR1</spring-cloud.version>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-gateway</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
			<version>2.1.1.RELEASE</version>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
</project>
```



## application.yml

```yml
server:
  port: 6677


#id：我们自定义的路由 ID，保持唯一
#uri：目标服务地址
#predicates：路由条件，Predicate 接受一个输入参数，返回一个布尔值结果。该接口包含多种默认方法来将 Predicate 组合成其他复杂的逻辑（比如：与，或，非）。
#filters：过滤规则，本示例暂时没用。

eureka:
  client:
    service-url:
      defaultZone: http://eureka5001.com:5001/eureka,http://eureka5002.com:5002/eureka,http://eureka5003.com:5003/eureka

logging:
  level:
    org.springframework.cloud.gateway: debug

spring:
  application:
    name: cloud-gateway-eureka
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true
      routes:
        - id: add_request_parameter_route
          uri: lb://PROVIDE-DATA
          predicates:
            - Path=/**
```

## 具体代码

```java
package com.example.demo;
/*
 * @Description: 请输入....
 * @Author: 麦子
 * @Date: 2019-05-15 17:37:12
 * @LastEditTime: 2019-05-15 21:30:34
 * @LastEditors: 麦子
 */

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}

}
```

## 效果

![Xnip2020-07-04_16-45-09](/img/Xnip2020-07-04_16-45-09.png)

![Xnip2020-07-04_16-56-24](/img/Xnip2020-07-04_16-56-24.png)



## 注意

配置了

```
 uri: lb://PROVIDE-DATA
```

那么就自带了负载均衡的处理了。 



# 总结

如上， 一般的分布式主体流程也就完成了， 如果你还需要其他的组件比如：Spring Cloud Sleuth（分布式服务跟踪）

Spring Cloud Config（分布式配置中心）Spring Boot Admin(监控并发等操作) 等。