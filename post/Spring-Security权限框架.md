---
title:       "Spring-Security权限框架"
subtitle:    ""
description: "Security框架学习, 认证和授权，核心组件，自带防火墙，密码加盐，@PreAuthorize，动态授权"
date:        2020-11-21
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["springboot系列", "认证和授权", "Security框架学习"]
categories:  ["Tech" ]
---

[TOC]

 

**转载地址：http://www.spring4all.com/article/443**

**好文列子地址：https://blog.csdn.net/weixin_39792935/article/details/84541194?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.pc_relevant_is_cache&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.pc_relevant_is_cache**

# 认证流程

![Xnip2020-10-31_13-37-40.png](/img/Xnip2020-10-31_13-37-40.png)

# 授权流程

![Xnip2020-10-31_14-45-51.png](/img/Xnip2020-10-31_14-45-51.png)

# 流程图

![3424642-7418a70abdfc7287](/img/3424642-7418a70abdfc7287.jpg)



# 核心组件



## SecurityContextHolder

`SecurityContextHolder`用于存储安全上下文（security context）的信息。当前操作的用户是谁，该用户是否已经被认证，他拥有哪些角色权限…这些都被保存在SecurityContextHolder中。`SecurityContextHolder`默认使用`ThreadLocal` 策略来存储认证信息。看到`ThreadLocal` 也就意味着，这是一种与线程绑定的策略。Spring Security在用户登录时自动绑定认证信息到当前线程，在用户退出时，自动清除当前线程的认证信息。但这一切的前提，是你在web场景下使用Spring Security，而如果是Swing界面，Spring也提供了支持，`SecurityContextHolder`的策略则需要被替换，鉴于我的初衷是基于web来介绍Spring Security，所以这里以及后续，非web的相关的内容都一笔带过。



### 获取当前用户的信息

因为身份信息是与线程绑定的，所以可以在程序的任何地方使用静态方法获取用户信息。一个典型的获取当前登录用户的姓名的例子如下所示：

```java
Object principal = SecurityContextHolder.getContext().getAuthentication().getPrincipal();
if (principal instanceof UserDetails) {
    String username = ((UserDetails)principal).getUsername();
} else {
    String username = principal.toString();
}
```

getAuthentication()返回了认证信息，再次getPrincipal()返回了身份信息，UserDetails便是Spring对身份信息封装的一个接口。



## Authentication

先看看这个接口的源码长什么样：

```java
package org.springframework.security.core;// <1>

public interface Authentication extends Principal, Serializable { // <1>
    Collection<? extends GrantedAuthority> getAuthorities(); // <2>

    Object getCredentials();// <2>

    Object getDetails();// <2>

    Object getPrincipal();// <2>

    boolean isAuthenticated();// <2>

    void setAuthenticated(boolean var1) throws IllegalArgumentException;
}
```

1. Authentication是spring security包中的接口，直接继承自Principal类，而Principal是位于`java.security`包中的。可以见得，Authentication在spring security中是最高级别的身份/认证的抽象。
2. 由这个顶级接口，我们可以得到用户拥有的权限信息列表，密码，用户细节信息，用户身份信息，认证信息。

**提供基础认证数据接口**



## Spring Security是如何完成身份认证的

1.  用户名和密码被过滤器获取到，封装成`Authentication`,通常情况下是`UsernamePasswordAuthenticationToken`这个实现类
2.  AuthenticationManager` 身份管理器负责验证这个`Authentication
3. 认证成功后，`AuthenticationManager`身份管理器返回一个被填充满了信息的（包括上面提到的权限信息，身份信息，细节信息，但密码通常会被移除）`Authentication`实例。
4. `SecurityContextHolder`安全上下文容器将第3步填充了信息的`Authentication`，通过SecurityContextHolder.getContext().setAuthentication(…)方法，设置到其中。



## AuthenticationManager

