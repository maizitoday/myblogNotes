---
title:       "mybatis系列-MyBatisPlus插件学习"
subtitle:    ""
description: ""
date:        2019-09-07
author:      "麦子"
image:       "https://c.pxhere.com/images/52/54/18c63f88716ecc0dd8dd729f47a3-1420031.jpg!d"
tags:        ["mybatis系列", "MyBatisPlus", "逆向工程"]
categories:  ["Tech" ]
---

[TOC]

**官网地址:<https://mybatis.plus/>**

# 特性

- **无侵入**：只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑
- **损耗小**：启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作
- **强大的 CRUD 操作**：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求
- **支持 Lambda 形式调用**：通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错
- **支持主键自动生成**：支持多达 4 种主键策略（内含分布式唯一 ID 生成器 - Sequence），可自由配置，完美解决主键问题
- **支持自定义全局通用操作**：支持全局通用方法注入（ Write once, use anywhere ）
- **内置代码生成器**：采用代码或者 Maven 插件可快速生成 Mapper 、 Model 、 Service 、 Controller 层代码，支持模板引擎，更有超多自定义配置等您来使用
- **内置分页插件**：基于 MyBatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通 List 查询
- **分页插件支持多种数据库**：支持 MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、Postgre、SQLServer2005、SQLServer 等多种数据库
- **内置性能分析插件**：可输出 Sql 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出慢查询
- **内置全局拦截插件**：提供全表 delete 、 update 操作智能分析阻断，也可自定义拦截规则，预防误操作



# 代码生成器

连接Mysql， 测试自动生成Mapper 、 Model 、 Service 、 Controller 层代码。

## pom.xml

```xml
<dependency>
  <groupId>com.baomidou</groupId>
  <artifactId>mybatis-plus-boot-starter</artifactId>
  <version>3.2.0</version>
</dependency>
<dependency>
  <groupId>com.baomidou</groupId>
  <artifactId>mybatis-plus-generator</artifactId>
  <version>3.2.0</version>
</dependency>
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <scope>runtime</scope>
</dependency>
<dependency>
  <groupId>org.freemarker</groupId>
  <artifactId>freemarker</artifactId>
  <version>2.3.29</version>
</dependency>
<dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
  <optional>true</optional>
</dependency>
<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>druid-spring-boot-starter</artifactId>
  <version>1.1.17</version>
</dependency>
```

## application.yml

```yml
server:
  port: 8888


spring:
  datasource:
    username: root
    password: root
    url: jdbc:mysql://localhost:32769/maizi?useUnicode=true&characterEncoding=utf-8&noDatetimeStringSync=true
    driver-class-name: com.mysql.cj.jdbc.Driver
    sql-script-encoding: utf-8
    type: com.alibaba.druid.pool.DruidDataSource   


logging:
  level:
    com: 
      example: 
        mybatisplus:
          user: debug

```



## CodeGenerator模板

