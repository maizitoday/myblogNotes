---
title:       "简单实现IOC"
subtitle:    ""
description: ""
date:        2019-07-10
author:      "麦子"
image:       "https://img.zhaohuabing.com/in-post/2018-04-16-using-helm-to-deploy-to-kubernetes/buffalo.jpg"
tags:        ["java基础", "ioc"]
categories:  ["Tech" ]
---

[TOC]

# 实现的功能

简单实现IOC功能，模拟web请求找到对应的Controller和Controller里面对应的方法，然后反射调用方法。 

步骤分为以下几步：

1.  自动加载某一个包里面的所有class类，进行反射实例化。 
2.  定义自定义属性注解，进行自动注入到对应的Controller类。
3.  实现方法自定义注解和Controller，以及Controller里面的方法的绑定。
4.  最终实现模拟web一个请求，然后找到对应的Controller里面的方法，反射调用这个方法。 

# 自定义注解类

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyMethodAnnotation {
    String requestAction();
}
```

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyFieldAnnotation {
    
}
```

# 模拟service类

```java
public class MyService {

    public void handlerData() {
        System.out.println("service: 处理消息完成");
    }

}
```



# 模拟Controller类

```java
public class MyController {

    @MyFieldAnnotation
    public MyService myService;

    @MyMethodAnnotation(requestAction = "login.action")
    public void showMessage() {
        myService.handlerData();
        System.out.println("controller: 运行完成");
    }

}
```



# 反射实现包里面的class文件

```java
public static Map<Class<?>, Object> loadAllClass(String packageName) throws IOException {
        Map<Class<?>, Object> resultMap = new HashMap<>();
        String packageDirName = packageName.replace('.', '/');
        Enumeration<URL> dirs = Thread.currentThread().getContextClassLoader().getResources(packageDirName);
        while (dirs.hasMoreElements()) {
            URL url = dirs.nextElement();
            String protocol = url.getProtocol();
            if ("file".equals(protocol)) {
                String filePath = URLDecoder.decode(url.getFile(), "UTF-8");
                log.info("文件路径: " + filePath);

                File dir = new File(filePath);
                if (!dir.exists() || !dir.isDirectory()) {
                    log.info("文件不存在");
                    return null;
                }
                File[] dirfiles = dir.listFiles();
                Stream<File> stream = Stream.of(dirfiles);
                stream.forEach(file -> {
                    String className = file.getName().substring(0, file.getName().length() - 6);
                    Class<?> clazz = null;
                    try {
                        clazz = Class.forName(packageName + "." + className);
                    } catch (ClassNotFoundException e) {
                        log.info(packageName + "." + className + " -> 反射失败");
                        e.printStackTrace();
                    }
                    if (!clazz.isAnnotation()) {
                        try {
                            resultMap.put(clazz, clazz.newInstance());
                        } catch (InstantiationException | IllegalAccessException e) {
                            e.printStackTrace();
                        }
                    }

                });
            }
        }

        for (Class<?> var : resultMap.keySet()) {
            log.info(var + "  : " + resultMap.get(var));
        }
        return resultMap;
    }
```



# 属性注入

```java
 /**
     * 标有 MyFieldAnnotation的注解， 自动进行属性设置
     */
    public static List<Object> setField(Map<Class<?>, Object> resultMap) {
        List<Object> resultList = new ArrayList<>();
        for (Class<?> var : resultMap.keySet()) {
            Object beanInstance = resultMap.get(var);
            Field[] fields = var.getDeclaredFields(); // 获取所有的属性
            if (fields != null) {
                Stream.of(fields).forEach((field) -> {
                    if (field.isAnnotationPresent(MyFieldAnnotation.class)) {
                        log.info("拥有 MyFieldAnnotation 注解的类： " + beanInstance);

                        field.setAccessible(true); // 对私有字段的访问取消检查
                        Class<?> beanFidldClass = field.getType();
                        try {
                            field.set(beanInstance, resultMap.get(beanFidldClass));
                            resultList.add(beanInstance);
                        } catch (IllegalArgumentException | IllegalAccessException e) {
                            e.printStackTrace();
                        }
                    }
                });
            }
        }
        return resultList;
    }
```



# 方法注解和Controller绑定

