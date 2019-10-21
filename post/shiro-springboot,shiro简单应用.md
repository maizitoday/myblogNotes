---
title:       "shiro-springboot,shiro简单应用"
subtitle:    ""
description: ""
date:        2019-10-21
author:      "麦子"
image:       "https://c.pxhere.com/images/e4/46/bd237f6583029424694a0d16589b-1435053.jpg!d"
tags:        ["shiro","用户角色权限设计基本设计"]
categories:  ["Tech" ]
---

[TOC]

# 简述

实现SpringBoot和shiro的整合，不同的权限出现不同的页面的效果。 实现登录认证和授权处理。 

# pom.xml

```xml
<dependency>
  <groupId>org.apache.shiro</groupId>
  <artifactId>shiro-spring-boot-starter</artifactId>
  <version>1.4.1</version>
</dependency>

<dependency>
  <groupId>com.github.theborakompanioni</groupId>
  <artifactId>thymeleaf-extras-shiro</artifactId>
  <version>2.0.0</version>
</dependency>
```

# ShiroConfig包含相关基础组件

```java
@Configuration
public class ShiroConfig {

    @Bean
    public ShiroFilterFactoryBean shiroFilterFactory(@Qualifier("securityManager") DefaultWebSecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactory = new ShiroFilterFactoryBean();
        shiroFilterFactory.setSecurityManager(securityManager);

        /****
         * 
         * shiro内置过滤器，可以实现权限相关的拦截器 
         * 常用的过滤器 
         * anon: 无需认证（登录）可以访问 
         * authc: 必须认证才可以访问
         * user:如果使用rememberMe的功能可以直接访问 
         * perms: 该资源必须得到资源权限才可以访问 
         * role: 该资源必须得到角色权限才可以访问
         * 
         */

        Map<String, String> filtermap = new LinkedHashMap<String, String>();
        filtermap.put("/addUser", "perms[user:addUser]");
        filtermap.put("/listUser", "perms[user:listUser]");
        shiroFilterFactory.setFilterChainDefinitionMap(filtermap);

        /**
         * 修改返回登录页面地址
         */
        shiroFilterFactory.setLoginUrl("login");

        /***
         * 设置未授权页面
         */
        shiroFilterFactory.setUnauthorizedUrl("noAuthor");
        return shiroFilterFactory;
    }

    @Bean(name = "securityManager")
    public DefaultWebSecurityManager getDefaultWebSecurityManager(@Qualifier("userRealm") UserRealm userRealm) {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        securityManager.setRealm(userRealm);
        return securityManager;
    }

    @Bean(name = "userRealm")
    public UserRealm getRealm() {
        UserRealm userRealm = new UserRealm();
        return userRealm;
    }

    /**
     * 用于处理在网页中使用标签
     * @return
     */
    @Bean
    public ShiroDialect getDialect() {

        return new ShiroDialect();
    }

}
```

# 自定义UserRealm

```java
public class UserRealm extends AuthorizingRealm {

    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection arg0) {
        System.out.println("执行授权逻辑");
        SimpleAuthorizationInfo authorizationInfo = new SimpleAuthorizationInfo();
        Subject subject = SecurityUtils.getSubject();
        User user = (User) subject.getPrincipal();
        authorizationInfo.addStringPermission(user.getPermission());
        return authorizationInfo;
    }

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken arg0) throws AuthenticationException {
        System.out.println("执行认证逻辑");

        // 数据库中查询出来用户名和密码
        String addUsername = "1";
        String listUsername = "2";
        String password = "123";

        UsernamePasswordToken token = (UsernamePasswordToken) arg0;
        if (!token.getUsername().equals(addUsername) && !token.getUsername().equals(listUsername)) {
            return null; // 抛出 UnknownAccountException 异常
        }


        User user = new User();
        if (token.getUsername().equals("1")) {
            String permission = "user:addUser";
            user.setUsername(addUsername);
            user.setPassword(password);
            user.setPermission(permission);
        } else if (token.getUsername().equals("2")) {
            String permission = "user:listUser";
            user.setUsername(addUsername);
            user.setPassword(password);
            user.setPermission(permission);
        }
        // info 会和 token对象进行匹配密码，如果不对，抛出异常处理。
        SimpleAuthenticationInfo info = new SimpleAuthenticationInfo(user, password, "");
        return info;
    }

}
```

# 测试Controller

```java
@Controller
public class LoginController {

    @GetMapping(value = "index")
    public String showMsg(Model model) {
        model.addAttribute("index", "首页");
        return "index";
    }

    @GetMapping(value = "addUser")
    public String addUser() {

        return "/user/addUser";
    }

    @GetMapping(value = "listUser")
    public String listUser() {

        return "/user/listUser";
    }

    @GetMapping(value = "login")
    public String login() {

        return "/login";
    }

    @GetMapping(value = "noAuthor")
    public String unauthorizedUrl( Model model) {
        model.addAttribute("msg", "授权不通过");
        return "/noAuthor";
    }

    

    @GetMapping(value = "loginAction")
    public String loginAction(String username, String password, Model model) {
        Subject subject = SecurityUtils.getSubject();
        UsernamePasswordToken token = new UsernamePasswordToken(username, password);
        token.setRememberMe(true);
        try {
            subject.login(token);
            // 登录成功后， 需要认证的页面就可以登录进入了。 
            model.addAttribute("msg", "登录成功");
        } catch (UnknownAccountException e) {
            model.addAttribute("msg", "用户名不存在");
            return "/login";
        } catch (IncorrectCredentialsException e) {
            model.addAttribute("msg", "密码错误");
            return "/login";
        }
        return "/index";
    }
}
```

# 相关页面

## index.html

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="UTF-8">
    <title>Insert title here</title>
    <script type="text/javascript"></script>
</head>

<body>
    <h th:text="${msg}" style="color:red"></h><br /><br />


    <a href="login">登录</a><br /><br /><br />


    <div shiro:hasPermission="user:addUser">
        <a href="addUser">用户添加</a><br /><br /><br />
    </div>

    <div shiro:hasPermission="user:listUser">
        <a href="listUser">用户列表</a>
    </div>


</body>

</html>
```

## login.html

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="UTF-8">
    <title>Insert title here</title>
    <script type="text/javascript"></script>
</head>

<body>
    <div id="mydiv" align="center">
        <h>我是登录页面</h>

        <form id="myform" action="loginAction">
            用户名:<input type="text" name="username" /><br />
            密码:<input type="text" name="password" /><br />
            <input type="submit" value="提交" />
        </form>

        <h th:text="${msg}" style="color:red"></h>

    </div>

</body>

</html>
```

## noAuthor.html

```html
<!DOCTYPE html>
<html>

<head>
        <meta charset="UTF-8">
        <title>Insert title here</title>
        <script type="text/javascript"></script>
</head>

<body>
        <h th:text="${msg}" style="color:red"></h>
</body>

</html>
```

