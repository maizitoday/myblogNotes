---
title:       "跨域"
subtitle:    ""
description: "什么是跨域，为什么跨域，跨域和网关的联系"
date:        2020-06-06
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["java相关概念"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：https://cloud.tencent.com/developer/article/1175899**

**好文推荐：https://segmentfault.com/a/1190000015597029，https://www.jianshu.com/p/f880878c1398**

# 什么是跨域？

跨域是指一个域下的文档或脚本试图去请求另一个域下的资源，这里跨域是广义的。

广义的跨域：

```java
1.) 资源跳转： A链接、重定向、表单提交
2.) 资源嵌入： <link>、<script>、<img>、
     <frame>等dom标签，
    还有样式中background:url()、
    @font-face()等文件外链
3.) 脚本请求： js发起的ajax请求、dom和js对象的跨域操作等
```

其实我们通常所说的跨域是狭义的，是由浏览器同源策略限制的一类请求场景。



# 什么是同源策略？

同源策略/SOP（Same origin policy）是一种约定，由Netscape公司1995年引入浏览器，它是浏览器最核心也最基本的安全功能，如果缺少了同源策略，浏览器很容易受到XSS、CSFR等攻击。所谓同源是指"协议+域名+端口"三者相同，即便两个不同的域名指向同一个ip地址，也非同源。

同源策略限制以下几种行为：

```java
1.) Cookie、LocalStorage 和 IndexDB 无法读取
2.) DOM 和 Js对象无法获得
3.) AJAX 请求不能发送
```

# 常见跨域场景

跨域，指的是浏览器不能执行其他网站的脚本。它是由浏览器的同源策略造成的，是浏览器施加的安全限制。

所谓同源是指，域名，协议，端口均相同，不明白没关系，举个栗子：

```java
http://www.123.com/index.html 调用 http://www.123.com/server.php （非跨域）

http://www.123.com/index.html 调用 http://www.456.com/server.php （主域名不同:123/456，跨域）

http://abc.123.com/index.html 调用 http://def.123.com/server.php （子域名不同:abc/def，跨域）

http://www.123.com:8080/index.html 调用 http://www.123.com:8081/server.php （端口不同:8080/8081，跨域）

http://www.123.com/index.html 调用 https://www.123.com/server.php （协议不同:http/https，跨域）
```

**请注意：localhost和127.0.0.1虽然都指向本机，但也属于跨域。**

**浏览器执行javascript脚本时，会检查这个脚本属于哪个页面，如果不是同源页面，就不会被执行。**

# 跨域解决方案

Access-Control-Allow-Origin是HTML5中定义的一种解决资源跨域的策略。他是通过服务器端返回带有Access-Control-Allow-Origin标识的Response header，用来解决资源的跨域权限问题。

# 单个应用解决跨域问题

转载地址：https://www.cnblogs.com/li-zhi-long/p/11465565.html

```java
@Configuration
public class CorsConfig extends WebMvcConfigurerAdapter {
    static final String[] ORIGINS = new String[]{"GET", "POST", "PUT", "DELETE"};

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                //可访问ip，ip最好从配置文件中获取，
                .allowedOrigins("*")
                .allowedMethods(ORIGINS)
                //.allowedHeaders("*")
                .exposedHeaders("access-control-allow-headers",
                        "access-control-allow-methods",
                        "access-control-allow-origin",
                        "access-control-max-age",
                        "X-Frame-Options",
                        "token",
                        "channel")
                .allowCredentials(true).maxAge(3600);
    }
}
```

或者可以使用Filter，或者在网关层

```java
@Bean
public CorsFilter corsFilter() {
    final UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    final CorsConfiguration config = new CorsConfiguration();
    config.setAllowCredentials(true); // 允许cookies跨域
    config.addAllowedOrigin("*");// 允许向该服务器提交请求的URI，*表示全部允许。。这里尽量限制来源域，比如http://xxxx:8080 ,以降低安全风险。。
    config.addAllowedHeader("*");// 允许访问的头信息,*表示全部
    config.setMaxAge(18000L);// 预检请求的缓存时间（秒），即在这个时间段里，对于相同的跨域请求不会再预检了
    config.addAllowedMethod("*");// 允许提交请求的方法，*表示全部允许，也可以单独设置GET、PUT等
    config.addAllowedMethod("HEAD");
    config.addAllowedMethod("GET");// 允许Get的请求方法
    config.addAllowedMethod("PUT");
    config.addAllowedMethod("POST");
    config.addAllowedMethod("DELETE");
    config.addAllowedMethod("PATCH");
    source.registerCorsConfiguration("/**", config);
    return new CorsFilter(source);
}
```



# 网关和单服务都做了跨域

如果在微服务环境中，网关层做了跨域问题解决，单个服务也做了跨域问题处理，这时就会出现*多次配置问题

![1438634-20190905135627356-266233029](/img/1438634-20190905135627356-266233029.png)

这时候需要在Zuul配置忽略头部信息

```yaml
zuul:
#需要忽略的头部信息，不在传播到其他服务
  sensitive-headers: Access-Control-Allow-Origin
  ignored-headers: Access-Control-Allow-Origin,H-APP-Id,Token,APPToken
```

