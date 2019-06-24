---
title:       "springboot系列-多数据源"
subtitle:    ""
description: ""
date:        2019-06-14
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["springboot系列", "多数据源"]
categories:  ["Tech" ]
---

[TOC]

实现多数据源的配置。

# 配置数据源

```java
@Configuration
@MapperScan(basePackages = "com.example.moredatademo.mysqlmapper", sqlSessionTemplateRef  = "mysqlSqlSessionTemplate")
public class MysqlDataSourceConfig {
    
    @Bean(name = "mySqlDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.mysqlone")
    @Primary
    public DataSource mySqlDataSource() {
        DataSource dataSource = DruidDataSourceBuilder.create().build();
        return dataSource;
    }


    @Bean(name = "mysqlSqlSessionFactory")
    @Primary /*此处必须在主数据库的数据源配置上加上@Primary*/
    public SqlSessionFactory mysqlSqlSessionFactory(@Qualifier("mySqlDataSource") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        /*加载mybatis全局配置文件*/
        // bean.setConfigLocation(new PathMatchingResourcePatternResolver().getResource("classpath:mybatis/mybatis-config.xml"));
        /*加载所有的mapper.xml映射文件*/
        bean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mybatis/mapper/mysql/*.xml"));
        return bean.getObject();
    }

    @Bean(name = "mysqlTransactionManager")
    @Primary
    public DataSourceTransactionManager mysqlTransactionManager(@Qualifier("mySqlDataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean(name = "mysqlSqlSessionTemplate")
    @Primary
    public SqlSessionTemplate mysqlSqlSessionTemplate(@Qualifier("mysqlSqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
        return new SqlSessionTemplate(sqlSessionFactory);
    }

}
```

另外一个同上，修改对应的名字就好。 

# application.yml

```yaml

spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource    
    #主数据源配置
    mysqlone:
      driver-class-name: com.mysql.jdbc.Driver
      url: jdbc:mysql://localhost:3306/work?useUnicode=true&characterEncoding=utf-8
      username: root
      password: 123456
    oracleone:
      driver-class-name: oracle.jdbc.driver.OracleDriver
      url: jdbc:oracle:thin:@localhost:49161:xe
      username: system
      password: oracle

logging:
  level:
    com.example.moredatademo.mysqlmapper: debug
```

# 其余相关代码

```java

public interface SchoolXmlMapper {
    public School getSchoolById(Integer id);

    public void updateById(HashMap<String,Object> map);
    
}
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.moredatademo.mysqlmapper.SchoolXmlMapper">
    
    <select id="getSchoolById" resultType="com.example.moredatademo.mysqlmapper.School">
         select * from school where id=#{id}
    </select>

   <update id="updateById" parameterType="Map">
       update school SET  name = #{name} , address = #{address} WHERE id = #{id}
   </update>

</mapper>
```

# 注意

@*Transactional*这个注解在多数据源中支持事物处理。 