初次接触Spring Security的朋友相信会被`AuthenticationManager`，`ProviderManager` ，`AuthenticationProvider` …这么多相似的Spring认证类搞得晕头转向，但只要稍微梳理一下就可以理解清楚它们的联系和设计者的用意。AuthenticationManager（接口）是认证相关的核心接口，也是发起认证的出发点，因为在实际需求中，我们可能会允许用户使用用户名+密码登录，同时允许用户使用邮箱+密码，手机号码+密码登录，甚至，可能允许用户使用指纹登录（还有这样的操作？没想到吧），所以说AuthenticationManager一般不直接认证，AuthenticationManager接口的常用实现类`ProviderManager` 内部会维护一个`List<AuthenticationProvider>`列表，存放多种认证方式，实际上这是委托者模式的应用（Delegate）。也就是说，核心的认证入口始终只有一个：AuthenticationManager，不同的认证方式：用户名+密码（UsernamePasswordAuthenticationToken），邮箱+密码，手机号码+密码登录则对应了三个AuthenticationProvider。这样一来四不四就好理解多了？熟悉shiro的朋友可以把AuthenticationProvider理解成Realm。在默认策略下，只需要通过一个AuthenticationProvider的认证，即可被认为是登录成功。



ProviderManager` 中的List<AuthenticationProvider>，会依照次序去认证，认证成功则立即返回，若认证失败则返回null，下一个AuthenticationProvider会继续尝试认证，如果所有认证器都无法认证成功，则`ProviderManager` 会抛出一个ProviderNotFoundException异常。

**进行数据认证接口**

**到这里，如果不纠结于AuthenticationProvider的实现细节以及安全相关的过滤器，认证相关的核心类其实都已经介绍完毕了：身份信息的存放容器SecurityContextHolder，身份信息的抽象Authentication，身份认证器AuthenticationManager及其认证流程。姑且在这里做一个分隔线。下面来介绍下AuthenticationProvider接口的具体实现。**



### DaoAuthenticationProvider

AuthenticationProvider最最最常用的一个实现便是DaoAuthenticationProvider。顾名思义，Dao正是数据访问层的缩写，也暗示了这个身份认证器的实现思路。

按照我们最直观的思路，怎么去认证一个用户呢？用户前台提交了用户名和密码，而数据库中保存了用户名和密码，认证便是负责比对同一个用户名，提交的密码和保存的密码是否相同便是了。在Spring Security中。提交的用户名和密码，被封装成了UsernamePasswordAuthenticationToken，而根据用户名加载用户的任务则是交给了UserDetailsService，在DaoAuthenticationProvider中，对应的方法便是retrieveUser，虽然有两个参数，但是retrieveUser只有第一个参数起主要作用，返回一个UserDetails。还需要完成UsernamePasswordAuthenticationToken和UserDetails密码的比对，这便是交给additionalAuthenticationChecks方法完成的，如果这个void方法没有抛异常，则认为比对成功。比对密码的过程，用到了PasswordEncoder和SaltSource，密码加密和盐的概念相信不用我赘述了，它们为保障安全而设计，都是比较基础的概念。

如果你已经被这些概念搞得晕头转向了，不妨这么理解DaoAuthenticationProvider：**它获取用户提交的用户名和密码，比对其正确性，如果正确，返回一个数据库中的用户信息（假设用户信息被保存在数据库中）。**



### UserDetails与UserDetailsService

UserDetailsService只负责从特定的地方（通常是数据库）加载用户信息，仅此而已，记住这一点，可以避免走很多弯路。UserDetailsService常见的实现类有JdbcDaoImpl，InMemoryUserDetailsManager，前者从数据库加载用户，后者从内存中加载用户，也可以自己实现UserDetailsService，通常这更加灵活。我们一般都是

```java
public class JwtUserDetailsServiceImpl implements UserDetailsService {
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // 查询数据库，填充数据，主要是填充 username 和 password，他主要是通过这两个值判断。
        // 同时你也可以添加扩展属性，保存值，成功后，用户数据都在这里面 
        return null;
    }
}
```



## WebSecurityConfiguration

WebSecurityConfiguration中完成了声明springSecurityFilterChain的作用，并且最终交给DelegatingFilterProxy这个代理类，负责拦截请求（注意DelegatingFilterProxy这个类不是spring security包中的，而是存在于web包中，spring使用了代理模式来实现安全过滤的解耦）。



## WebSecurityConfigurerAdapter

适配器模式在spring中被广泛的使用，在配置中使用Adapter的好处便是，我们可以选择性的配置想要修改的那一部分配置，而不用覆盖其他不相关的配置。WebSecurityConfigurerAdapter中我们可以选择自己想要修改的内容，来进行重写，而其提供了三个configure重载方法，是我们主要关心的

**分别是对AuthenticationManagerBuilder，WebSecurity，HttpSecurity进行个性化的配置。**



### HttpSecurity

**转载地址：https://www.jianshu.com/p/6f1b129442a1**

1. `HttpSecurity`最终可以得到一个`DefaultSecurityFilterChain`通过的是`build()`方法
2. `HttpSecurity`维护了一个过滤器的列表，这个过滤器的列表最终放入了`DefaultSecurityFilterChain`这个过滤器链中
3. `HttpSecurity`最终提供了很多的配置，然而所有的配置也都是为了处理维护我们的过滤器列表

**允许基于选择匹配在资源级配置基于网络的安全性**

```java
 http.authorizeRequests().antMatchers(HttpMethod.OPTIONS, "/**").permitAll().anyRequest()
                                .authenticated();