```java
package com.example.mybatisplus;
// 演示例子，执行 main 方法控制台输入模块表名回车自动生成对应项目目录中
public class CodeGenerator {

    /**
     * <p>
     * 读取控制台内容
     * </p>
     */
    public static String scanner(String tip) {
        Scanner scanner = new Scanner(System.in);
        StringBuilder help = new StringBuilder();
        help.append("请输入" + tip + "：");
        System.out.println(help.toString());
        if (scanner.hasNext()) {
            String ipt = scanner.next();
            if (StringUtils.isNotEmpty(ipt)) {
                return ipt;
            }
        }
        throw new MybatisPlusException("请输入正确的" + tip + "！");
    }

    public static void main(String[] args) {
        // 代码生成器
        AutoGenerator mpg = new AutoGenerator();

        // 全局配置
        GlobalConfig gc = new GlobalConfig();
        String projectPath = System.getProperty("user.dir");
        gc.setOutputDir(projectPath + "/src/main/java");
        gc.setAuthor("maizi");
        gc.setOpen(false);
        // gc.setSwagger2(true); 实体属性 Swagger2 注解
        gc.setFileOverride(true); // 如果生成的不好，进行覆盖处理。
        // gc.setActiveRecord(true);
        gc.setEnableCache(false);// XML 二级缓存
        gc.setBaseResultMap(true);// XML ResultMap
        gc.setBaseColumnList(true);// XML columList
        // 自定义文件命名，注意 %s 会自动填充表实体属性！
        gc.setEntityName("%sEntity");
        // gc.setMapperName("%sMapper");
        // gc.setXmlName("%sMapper");
        // gc.setServiceName("I%sService");
        // gc.setServiceImplName("%sServiceImpl");
        // gc.setControllerName("%sController");
        mpg.setGlobalConfig(gc);

        // 数据源配置
        DataSourceConfig dsc = new DataSourceConfig();
        dsc.setUrl("jdbc:mysql://localhost:32769/maizi?useUnicode=true&characterEncoding=utf-8&noDatetimeStringSync=true");
        // dsc.setSchemaName("public");
        dsc.setDriverName("com.mysql.cj.jdbc.Driver");
        dsc.setUsername("root");
        dsc.setPassword("root");
        mpg.setDataSource(dsc);

        // 包配置
        PackageConfig pc = new PackageConfig();
        pc.setModuleName(scanner("模块名"));
        pc.setParent("com.example.mybatisplus");
        mpg.setPackageInfo(pc);

        // 自定义配置
        InjectionConfig cfg = new InjectionConfig() {
        @Override
        public void initMap() {
        // to do nothing
        }
        };

        // 如果模板引擎是 freemarker
        String templatePath = "/templates/mapper.xml.ftl";
        // 自定义输出配置
        List<FileOutConfig> focList = new ArrayList<>();
        // 自定义配置会被优先输出
        focList.add(new FileOutConfig(templatePath) {
        @Override
        public String outputFile(TableInfo tableInfo) {
        // 自定义输出文件名 ， 如果你 Entity 设置了前后缀、此处注意 xml 的名称会跟着发生变化！！
        return projectPath + "/src/main/resources/mapper/" + pc.getModuleName() + "/"
        + tableInfo.getEntityName() + "Mapper" + StringPool.DOT_XML;
        }
        });
        cfg.setFileOutConfigList(focList);
        mpg.setCfg(cfg);

        // 配置模板
        TemplateConfig templateConfig = new TemplateConfig();

        // 配置自定义输出模板
        // 指定自定义模板路径，注意不要带上.ftl/.vm, 会根据使用的模板引擎自动识别
        // templateConfig.setEntity("templates/entity2.java");
        // templateConfig.setService();
        // templateConfig.setController();

        templateConfig.setXml(null);
        mpg.setTemplate(templateConfig);

        // 策略配置
        StrategyConfig strategy = new StrategyConfig();
        strategy.setNaming(NamingStrategy.underline_to_camel);
        strategy.setColumnNaming(NamingStrategy.underline_to_camel);
        strategy.setSuperEntityClass("com.example.mybatisplus.common.BaseEntity");
        strategy.setEntityLombokModel(true);
        strategy.setRestControllerStyle(true);
        // 公共父类
        strategy.setSuperControllerClass("com.example.mybatisplus.common.BaseController");
        // 写于父类中的公共字段
        strategy.setSuperEntityColumns("id");
        strategy.setInclude(scanner("表名，多个英文逗号分割").split(","));
        strategy.setControllerMappingHyphenStyle(true);
        strategy.setTablePrefix(pc.getModuleName() + "_");
        mpg.setStrategy(strategy);
        mpg.setTemplateEngine(new FreemarkerTemplateEngine());
        mpg.execute();
    }

}

```



## 自动生成代码效果

### Entity

```java
@Data
@EqualsAndHashCode(callSuper = true)
@Accessors(chain = true)
@TableName("user")
public class UserEntity extends BaseEntity {

    private static final long serialVersionUID = 1L;

    /**
     * 姓名
     */
    private String name;

    /**
     * 年龄
     */
    private Integer age;

    /**
     * 邮箱
     */
    private String email;


}
```

### UserMapper

```java
public interface UserMapper extends BaseMapper<UserEntity> {

}

```

### IUserService

```java
public interface IUserService extends IService<UserEntity> {

}
```

### UserServiceImpl

```java
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, UserEntity> implements IUserService {

}
```

### UserController

```java
@RestController
@RequestMapping(value="/user/user")
public class UserController extends BaseController {


}
```

### UserEntityMapper.xml

```xml
 <mapper namespace="com.example.mybatisplus.user.mapper.UserMapper">

    <!-- 通用查询映射结果 -->
    <resultMap id="BaseResultMap" type="com.example.mybatisplus.user.entity.UserEntity">
    <result column="id" property="id" />
        <result column="name" property="name" />
        <result column="age" property="age" />
        <result column="email" property="email" />
    </resultMap>

    <!-- 通用查询结果列 -->
    <sql id="Base_Column_List">
        id,
        name, age, email
    </sql>

</mapper>

```



# Mapper CRUD 接口

对于一般的单表操作， 直接使用提供就好了， 如果是复杂一点的sql语句，我们还是使用xml的格式，这样维护和查看都更加的方便和清晰。 

## 实体结构