```java
 /***
     * 设置 MyMethodAnnotation 和 Controller 的对应关系
     * 
     */
    public static Map<String, Map<Method, Object>> functionToController(Map<Class<?>, Object> resultMap) {
        Map<String, Map<Method, Object>> requestMapping = new HashMap<>();
        for (Class<?> var : resultMap.keySet()) {
            Object beanInstance = resultMap.get(var);
            Method[] methods = var.getDeclaredMethods();
            if (methods != null) {
                Stream.of(methods).forEach((method) -> {
                    if (method.isAnnotationPresent(MyMethodAnnotation.class)) {
                        log.info("拥有 MyMethodAnnotation 注解的类： " + beanInstance);
                        MyMethodAnnotation action = method.getAnnotation(MyMethodAnnotation.class);
                        String requestAction = action.requestAction();
                        Map<Method, Object> controllerToMethod = new HashMap<Method, Object>();
                        controllerToMethod.put(method, beanInstance);
                        requestMapping.put(requestAction, controllerToMethod);
                    }
                });
            }
        }
        for (String var : requestMapping.keySet()) {
            log.info(var + "  : " + requestMapping.get(var));
        }
        return requestMapping;
    }
```

# 模拟实现web请求进行方法调用

```java
  public static void main(String[] args)
            throws IOException, IllegalAccessException, IllegalArgumentException, InvocationTargetException {
        Map<Class<?>, Object> resultMap = IocHelper.loadAllClass("com.example.basedemo.reflectdemo.iocdemo");
        System.out.println();

        List<Object> resultList = setField(resultMap);
        System.out.println();

        Map<String, Map<Method, Object>> functionToController = functionToController(resultMap);
        System.out.println();

        // 模拟web请求，因为我们通过Tomcat，request.getMethod()方法可以知道你的具体的请求路径是什么
       //  这里就是一个简单的 login.action的服务请求。
        String loginAction = "login.action";
        Map<Method, Object> methodMap = (Map<Method, Object>) functionToController.get(loginAction);
        if (methodMap.size() == 1) {
            for (Method method : methodMap.keySet()) {
                MyController myController = (MyController) methodMap.get(method);
                method.invoke(myController, null);
            }
        }

    }
```

# 运行结果

```java
22:09:15.678 [main] INFO com.example.basedemo.reflectdemo.iocdemo.IocHelper - 文件路径: /vscode/javaBaseDemo/basedemo/target/classes/com/example/basedemo/reflectdemo/iocdemo
22:09:15.760 [main] INFO com.example.basedemo.reflectdemo.iocdemo.IocHelper - class com.example.basedemo.reflectdemo.iocdemo.MyController  : com.example.basedemo.reflectdemo.iocdemo.MyController@27abe2cd
22:09:15.760 [main] INFO com.example.basedemo.reflectdemo.iocdemo.IocHelper - class com.example.basedemo.reflectdemo.iocdemo.MyService  : com.example.basedemo.reflectdemo.iocdemo.MyService@5f5a92bb
22:09:15.760 [main] INFO com.example.basedemo.reflectdemo.iocdemo.IocHelper - class com.example.basedemo.reflectdemo.iocdemo.IocHelper  : com.example.basedemo.reflectdemo.iocdemo.IocHelper@6fdb1f78

22:09:15.772 [main] INFO com.example.basedemo.reflectdemo.iocdemo.IocHelper - 拥有 MyFieldAnnotation 注解的类： com.example.basedemo.reflectdemo.iocdemo.MyController@27abe2cd

22:09:15.776 [main] INFO com.example.basedemo.reflectdemo.iocdemo.IocHelper - 拥有 MyMethodAnnotation 注解的类： com.example.basedemo.reflectdemo.iocdemo.MyController@27abe2cd
22:09:15.776 [main] INFO com.example.basedemo.reflectdemo.iocdemo.IocHelper - login.action  : {public void com.example.basedemo.reflectdemo.iocdemo.MyController.showMessage()=com.example.basedemo.reflectdemo.iocdemo.MyController@27abe2cd}

service: 处理消息完成
controller: 运行完成

```

可以看到我们就是通过一个login.action的方法， 就找到了对应的这个Controller的里面的方法， 然后进行执行。

# 总结

上面我们实现了一个简单的IOC， 实现了，MyController类里面通过反射去设置MyController类里面的MyService属性，然后通过模拟web， 通过一个URL的请求去找到对应的Controller和对应的方法。最后调用的时候，进行反射来实现，方法的调用。 简单实现了一个IOC的注入功能。