```



### WebSecurity

**转载地址: https://blog.csdn.net/andy_zhang2007/article/details/82111647**

`WebSecurity`是`Spring Security`的一个`SecurityBuilder`。它的任务是基于一组`WebSecurityConfigurer`构建出一个`Servlet Filter`,具体来讲就是构建一个`Spring Security`的`FilterChainProxy`实例。这个`FilterChainProxy`实现了`Filter`接口，也是通常我们所说的`Spring Security Filter Chain`,所构建的`FilterChainProxy`实例缺省会使用名称`springSecurityFilterChain`作为`bean`注册到容器，运行时处理`web`请求过程中会使用该`bean`进行安全控制。

每个`FilterChainProxy`包装了一个`HttpFirewall`和若干个`SecurityFilterChain`, 这里每个 `SecurityFilterChain`要么对应于一个要忽略安全控制的`URL`通配符(`RequestMatcher`)；要么对应于一个要进行安全控制的`URL`通配符(`HttpSecurity`)。

```java
助记公式 : 1 FilterChainProxy = 1 HttpFirewall + n SecurityFilterChain
```

**用于影响全局安全性(配置资源，设置调试模式，通过实现自定义防火墙定义拒绝请求)的配置设置。一般用于配置全局的某些通用事物，例如静态资源等**

```java
@Override
public void configure(WebSecurity web) throws Exception {
    web.httpFirewall(httpFirewall);
    web.ignoring().antMatchers(SerurityConstant.IGNORING_URL);
}
```



### HttpFirewall

**转载地址： https://blog.csdn.net/andy_zhang2007/article/details/90511688**

`StrictHttpFirewall`是`Spring Security Web`提供的一个`HTTP`防火墙(对应概念模型接口`HttpFirewall`)实现,该实现采用了严格模式，遇到任何可疑的请求，会通过抛出异常`RequestRejectedException`拒绝该请求。`StrictHttpFirewall`也是`Spring Security Web`在安全过滤器代理`FilterChainProxy`内置缺省使用的`HTTP`防火墙机制。

```java
@Override
public void configure(WebSecurity web) throws Exception {
    web.httpFirewall(httpFirewall);
    web.ignoring().antMatchers(SerurityConstant.IGNORING_URL);
}

/**
* @Description: 自定义防火墙
* @Date: 2020-11-19 10:45:02
* @param {*}
* @return {*}
* @LastEditors: Do not edit
*/
@Bean
public HttpFirewall httpFirewall() {
    StrictHttpFirewall firewall = new StrictHttpFirewall();
    firewall.setAllowSemicolon(true);
    firewall.setAllowUrlEncodedDoubleSlash(true);
    firewall.setAllowUrlEncodedPercent(true);
    firewall.setAllowBackSlash(true);
    firewall.setAllowUrlEncodedSlash(true);
    firewall.setAllowUrlEncodedPeriod(true);
    return firewall;
}

```



### AuthenticationManagerBuilder

用于通过允许AuthenticationProvider容易地添加来建立认证机制。也就是说用来记录账号，密码，角色信息。

```java
@Autowired
public void configureAuthentication(AuthenticationManagerBuilder authenticationManagerBuilder)
                        throws Exception {             authenticationManagerBuilder.userDetailsService(userDetailsService).passwordEncoder(passwordEncoder());
        }