```java
@Data
@EqualsAndHashCode(callSuper = true)
@Accessors(chain = true)
@TableName("user")
public class UserEntity extends BaseEntity {

    private static final long serialVersionUID = 1L;

    // 确定主键是自动增长的
    @TableId(value = "id",type=IdType.AUTO)
    private Long  id; 

    /**
     * 姓名
     */
    private String name;

    /**
     * 年龄
     */
    private Integer age;

    /**
     * 邮箱
     */
    private String email;
}

```



## BaseMapper常用CRUD

```java
 // 新增
 UserEntity userEntity = new UserEntity();
 userEntity.setAge(10);
 userEntity.setName("麦子-10");
 userEntity.setEmail("maizi@sina.com-10");
 int count = userMapper.insert(userEntity);
 System.out.println("插入返回主键： "+userEntity.getId());
 
 // 查询
 UserEntity userEntity2 = userMapper.selectById(8);
 System.out.println("查询返回对象:"+userEntity2);

// 修改
UserEntity userEntity = new UserEntity();
userEntity.setAge(10);
userEntity.setName("麦子-100");
userEntity.setEmail("maizi@sina.com-100");
userEntity.setId(new Long(1));
int count = userMapper.updateById(userEntity);
System.out.println("updateById:"+count+"---"+userEntity.getName());

// 删除
int index = userMapper.deleteById(new Long(8));
System.out.println("index:"+index);


// 查询
List<UserEntity> users = userMapper.selectList(null);
users.forEach(System.out::println);
```



## IService常用CRUD

```java
// 新增
UserEntity userEntity = new UserEntity();
userEntity.setAge(100);
userEntity.setName("麦子-100");
userEntity.setEmail("maizi@sina.com-100");
boolean flag = userService.save(userEntity);
System.out.println("返回主键: "+userEntity.getId());

// 删除
boolean bloolean = userService.removeById(10);
System.out.println("删除成功否: "+bloolean);

// 查询
List<UserEntity> list = userService.list();
list.forEach(System.out::println);




```

# 条件构造器

主要是 Wrapper， AbstractWrapper，QueryWrapper，UpdateWrapper， 这几个对象可以像面向对象一样的拼凑SQL语句。 比如 and ， in，  or等。 

## QueryWrapper

```java
// 查询 
QueryWrapper<UserEntity> wrapper = new QueryWrapper<UserEntity>();
wrapper.eq("id", 9).select("name");
List<UserEntity> list = userService.list(wrapper);
list.forEach(System.out::println);
```

## UpdateWrapper

```java
UpdateWrapper<UserEntity>  updateWrapper = new UpdateWrapper<UserEntity> ();
updateWrapper.set("name", "小强---0001");
updateWrapper.set("age", 1);
updateWrapper.set("email", "0001@sina.com");
updateWrapper.eq("id", 1);
boolean flag = userService.update(null, updateWrapper);
```

可以看到update的两个参数可以各自组合条件。

```java
UpdateWrapper<UserEntity>  updateWrapper = new UpdateWrapper<UserEntity> ();
UserEntity userEntity = new UserEntity();
userEntity.setName("name") ;
userEntity.setEmail("email@sina.com");
userEntity.setAge(1000);
updateWrapper.eq("id", 1);
boolean flag = userService.update(userEntity, updateWrapper);
```



# 扩展BaseMapper方法

程序加载后， 自动加载定义全局的方法。 这个方法将在启动时候加入到BaseMapper中。 

<https://gitee.com/baomidou/mybatis-plus-samples/tree/master/mybatis-plus-sample-sql-injector>

# 分页插件

## 添加分页拦截器

```java
@Bean
public PaginationInterceptor paginationInterceptor() {
PaginationInterceptor paginationInterceptor = new PaginationInterceptor();
   // paginationInterceptor.setLimit(你的最大单页限制数量，默认 500 条，小于 0 如 -1 不受限制);
   return paginationInterceptor;
}
```

## BaseMapper分页

```java
Page<UserEntity> page = new Page<UserEntity>();
page.setSize(4);
page.setCurrent(1L);
Page<UserEntity> userEntityPage = (Page<UserEntity>)userMapper.selectPage(page, null);
userEntityPage.getRecords().forEach(System.out::println);
System.out.println(userEntityPage.getTotal()+"=="+userEntityPage.getPages());
```

## IService分页

```java
Page<UserEntity> page = new Page<UserEntity>();
page.setSize(4);
page.setCurrent(1L);
Page<UserEntity> userEntityPage  = (Page<UserEntity>)userService.page(page);
userEntityPage.getRecords().forEach(System.out::println);
```

## 自定义sql分页

- UserMapper.xml 等同于编写一个普通 list 查询，mybatis-plus 自动替你分页

```xml
<select id="queryAll" resultMap="BaseResultMap"> 
        select * from user u 
                      left join student  stu on  u.sid= stu.id 

</select>
```

