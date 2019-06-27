---
title:       "json-web-token"
subtitle:    ""
description: ""
date:        2019-05-30
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["分布式系列", "权限安全", "前后端分离", "单点登录", "jwt", "数字签名"]
categories:  ["Tech" ]
---

[TOC]

**转载地址： https://blog.csdn.net/yangdiao127/article/details/70336467  数字签名解释**

**转载地址： https://www.cnblogs.com/cjsblog/p/9277677.html** 

# JSON WEB Token是什么

JSON Web Token (JWT)是一个**开放标准**(RFC 7519)，它定义了一种紧凑的、自包含的方式，用于作为JSON对象在各方之间安全地传输信息。该信息可以被验证和信任，**因为它是数字签名的。**

所谓的数字签名就是，用公钥和私钥的一种对称。

# JWT与Session的差异

相同点是，它们都是存储用户信息；然而，Session是在服务器端的，而JWT是在客户端的。Session方式存储用户信息的最大问题在于要占用大量服务器内存，增加服务器的开销。而JWT方式将用户状态分散到了客户端中，可以明显减轻服务端的内存压力。Session的状态是存储在服务器端，客户端只有session id；而Token的状态是存储在客户端。

# JWT的好处

身份认证在这种场景下，一旦用户完成了登陆，在接下来的每个请求中包含JWT，可以用来验证用户身份以及对路由，服务和资源的访问权限进行验证。由于它的开销非常小，可以轻松的在不同域名的系统中传递，所有目前在单点登录（SSO）中比较广泛的使用了该技术。 信息交换在通信的双方之间使用JWT对数据进行编码是一种非常安全的方式，由于它的信息是经过签名的，可以确保发送者发送的信息是没有经过伪造的。

## 无状态和可扩展性

Tokens存储在客户端。完全无状态，可扩展。我们的负载均衡器可以将用户传递到任意服务器，因为在任何地方都没有状态或会话信息。

## 安全

Token不是Cookie。（The token, not a cookie.）每次请求的时候Token都会被发送。而且，由于没有Cookie被发送，还有助于防止CSRF攻击。即使在你的实现中将token存储到客户端的Cookie中，**这个Cookie也只是一种存储机制，而非身份认证机制。**没有基于会话的信息可以操作，因为我们没有会话!  **应为我们还需要用私钥来进行解密才是认证。**

## 单点服务

单点登录是现在广泛使用的JWT的一个特性，因为它的开销很小，并且可以轻松地跨域使用。

## 前后端分离

前后端分离，在分布式系统中，我们处理跨域的问题。 

# 认证和授权

认证意味着证实某个用户是他所声明的那个人；授权意味决定一个身份确定的用户能够访问那些资源。

# JTW结构

JWT包含了使用.分隔的三部分

## Header 头部

header典型的由两部分组成：token的类型（“JWT”）和算法名称（比如：HMAC SHA256或者RSA等等）。

例如：

```json
{
  "alg":"HS256",
  "type":"JWT"
 }
```

然后，用Base64对这个JSON编码就得到了JWT的第一部分

## Payload 负载

JWT的第二部分是payload，它包含声明（要求）。声明是关于实体(通常是用户)和其他数据的声明。声明有三种类型: registered, public 和 private。

- Registered claims : 这里有一组预定义的声明，它们不是强制的，但是推荐。比如：iss (issuer), exp (expiration time), sub (subject), aud (audience)等。
- Public claims : 可以随意定义。
- Private claims : 用于在同意使用它们的各方之间共享信息，并且不是注册的或公开的声明。

下面是一个列子

```json
{
  "sub":"1234567890",
  "name":"John Doe",
  "admin":true
 }
```

对payload进行Base64编码就得到JWT的第二部分

**注意：不要在JWT的playload或者header中放置敏感信息，除非它们是加密的。** 

## Signature 签名

为了得到签名部分，你必须有编码过的header、编码过的payload、一个秘钥，签名算法是header中指定的那个，然对它们签名即可。 

如下： 

```java
HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
```

hash方法用到了一个secret，这个东西需要application server和authentication server双方都知道，相当于约好了同一把验证的钥匙，最终才好做认证。

**签名是用于验证消息在传递过程中有没有被更改，并且，对于使用私钥签名的token，它还可以验证JWT的发送方是否为它所称的发送方。保证了数据安全性，可以说就是一种认证处理。**



## 完整的一个JWT

查看官网：https://jwt.io/#debugger-io

![Xnip2019-06-06_13-47-10](/img/Xnip2019-06-06_13-47-10.png)

**注意： 发送JWT要用https，JWT本身不保证数据安全，需要认证机构CA发送秘钥这样的话才是安全的。** 

# session,token,JWT工作原理

**转载地址：https://blog.csdn.net/qq_40707682/article/details/88560565**

## session认证流程

![6-6-1](/img/6-6-1.png)

## 直接token认证流程

![6-6-2](/img/6-6-2.png)

## JWT认证流程

![6-6-3](/img/6-6-3.png)



# Springboot+JWT

## pom.xml

```xml
<dependency>
			<groupId>io.jsonwebtoken</groupId>
			<artifactId>jjwt</artifactId>
			<version>0.7.0</version>
</dependency>
```

## 加密解密

```java
@Data
@ToString
public class Employee {
    private String id;
    private String username;
    private String password;

    public Employee() {
        this.setId("testId");
        this.setUsername("testUsername");
        this.setPassword("testPassword");
    }
}

public static void main(String[] args) {
        System.out.println("开始测试jwt加密");

        String token = Jwts.builder()
                .setSubject(new Employee().toString())
                .setExpiration(new Date(System.currentTimeMillis() + 30 * 60 * 1000))
                .signWith(SignatureAlgorithm.HS512, "PrivateSecret") 
                .compact();
        System.out.println(token);

        System.out.println("开始测试jwt Subject 解密");
        String user = Jwts.parser()
                    .setSigningKey("PrivateSecret")
                    .parseClaimsJws(token)
                    .getBody()
                    .getSubject();
        System.out.println(user);

  
        System.out.println("显示所有的信息");
        Object object = Jwts.parser()
                    .setSigningKey("PrivateSecret")
                    .parseClaimsJws(token)
                    .getBody();
        System.out.println(object);  
    }
```



## 运行结果

```
开始测试jwt加密
eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJFbXBsb3llZShpZD10ZXN0SWQsIHVzZXJuYW1lPXRlc3RVc2VybmFtZSwgcGFzc3dvcmQ9dGVzdFBhc3N3b3JkKSIsImV4cCI6MTU1OTgwNjk0OX0.6g2n2upcRNiecmjhw5V9HiUs0fHeg7-YBCTtFIpNhvpAhTLZQ79n5qOc2SNpd4rNx15gozy0krDo-zKBSumrlQ
开始测试jwt Subject 解密
Employee(id=testId, username=testUsername, password=testPassword)
显示所有的信息
{sub=Employee(id=testId, username=testUsername, password=testPassword), exp=1559806949}
```

