---
title:       "springCloud系列-zuul网关"
subtitle:    ""
description: "gateway,网关,zuul,默认路由规则,与GateWay对比,跨域问题"
date:        2020-09-14
author:      "麦子"
image:       "https://zhaohuabing.com/img/2018-12-27-the-obstacles-to-put-istio-into-production/background.jpg"
tags:        ["springCloud"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：[http://c.biancheng.net/view/5380.html**](http://c.biancheng.net/view/5380.html)

**好文：https://cloud.tencent.com/developer/article/1009223**

# 简介

Zuul 是 Netflix OSS 中的一员，是一个基于 JVM 路由和服务端的负载均衡器。提供路由、监控、弹性、安全等方面的服务框架。Zuul 能够与 Eureka、Ribbon、Hystrix 等组件配合使用。

Zuul 的核心是过滤器，通过这些过滤器我们可以扩展出很多功能，比如：

**1）动态路由**

动态地将客户端的请求路由到后端不同的服务，做一些逻辑处理，比如聚合多个服务的数据返回。

**2）请求监控**

可以对整个系统的请求进行监控，记录详细的请求响应日志，可以实时统计出当前系统的访问量以及监控状态。

**3）认证鉴权**

对每一个访问的请求做认证，拒绝非法请求，保护好后端的服务。

**4）压力测试**

压力测试是一项很重要的工作，像一些电商公司需要模拟更多真实的用户并发量来保证重大活动时系统的稳定。通过 Zuul 可以动态地将请求转发到后端服务的集群中，还可以识别测试流量和真实流量，从而做一些特殊处理。

**5）灰度发布**

灰度发布可以保证整体系统的稳定，在初始灰度的时候就可以发现、调整问题，以保证其影响度。



# 路由配置详解

当 Zuul 集成 Eureka 之后，其实就可以为 Eureka 中所有的服务进行路由操作了，**默认的转发规则就是“API 网关地址+访问的服务名称+接口 URI”。**



## 1. 指定具体服务路由

我们可以为每一个服务都配置一个路由转发规则：

```java
zuul.routes.fsh-house.path=/api-house/**
```

上述代码将 fsh-house 服务的路由地址配置成了 api-house，也就是当需要访问 fsh-house 中的接口时，我们可以通过 api-house/house/hello 来进行。这其实就是将服务名称变成了我们自定义的名称。

有的时候服务名称太长了，放在 URL 中不太友好，我们希望它变得更友好一点，就可以这么去配置。这里的 api-house/** 后面一定要配置两个星号，两个星号表示可以转发任意层级的 URL，比如“/api-house/house/1”。如果只配置一个星号，那么就只能转发一级，比如“/api-house/house”。



## 2. 路由前缀

有的时候我们会想在 API 前面配置一个统一的前缀，比如像 http://c.biancheng.net/user/login 这样登录接口，如果想将其变成 http://c.biancheng.net/rest/user/login，即在每个接口前面加一个 rest，此时我们就可以通过 Zuul 中的配置来实现：

```java
zuul.prefix=/rest
```



## 3. 本地跳转

Zuul 的 API 路由还提供了本地跳转功能，通过 forward 就可以实现。

```java
zuul.routes.fsh-substitution.path=/api/**
zuul.routes.fsh-substitution.url=forward:/local
```

当我们想在访问 api/1 的时候会路由到本地的 local/1 上去，就可以参照上述代码实现。local 是本地接口需要我们自行添加，因此我们要建一个 Controller，代码如下所示。

```java
@RestController
public class LocalController {
    @GetMapping("/local/{id}")
    public String local(@PathVariable String id) {
        return id;
    }
}
```

# 过滤器

Zuul 可以实现很多高级的功能，比如限流、认证等。想要实现这些功能，必须要基于 Zuul 给我们提供的核心组件“过滤器”。



## 过滤器类型

Zuul 中的过滤器跟我们之前使用的 javax.servlet.Filter 不一样，javax.servlet.Filter 只有一种类型，可以通过配置 urlPatterns 来拦截对应的请求。而 Zuul 中的过滤器总共有 4 种类型，且每种类型都有对应的使用场景。

### 1）pre

可以在请求被路由之前调用。适用于身份认证的场景，认证通过后再继续执行下面的流程。

### 2）route

在路由请求时被调用。适用于灰度发布场景，在将要路由的时候可以做一些自定义的逻辑。

### 3）post

在 route 和 error 过滤器之后被调用。这种过滤器将请求路由到达具体的服务之后执行。适用于需要添加响应头，记录响应日志等应用场景。

### 4）error

处理请求时发生错误时被调用。在执行过程中发送错误时会进入 error 过滤器，可以用来统一记录错误信息。



## 请求生命周期

![Xnip2020-09-10_15-07-10](/img/Xnip2020-09-10_15-07-10.png)

通过上面的图可以清楚地知道整个执行的顺序，请求发过来首先到 pre 过滤器，再到 routing 过滤器，最后到 post 过滤器，任何一个过滤器有异常都会进入 error 过滤器。

通过 com.netflix.zuul.http.Zuul[Servlet](http://c.biancheng.net/servlet/) 也可以看出完整执行顺序，ZuulServlet 类似 [Spring](http://c.biancheng.net/spring/)-Mvc 的 DispatcherServlet，所有的 Request 都要经过 ZuulServlet 的处理。

ZuulServlet 源码如下所示：

```java
@Override
public void service(javax.servlet.ServletRequest servletRequest, javax.servlet.ServletResponse servletResponse)
        throws ServletException, IOException {
    try {
        init((HttpServletRequest) servletRequest, (HttpServletResponse) servletResponse);
        RequestContext context = RequestContext.getCurrentContext();
        context.setZuulEngineRan();
        try {
            preRoute();
        } catch (ZuulException e) {
            error(e);
            postRoute();
            return;
        }
        try {
            route();
        } catch (ZuulException e) {
            error(e);
            postRoute();
            return;
        }
        try {
            postRoute();
        } catch (ZuulException e) {
            error(e);
            return;
        }
    } catch (Throwable e) {
        error(new ZuulException(e, 500, "UNHANDLED_EXCEPTION_" + e.getClass().getName()));
    } finally {
        RequestContext.getCurrentContext().unset();
    }
}
```



## 使用过滤器

我们创建一个 pre 过滤器，来实现 IP 黑名单的过滤操作，代码如下所示。

```java
public class IpFilter extends ZuulFilter {

    // IP黑名单列表
    private List<String> blackIpList = Arrays.asList("127.0.0.1");

    public IpFilter() {
        super();
    }

    @Override
    public boolean shouldFilter() {
        return true
    }

    @Override
    public String filterType() {
        return "pre";
    }

    @Override
    public int filterOrder() {
        return 1;
    }

    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        String ip = IpUtils.getIpAddr(ctx.getRequest());
        // 在黑名单中禁用
        if (StringUtils.isNotBlank(ip) && blackIpList.contains(ip)) {

            ctx.setSendZuulResponse(false);
            ResponseData data = ResponseData.fail("非法请求 ", ResponseCode.NO_AUTH_CODE.getCode());
            ctx.setResponseBody(JsonUtils.toJson(data));
            ctx.getResponse().setContentType("application/json; charset=utf-8");
            return null;
        }
        return null;
    }
}
```

由代码可知，自定义过滤器需要继承 ZuulFilter，并且需要实现下面几个方法：

### 1）shouldFilte

是否执行该过滤器，true 为执行，false 为不执行，这个也可以利用配置中心来实现，达到动态的开启和关闭过滤器。

### 2）filterType

过滤器类型，可选值有 pre、route、post、error。

### 3）filterOrder

过滤器的执行顺序，数值越小，优先级越高。

### 4）run

执行自己的业务逻辑，本段代码中是通过判断请求的 IP 是否在黑名单中，决定是否进行拦截。blackIpList 字段是 IP 的黑名单，判断条件成立之后，通过设置 ctx.setSendZuulResponse（false），告诉 Zuul 不需要将当前请求转发到后端的服务了。通过 setResponseBody 返回数据给客户端。

### 实例化

过滤器定义完成之后我们需要配置过滤器才能生效，IP 过滤器配置代码如下所示。

```java
@Configuration
public class FilterConfig {

    @Bean
    public IpFilter ipFilter() {
        return new IpFilter();
    }
}
```



## 过滤器禁用

有的场景下，我们需要禁用过滤器，此时可以采取下面的两种方式来实现：

1. 利用 shouldFilter 方法中的 return false 让过滤器不再执行
2. 通过配置方式来禁用过滤器，格式为“zuul. 过滤器的类名.过滤器类型 .disable=true”。如果我们需要禁用“使用过滤器”部分中的 IpFilter，可以用下面的配置：

```properties
zuul.IpFilter.pre.disable=true
```



## 过滤器中传递数据

项目中往往会存在很多的过滤器，执行的顺序是根据 filterOrder 决定的，那么肯定有一些过滤器是在后面执行的，如果你有这样的需求：第一个过滤器需要告诉第二个过滤器一些信息，这个时候就涉及在过滤器中怎么去传递数据给后面的过滤器。

实现这种传值的方式笔者第一时间就想到了用 ThreadLocal，既然我们用了 Zuul，那么 Zuul 肯定有解决方案，比如可以通过 RequestContext 的 set 方法进行传递，RequestContext 的原理就是 ThreadLocal。

```java
RequestContext ctx = RequestContext.getCurrentContext();
ctx.set("msg", "你好吗");
```

后面的过滤就可以通过 RequestContext 的 get 方法来获取数据：

```java
RequestContext ctx = RequestContext.getCurrentContext();
ctx.get("msg");
```

上面我们说到 RequestContext 的原理就是 ThreadLocal，这不是笔者自己随便说的，而是笔者看过源码得出来的结论，下面请看源码，代码如下所示。

```java
protected static final ThreadLocal<? extends RequestContext> threadLocal = new ThreadLocal<RequestContext>() {
    @Override
    protected RequestContext initialValue() {
        try {
            return contextClass.newInstance();
        } catch (Throwable e) {
            throw new RuntimeException(e);
        }
    }
};

public static RequestContext getCurrentContext() {
    if (testContext != null)
        return testContext;

    RequestContext context = threadLocal.get();
    return context;
}
```

## 过滤器拦截请求

如果有多个过滤器的话，就算第一个拒绝了，**那么不会影响他往下传递**，我们大致知道了为什么所有的过滤器都会执行，解决这个问题的办法就是通过 shouldFilter 来处理，即在拦截之后**通过数据传递的方式**告诉下一个过滤器是否要执行。

增加一行数据传递的代码：

```java
ctx.set("isSuccess", false);
```

在 RequestContext 中设置一个值来标识是否成功，当为 true 的时候，后续的过滤器才执行，若为 false 则不执行。

利用这种方法，在后面的过滤器就需要用到这个值来决定自己此时是否需要执行，此时只需要在 shouldFilter 方法中加上如下所示的代码即可。

```java
public boolean shouldFilter() {
    RequestContext ctx = RequestContext.getCurrentContext();
    Object success = ctx.get("isSuccess");
    return success == null ? true : Boolean.parseBoolean(success.toString());
}
```



## 过滤器中异常处理

对于异常来说，无论在哪个地方都需要处理。过滤器中的异常主要发生在 run 方法中，可以用 try catch 来处理。Zuul 中也为我们提供了一个异常处理的过滤器，当过滤器在执行过程中发生异常，若没有被捕获到，就会进入 error 过滤器中。

我们可以定义一个 error 过滤器来记录异常信息，代码如下所示。

```java
public class ErrorFilter extends ZuulFilter {

    private Logger log = LoggerFactory.getLogger(ErrorFilter.class);

    @Override
    public String filterType() {
        return "error";
    }

    @Override
    public int filterOrder() {
        return 100;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        Throwable throwable = ctx.getThrowable();
        log.error("Filter Erroe : {}", throwable.getCause().getMessage());
        return null;
    }
}
```



# 与GateWay对比

1. 在微服务架构，如果使用了[Spring Cloud生态的基础组件](https://www.jianshu.com/p/d228d4a50a21)，则Spring Cloud Gateway相比而言更加具备优势，单从[流式编程](https://www.jianshu.com/p/b3f2b518c1b7)+支持异步上就足以让开发者选择它了，他可以单独脱离spring cloud 体系。 
2. 对于小型微服务架构或是复杂架构（不仅包括微服务应用还有其他非Spring Cloud服务节点），zuul也是一个不错的选择。

# 设置转发请求的请求头信息

系统和系统直接有时候需要传递 Token这样的数据

```java
@Component
public class RequestHeaderFilter extends ZuulFilter {
    @Override
    public String filterType() {
        return FilterConstants.PRE_TYPE;
    }

    @Override
    public int filterOrder() {
        return FilterConstants.PRE_DECORATION_FILTER_ORDER + 2;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() {
        RequestContext requestContext = RequestContext.getCurrentContext();
        requestContext.addZuulRequestHeader(ServletConstants.HEADER_TRACE_ID, ServletContextHolder.fetchTraceId());
        requestContext.addZuulRequestHeader(ServletConstants.HEADER_VERSION, ServletContextHolder.fetchVersion());
        requestContext.addZuulRequestHeader(CommonConstants.TOKEN, ServletContextHolder.getRequest().getHeader(CommonConstants.TOKEN));
        return null;
    }
}
```

**说明：ServletContextHolder 是对 RequestContextHolder 封装。**

1. RequestContextHolder：持有上下文的Request容器。

2. 通过RequestContextHolder的静态方法可以随时随地取到当前请求的request对象。



