```java
public interface UserMapper extends BaseMapper<UserEntity> {
    List<UserEntity> queryAll(Page<UserEntity> page);
}
```

```java
Page<UserEntity> page = new Page<UserEntity>();
page.setSize(4);
page.setCurrent(1L);
List<UserEntity> users = userMapper.queryAll(page);
users.forEach(System.out::println);
```

# Sequence主键

文档详细使用说明： <https://mybatis.plus/guide/sequence.html>



# 逻辑删除

## application.yml 

```yaml
mybatis-plus:
  global-config:
    db-config:
      logic-delete-value: 1 # 逻辑已删除值(默认为 1)
      logic-not-delete-value: 0 # 逻辑未删除值(默认为 0)
```

## 实体类

这个字段和数据库的标识对应就可以了。

```java
@TableLogic
private Integer deleted;
```

## 调用

```java
int count = userMapper.deleteById(1);
System.out.println(count);

boolean flag = userService.removeById(2);
System.out.println(flag);
```

## 显示SQL语句

效果: 使用mp自带方法删除和查找都会附带逻辑删除功能 (自己写的xml不会)

```properties
: ==>  Preparing: UPDATE user SET flag=1 WHERE id=? AND flag=0 
UserMapper.deleteById  : ==> Parameters: 2(Integer)
UserMapper.deleteById  : <==    Updates: 1
```

可以看到上面代码调用的事删除方法， 但是后面SQL执行的事update语句。 



# 乐观锁插件

innodb 是默认采用行锁， 这个插件也是相当于行锁。 如果实在要添加的话， 查看文档：

<https://mybatis.plus/guide/optimistic-locker-plugin.html>



# 动态数据源

实例说明文档： <https://qq343509740.gitee.io/2018/10/09/Spring%E5%85%A8%E5%AE%B6%E6%A1%B6/SpringBoo2.x/dynamic-datasource/dynamic-datasource%20%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8(%E4%B8%80)/#%E9%9B%86%E6%88%90Druid>

开发手册文档：<https://gitee.com/baomidou/dynamic-datasource-spring-boot-starter/wikis/pages>

## application.yml

```yml
spring:
  datasource:
    dynamic:
      primary: mysql #记得设置一个默认数据源
      datasource:
        mysql:
          username: root
          password: root
          url: jdbc:mysql://localhost:32769/maizi 
          driver-class-name: com.mysql.cj.jdbc.Driver
        oracle:
          username: SYSTEM
          password: oracle
          url: jdbc:oracle:thin:@localhost:1521:ORCL
          driver-class-name: oracle.jdbc.driver.OracleDriver
          
   
mybatis-plus:
  global-config:
    db-config:
      logic-delete-value: 1 # 逻辑已删除值(默认为 1)
      logic-not-delete-value: 0 # 逻辑未删除值(默认为 0)
  configuration:
    map-underscore-to-camel-case: true
```



## Application.java

DruidDataSourceAutoConfigure会注入一个DataSourceWrapper，其会在原生的spring.datasource下找url,username,password，所以要排除DruidDataSourceAutoConfigure 。

```java
@SpringBootApplication(exclude = DruidDataSourceAutoConfigure.class)
@MapperScan(value = "com.example.mybatisplus.user") // 扫描你
public class DemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}
}
```

## pom.xml

```xml
<dependency>
			<groupId>com.baomidou</groupId>
			<artifactId>dynamic-datasource-spring-boot-starter</artifactId>
			<version>2.5.6</version>
</dependency>
```

## mappper

@DS这个也可以放到方法上面。方法上面的优先类上面的 

```
@DS("oracle")
public interface StudentMapper extends BaseMapper<Student> {

}


@DS("mysql")
public interface UserMapper extends BaseMapper<UserEntity> {
 
}
```

## entity

```java
@Data
@Accessors(chain = true)
@TableName("stu")
public class Student {
    private int id;
    private String name;
    private String sex;
    private String age;
}


@Data
@EqualsAndHashCode(callSuper = true)
@Accessors(chain = true)
@TableName("user")
public class UserEntity extends BaseEntity {
    @TableId(value = "id",type=IdType.AUTO)
    private Long  id; 
    private String name;
    private Integer age;
    private String email;
    @TableLogic
    private Integer flag;
}
```

## Controller调用

```java
@Autowired 
private UserServiceImpl userService;

@Autowired 
private StudentMapper studentMapper;

userService.list().forEach(System.out::println);
studentMapper.selectList(null).forEach(System.out::println);
```

从这里就可以看到从两个数据源里面获取不同的数据了。

## 问题

多数据源的时候， 无法开启驼峰命名的设置。 暂时不知道怎么解决。 



