---
title:       "springboot系列-全局异常处理"
subtitle:    ""
description: ""
date:        2019-06-24
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["springboot系列", "全局异常"]
categories:  ["Tech" ]
---

# GlobalExceptionHandler

```java
package com.example.basedemo.globexception;
/*
 * @Description: 请输入....
 * @Author: 麦子
 * @Date: 2019-06-24 10:52:09
 * @LastEditTime: 2019-06-24 10:54:04
 * @LastEditors: 麦子
 */

import java.util.HashMap;
import java.util.Map;

import javax.servlet.http.HttpServletRequest;

import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.servlet.ModelAndView;

@ControllerAdvice
public class GlobalExceptionHandler {

    public static final String DEFAULT_ERROR_VIEW = "errorPrompt";
    /***
     * 页面错误
     * @param req
     * @param e
     * @return
     * @throws Exception
     */
    @ExceptionHandler(value = MyPageException.class)
    public ModelAndView businessExceptionHandler(HttpServletRequest req, MyPageException e){
        ModelAndView mav = new ModelAndView();
        mav.addObject("msg", e.getExceptionMsg());
        mav.addObject("code", 410);
        mav.setViewName(DEFAULT_ERROR_VIEW);
        return mav;
    }

    /***
     * JSON 格式错误
     * @param req
     * @param e
     * @return
     */
    @ExceptionHandler(value = MyJsonException.class)
    @ResponseBody
    public Map<String, Object> jsonExceptionHandler(HttpServletRequest req, MyJsonException e) {
        Map<String, Object> re = new HashMap<String, Object>();
        re.put("code",e.getCount());
        re.put("msg", e.getExceptionMsg());
        return re;
    }
    
}
```

# MyJsonException

```java

@Data
@NoArgsConstructor
@AllArgsConstructor
public class MyJsonException extends Exception {
    private String exceptionMsg;
    private int count;
}
```

# MyPageException

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class MyPageException  extends Exception{
    private String exceptionMsg;
    private int code;
}
```



# LoginService

```java
@Service
public class LoginService {

    public String showException() throws MyJsonException {
        throw  new MyJsonException("json错误异常",504);
    }
    
}
```



# LoginController

```java
@Controller
public class LoginController {


    @Autowired
    private LoginService loginService;

    @RequestMapping("loginView")
    public ModelAndView  loginView() throws MyPageException {
        throw  new MyPageException("page页面错误异常",404);
    }

    @RequestMapping("loginJson")
    @ResponseBody
    public String  loginJson() throws MyJsonException {
        return loginService.showException();
    }
}
```

# errorPrompt.ftl

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>错误页面</title>
</head>
<body>
<h1>${code}</h1>
<h1>${msg}</h1>
</body>
</html>
```



# 运行结果如下

![Xnip2019-06-24_12-04-32](/img/Xnip2019-06-24_12-04-32.png)

