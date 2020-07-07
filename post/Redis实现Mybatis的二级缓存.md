---
title:       "Redis实现Mybatis的二级缓存"
subtitle:    ""
description: "配置文件中开启二级缓存, 实现Mybatis的Cache接口,二级缓存的实用"
date:        2020-07-06
author:      "麦子"
image:       "https://img.zhaohuabing.com/in-post/2018-04-16-using-helm-to-deploy-to-kubernetes/buffalo.jpg"
tags:        ["redis", "mysql系列"]
categories:  ["Tech" ]
---

[TOC]

**转载：https://www.cnblogs.com/lykxqhh/p/5690941.html**

# Mybatis的缓存

通大多数ORM层框架一样，Mybatis自然也提供了对一级缓存和二级缓存的支持。一下是一级缓存和二级缓存的作用于和定义。

1. 一级缓存是SqlSession级别的缓存。在操作数据库时需要构造 sqlSession对象，在对象中有一个(内存区域)数据结构（HashMap）用于存储缓存数据。不同的sqlSession之间的缓存数据区域（HashMap）是互相不影响的。

2. 二级缓存是mapper级别的缓存，多个SqlSession去操作同一个Mapper的sql语句，多个SqlSession去操作数据库得到数据会存在二级缓存区域，多个SqlSession可以共用二级缓存，二级缓存是跨SqlSession的。

3. 一级缓存的作用域是同一个SqlSession，在同一个sqlSession中两次执行相同的sql语句，第一次执行完毕会将数据库中查询的数据写 到缓存（内存），第二次会从缓存中获取数据将不再从数据库查询，从而提高查询效率。当一个sqlSession结束后该sqlSession中的一级缓存 也就不存在了。Mybatis默认开启一级缓存。

4.  二级缓存是多个SqlSession共享的，其作用域是mapper的同一个namespace，不同的sqlSession两次执行相同 namespace下的sql语句且向sql中传递参数也相同即最终执行相同的sql语句，第一次执行完毕会将数据库中查询的数据写到缓存（内存），第二 次会从缓存中获取数据将不再从数据库查询，从而提高查询效率。Mybatis默认没有开启二级缓存需要在setting全局参数中配置开启二级缓存。

5. 一般的我们将Mybatis和Spring整合时，mybatis-spring包会自动分装sqlSession，而Spring通过动态代理 sqlSessionProxy使用一个模板方法封装了select()等操作，每一次select()查询都会自动先执行openSession()， 执行完close()以后调用close()方法，**相当于生成了一个新的session实例，所以我们无需手动的去关闭这个session()，当然也无 法使用mybatis的一级缓存，也就是说mybatis的一级缓存在spring中是没有作用的。**

**因此我们一般在项目中实现Mybatis的二级缓存，虽然Mybatis自带二级缓存功能，但是如果实在集群环境下，使用自带的二级缓存只是针对单个的节 点，所以我们采用分布式的二级缓存功能。一般的缓存NoSql数据库如redis，Mancache等，或者EhCache都可以实现，从而更好地服务 tomcat集群中ORM的查询。**



# Mybatis的二级缓存的实现

下面主要通过Redis实现Mybatis的二级缓存功能。



## 1、配置文件中开启二级缓存

```xml
<setting name="cacheEnabled" value="true"/>  
```

 默认二级缓存是开启的。



## 2.  实现Mybatis的Cache接口

Mybatis提供了第三方Cache实现的接口，我们自定义MybatisRedisCache实现Cache接口，代码如下：

```java
/**
 * 创建时间：2016年1月7日 上午11:40:00
 * 
 * Mybatis二级缓存实现类
 * 
 * @author andy
 * @version 2.2
 */
 
public class MybatisRedisCache implements Cache {
 
	private static final Logger LOG = Logger.getLogger(MybatisRedisCache.class); 
	
	private final ReadWriteLock readWriteLock = new ReentrantReadWriteLock(true);
	
	private RedisTemplate<Serializable, Serializable> redisTemplate =  (RedisTemplate<Serializable, Serializable>) SpringContextHolder.getBean("redisTemplate"); 
	
	private String id;
	
	private JdkSerializationRedisSerializer jdkSerializer = new JdkSerializationRedisSerializer();
	
	public MybatisRedisCache(final String id){
		if(id == null){
			throw new IllegalArgumentException("Cache instances require an ID");
		}
		LOG.info("Redis Cache id " + id);
		this.id = id;
	}
	
	@Override
	public String getId() {
		return this.id;
	}
 
	@Override
	public void putObject(Object key, Object value) {
		if(value != null){
			redisTemplate.opsForValue().set(key.toString(), jdkSerializer.serialize(value), 2, TimeUnit.DAYS);
		}
	}
 
	@Override
	public Object getObject(Object key) {
		try {
			if(key != null){
				Object obj = redisTemplate.opsForValue().get(key.toString());
				return jdkSerializer.deserialize((byte[])obj); 
			}
		} catch (Exception e) {
			LOG.error("redis ");
		}
		return null;
	}
 
	@Override
	public Object removeObject(Object key) {
		try {
			if(key != null){
				redisTemplate.expire(key.toString(), 1, TimeUnit.SECONDS);
			}
		} catch (Exception e) {
		}
		return null;
	}
 
	@Override
	public void clear() {
		//jedis nonsupport
	}
 
	@Override
	public int getSize() {
		Long size = redisTemplate.getMasterRedisTemplate().execute(new RedisCallback<Long>(){
			@Override
			public Long doInRedis(RedisConnection connection)
					throws DataAccessException {
				return connection.dbSize();
			}
		});
		return size.intValue();
	}
 
	@Override
	public ReadWriteLock getReadWriteLock() {
		return this.readWriteLock;
	}
	
}
```



## 3. 二级缓存的实用

我们需要将所有的实体类进行序列化，然后在Mapper中添加自定义cache功能。

```xml
   <cache
        type="org.andy.shop.cache.MybatisRedisCache"
        eviction="LRU"
        flushInterval="6000000"
        size="1024"
        readOnly="false"
	/>
```



## 4. Redis中的存储

redis会自动的将Sql+条件+Hash等当做key值，而将查询结果作为value，只有请求中的所有参数都符合，那么就会使用redis中的二级缓存。其查询结果如下：