```



## PasswordEncoder

密码进行加盐处理，我们也可以在注册用户的时候，对MD5的密码进行加盐处理，提高密码。

```java
/**
* 加密密码
*/
private void encryptPassword(UserEntity userEntity){
      String password = userEntity.getPassword();
      password = new BCryptPasswordEncoder().encode(password);
      userEntity.setPassword(password);
}
```

# 授权注解

```java
//开启全局方法调用权限验证
// 当@EnableGlobalMethodSecurity(prePostEnabled=true)的时候，@PreAuthorize @PostAuthorize才可以使用
@EnableGlobalMethodSecurity(prePostEnabled = true)
```

**@PreAuthorize**  方法之前调用， 我们一般都是用这个。

```java
@Service
public class UserServiceImpl implements UserService {
   @PreAuthorize("hasRole('ROLE_ADMIN')")
   @PreAuthorize("hasAnyRole('normal','admin')")
   public void addUser(User user) {
      System.out.println("addUser................" + user);
   }
   @PreAuthorize("hasRole('ROLE_USER') or hasRole('ROLE_ADMIN')")
   @PreAuthorize("hasRole('normal') AND hasRole('admin')")
   public User find(int id) {
      System.out.println("find user by id............." + id);
      return null;
   }
}

```



```java
public class UserServiceImpl implements UserService {
   /**
    * 限制只能查询Id小于10的用户
   */
   @PreAuthorize("#id<10")
   public User find(int id) {
      System.out.println("find user by id........." + id);
      return null;
   }
   /**
    * 限制只能查询自己的信息
    */
   @PreAuthorize("principal.username.equals(#username)")
   public User find(String username) {
      System.out.println("find user by username......" + username);
      return null;
   }
   /**
    * 限制只能新增用户名称为abc的用户
    */
   @PreAuthorize("#user.name.equals('abc')")
   public void add(User user) {
      System.out.println("addUser............" + user);
   }
}

```

**一般我们都是基于权限进行授权,  这样更加的灵活。**

```java
@PreAuthorize("hasAuthority('user:add')")
@PreAuthorize("hasAnyAuthority('user:add','user:del')")
public void addUser(User user) {
      System.out.println("addUser................" + user);
}
```

**总结：通过扫描 preAuthorize 注解， 查看我们封装的**

```java
 @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        QueryWrapper<UserEntity> queryWrapper = new QueryWrapper<UserEntity>();
        queryWrapper.eq("user_name", username);
        UserEntity userEntity = userServiceImpl.getOne(queryWrapper);
        JwtUser jwtUser = null;
        if (userEntity != null) {
            String password = passwordEncoder.encode(userEntity.getPassWord());
            // 在这个方法里面，进行对 GrantedAuthority 这个对象的填充，里面就是user:add等值
            List<GrantedAuthority> authorityList = getAuthorityList(userEntity);

            QueryWrapper<UserRoleEntity> roleQw = new QueryWrapper<UserRoleEntity>();
            roleQw.eq("fk_user_id", userEntity.getId()).select("fk_role_id", "fk_user_id");
            List<UserRoleEntity> userRoleList = userRoleServiceImpl.list(roleQw);
            jwtUser = new JwtUser(username, password, 0, authorityList, userEntity.getId(), userRoleList);
        } else {
            log.debug("账号不存在");
            handlerExceptionResolver.resolveException(request, response, null,
                    new MyException(ResultCodeEnmu.NO_USER.description, ResultCodeEnmu.NO_USER.code));
        }
        return jwtUser;
    }
```

 **是否有这个字符串，如果有的话就是权限ok。**



# 自定义异常抓取

如何抛出自定义异常

```java
import org.springframework.web.servlet.HandlerExceptionResolver; 

@Autowired
 private HandlerExceptionResolver handlerExceptionResolver;

