---
title:       "springboot系列-日志"
subtitle:    ""
description: ""
date:        2019-05-25
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["springboot系列", "日志"]
categories:  ["Tech" ]
---

[TOC]

**说明： 下列文章主要对尚硅谷《SpringBoot视频教程》的总结，下列文字描述来源他们的课件。**

**视频地址： https://www.bilibili.com/video/av38657363/?p=9**

**spingboot版本<version>2.1.5.RELEASE</version>**

# 日志级别

```java
//由低到高   trace<debug<info<warn<error
//可以调整输出的日志级别；日志就只会在这个级别以以后的高级别生效
logger.trace("这是trace日志...");
logger.debug("这是debug日志...");
//SpringBoot默认给我们使用的是info级别的，没有指定级别的就用SpringBoot默认规定的级别；root级别
logger.info("这是info日志...");
logger.warn("这是warn日志...");
logger.error("这是error日志...");
```

# 日志使用

添加**lombok**之后，直接注解

```java
import lombok.extern.log4j.Log4j2;

@Log4j2
@RestController
public class StudentController {

    @RequestMapping("/msg")
    public Student msg() {
        Student student = new Student();
        student.name = "麦子";
        student.age = 20;
        log.info("我是日志");
        return student;
    }

}
```

# 日志输出到文件

```yaml
#日志文件输入到固定的文件中，这个是当前项目路径
logging:
  file: springboot.log

#日志输出到具体的一个目录文件中
logging:
  file: /Users/maizi/Desktop/springboot.log #到指定目录
  pattern:
   console: '%d{yyyy-MMM-dd HH:mm:ss.SSS} %-5level [%thread] %logger{15} - %msg%n' #控制台格式
   file: '%d{yyyy-MMM-dd HH:mm:ss.SSS} %-5level [%thread] %logger{15} - %msg%n' #到文件格式
```

# logback自定义格式

**logback.xml：**直接就被日志框架识别了；

**logback-spring.xml：**日志框架就不直接加载日志的配置项，由SpringBoot解析日志配置，可以使用SpringBoot的高级Profile功能。

```xml
<springProfile name="staging">
    <!-- configuration to be enabled when the "staging" profile is active -->
  	可以指定某段配置只在某个环境下生效
</springProfile>
```

**注意：如果配置文件配置了，然后logback也配置了。 最后还是按照logback的格式来输出。**

## 完成一个logback.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
scan：当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。
scanPeriod：设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒当scan为true时，此属性生效。默认的时间间隔为1分钟。
debug：当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。
-->
<configuration scan="false" scanPeriod="60 seconds" debug="false">
    <!-- 定义日志的根目录 -->
    <property name="LOG_HOME" value="/Users/maizi/Desktop" />
    <!-- 定义日志文件名称 -->
    <property name="appName" value="logdemo"></property>
    <!-- ch.qos.logback.core.ConsoleAppender 表示控制台输出 -->
    <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
        <!--
        日志输出格式：
			%d表示日期时间，
			%thread表示线程名，
			%-5level：级别从左显示5个字符宽度
			%logger{50} 表示logger名字最长50个字符，否则按照句点分割。 
			%msg：日志消息，
			%n是换行符
        -->
        <layout class="ch.qos.logback.classic.PatternLayout">
            <springProfile name="dev">
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} ----> [%thread] ---> %-5level %logger{50} - %msg%n</pattern>
            </springProfile>
            <springProfile name="!dev">
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} ==== [%thread] ==== %-5level %logger{50} - %msg%n</pattern>
            </springProfile>
        </layout>
    </appender>

    <!-- 滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件 -->  
    <appender name="appLogAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 指定日志文件的名称 -->
        <file>${LOG_HOME}/${appName}.log</file>
        <!--
        当发生滚动时，决定 RollingFileAppender 的行为，涉及文件移动和重命名
        TimeBasedRollingPolicy： 最常用的滚动策略，它根据时间来制定滚动策略，既负责滚动也负责出发滚动。
        -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--
            滚动时产生的文件的存放位置及文件名称 %d{yyyy-MM-dd}：按天进行日志滚动 
            %i：当文件大小超过maxFileSize时，按照i进行文件滚动
            -->
            <fileNamePattern>${LOG_HOME}/${appName}-%d{yyyy-MM-dd}-%i.log</fileNamePattern>
            <!-- 
            可选节点，控制保留的归档文件的最大数量，超出数量就删除旧文件。假设设置每天滚动，
            且maxHistory是365，则只保存最近365天的文件，删除之前的旧文件。注意，删除旧文件是，
            那些为了归档而创建的目录也会被删除。
            -->
            <MaxHistory>365</MaxHistory>
            <!-- 
            当日志文件超过maxFileSize指定的大小是，根据上面提到的%i进行日志文件滚动 注意此处配置SizeBasedTriggeringPolicy是无法实现按文件大小进行滚动的，必须配置timeBasedFileNamingAndTriggeringPolicy
            -->
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <!-- 日志输出格式： -->     
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [ %thread ] - [ %-5level ] [ %logger{50} : %line ] - %msg%n</pattern>
        </layout>
    </appender>

    <!-- 
		logger主要用于存放日志对象，也可以定义日志类型、级别
		name：表示匹配的logger类型前缀，也就是包的前半部分
		level：要记录的日志级别，包括 TRACE < DEBUG < INFO < WARN < ERROR
		additivity：作用在于children-logger是否使用 rootLogger配置的appender进行输出，
		false：表示只用当前logger的appender-ref，true：
		表示当前logger的appender-ref和rootLogger的appender-ref都有效
    -->
    <!-- hibernate logger -->
    <logger name="com.example" level="debug" />
    <!-- Spring framework logger -->
    <logger name="org.springframework" level="debug" additivity="false"></logger>



    <!-- 
    root与logger是父子关系，没有特别定义则默认为root，任何一个类只会和一个logger对应，
    要么是定义的logger，要么是root，判断的关键在于找到这个logger，然后判断这个logger的appender和level。 
    -->
    <root level="info">
        <appender-ref ref="stdout" />
        <appender-ref ref="appLogAppender" />
    </root>
</configuration> 
```

**注意：在这里也可以设置控制台的输出格式和颜色等。** 

## 日志打印结果

```verilog
2019-05-25 22:49:56.576 [ http-nio-8082-exec-1 ] - [ INFO  ] [ com.example.logdemo.controller.StudentController : 26 ] - 我是日志
2019-05-25 22:49:57.402 [ http-nio-8082-exec-2 ] - [ INFO  ] [ com.example.logdemo.controller.StudentController : 26 ] - 我是日志
2019-05-25 22:49:58.087 [ http-nio-8082-exec-3 ] - [ INFO  ] [ com.example.logdemo.controller.StudentController : 26 ] - 我是日志
```

可以看到， 这个日志打印出来，设置了， **连打印的代码行数也显示出来了。** 