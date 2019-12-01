---
title:       "smart-doc接口文档自动生成"
subtitle:    ""
description: ""
date:        2019-11-01
author:      "麦子"
image:       "https://c.pxhere.com/photos/50/b0/tree_road_archway_symmetry_and_black_white-99031.jpg!d"
tags:        ["开发工具", "smart-doc接口文档自动生成"]
categories:  ["Tech" ]
---

**官网地址：https://my.oschina.net/u/1760791/blog/2250962**

**简单示例：https://segmentfault.com/a/1190000020548000**

# pom.xml

```xml
<dependency>
			<groupId>com.github.shalousun</groupId>
			<artifactId>smart-doc</artifactId>
			<version>1.7.0</version>
			<scope>test</scope>
</dependency>
```

# 实际代码

```java
import lombok.Data;

@Data
public class User {

    /**
     * 用户名
     */
    private String userName;

    /**
     * 昵称
     */
    private String nickName;

    /**
     * 用户地址
     */
    private String userAddress;

    /**
     * 用户年龄
     */
    private int userAge;

    /**
     * 手机号
     */
    private String phone;

    /**
     * 创建时间
     */
    private Long createTime;

    /**
     * ipv6
     */
    private String ipv6;

    /**
     * 固定电话
     */
    private String telephone;
}
```

# 生成文档

```java
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
class DemoApplicationTests {

	/**
	 * 包括设置请求头，缺失注释的字段批量在文档生成期使用定义好的注释
	 */
	@Test
	void contextLoads() {
		ApiConfig config = new ApiConfig();
		config.setServerUrl("http://localhost:8080");
		// true会严格要求注释，推荐设置true
		config.setStrict(true);
		// true会将文档合并导出到一个markdown
		// config.setAllInOne(true);
		// 生成html时加密文档名不暴露controller的名称
		config.setMd5EncryptedHtmlName(true);

		// 指定文档输出路径
		// @since 1.7 版本开始，选择生成静态html doc文档可使用该路径：DocGlobalConstants.HTML_DOC_OUT_PATH;
		config.setOutPath("/Users/maizi/Desktop");
		// @since 1.2,如果不配置该选项，则默认匹配全部的controller,
		// 如果需要配置有多个controller可以使用逗号隔开
		config.setPackageFilters("com.example.smart_doc_demo.doc.UserController");

		long start = System.currentTimeMillis();
		// 获取接口数据后自行处理
		ApiDocBuilder.builderControllersApi(config);// 此处使用HtmlApiDocBuilder，ApiDocBuilder提供markdown能力
		long end = System.currentTimeMillis();
		DateTimeUtil.printRunTime(end, start);
	}
}

```

# 优势

**可以看到就是用普通的java注释就可以生成规范的接口文档出来。** 