@Override
public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
  handlerExceptionResolver.resolveException(request, response, null,
                new MyException(ResultCodeEnmu.NO_USER.description, ResultCodeEnmu.NO_USER.code));  
}
return jwtUser;
}
```

然后在全局异常就可以抓取这个异常

```java
 @ExceptionHandler(value = MyException.class)
    public Map<String, Object> myExceptionHandler(HttpServletRequest req, MyException e) {
        ResultDataUtil.ResponseDataFormatEnmu exceptionFormat = ResultDataUtil.ResponseDataFormatEnmu.RESPONSE_FORMAT_CUSTOM_EXCEPTION;
        exceptionFormat.setMsg(getTrace(e));
        exceptionFormat.setRequestUrl(req.getRequestURL().toString());
        exceptionFormat.setBusinessMsg(e.getExceptionMsg());
        exceptionFormat.setParamsObj(req.getParameterMap());
        HashMap<String, Object> resopnseData = exceptionFormat.getResultData();
        log.error("系统业务日志", getTrace(e));
        // e.printStackTrace();
        return resopnseData;
    }
```



## @PreAuthorize 注解的异常

@PreAuthorize 注解的异常，抛出AccessDeniedException异常，不会被accessDeniedHandler捕获，而是会被全局异常捕获。



# 动态权限修改示例

**说明：列子来源网上，忘记地址了。**

```java
@GetMapping("/vip/test")
@Secured("ROLE_VIP")         // 需要ROLE_VIP权限可访问
public String vipPath() {
   return "仅 ROLE_VIP 可看";
}
```

```java
@GetMapping("/vip")
public boolean updateToVIP() {
   // 得到当前的认证信息
   Authentication auth = SecurityContextHolder.getContext().getAuthentication();
   // 生成当前的所有授权
   List<GrantedAuthority> updatedAuthorities = new ArrayList<>(auth.getAuthorities());
   // 添加 ROLE_VIP 授权
   updatedAuthorities.add(new SimpleGrantedAuthority("ROLE_VIP"));
   // 生成新的认证信息
   Authentication newAuth = new UsernamePasswordAuthenticationToken(auth.getPrincipal(), auth.getCredentials(), updatedAuthorities);      
   // 重置认证信息
    SecurityContextHolder.getContext().setAuthentication(newAuth);
    return true;
}
```

假设当前你的权限只有 ROLE_USER。那么按照上面的代码：

**1**、直接访问 /vip/test 路径将会得到403的Response；

**2**、访问 /vip 获取 ROLE_VIP 授权，再访问 /vip/test 即可得到正确的Response。

# 自定义 Filter

**说明：列子来源网上，忘记地址了。**

自定义的 Filter 建议继承 GenericFilterBean，本文示例：

```java
public class BeforeLoginFilter extends GenericFilterBean {

  @Override
  public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
      System.out.println("This is a filter before UsernamePasswordAuthenticationFilter.");
      // 继续调用 Filter 链
      filterChain.doFilter(servletRequest, servletResponse);
  }
}
```

配置自定义 Filter 在 Spring Security 过滤器链中的位置

配置很简单，本文示例：

```java
protected void configure(HttpSecurity http) throws Exception {
      http
        .authorizeRequests()
        .antMatchers("/").permitAll()
        .antMatchers("/user/**").hasRole("USER")
        .and()
        .formLogin().loginPage("/login").defaultSuccessUrl("/user")
        .and()
        .logout().logoutUrl("/logout").logoutSuccessUrl("/login");
  
      // 在 UsernamePasswordAuthenticationFilter 前添加 BeforeLoginFilter
      http.addFilterBefore(new BeforeLoginFilter(),UsernamePasswordAuthenticationFilter.class);
      // 在 CsrfFilter 后添加 AfterCsrfFilter
      http.addFilterAfter(new AfterCsrfFilter(), CsrfFilter.class);
  }
```

说明：**HttpSecurity** 有三个常用方法来配置：

- addFilterBefore(Filter filter, Class<? extends Filter> beforeFilter) 在 beforeFilter 之前添加 filter
- addFilterAfter(Filter filter, Class<? extends Filter> afterFilter) 在 afterFilter 之后添加 filter
- addFilterAt(Filter filter, Class<? extends Filter> atFilter) 在 atFilter 相同位置添加 filter， 此 filter 不覆盖 filter

通过在不同 Filter 的 doFilter() 方法中加断点调试，可以判断哪个 filter 先执行，从而判断 filter 的执行顺序 。













