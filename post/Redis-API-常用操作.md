---
title:       "Redis-API-常用操作"
subtitle:    ""
description: "redis的pom导入，5种常用API的用法，类型使用场景介绍"
date:        2020-02-29
author:      "麦子"
image:       "https://c.pxhere.com/images/91/4d/7a1a53a9399ed55ee697daf983d1-1589459.jpg!d"
tags:        ["redis"]
categories:  ["Tech" ]
---

[TOC]



**转载地址：https://blog.csdn.net/weixin_43835717/article/details/92802040** 

# maven导入包

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
</dependency>
```

# Reactive的方式访问Redis 

本文主要大概介绍一下响应式/反应式编程方式访问 redis，不能解决很多生产问题，只是帮助大家对响应式编程有一个认识。

# 常用方法归类

```java
StringRedisTemplate.opsForValue().* //操作String字符串类型
StringRedisTemplate.delete(key/collection) //根据key/keys删除
StringRedisTemplate.opsForList().*  //操作List类型
StringRedisTemplate.opsForHash().*  //操作Hash类型
StringRedisTemplate.opsForSet().*  //操作set类型
StringRedisTemplate.opsForZSet().*  //操作有序set
```



## opsForValue(操作字符串)

```java
   /**
     * 新增一个字符串类型的值,key是键，value是值。
     *
     * set(K key, V value)
     */
    public void set() {
        // 存入永久数据
        stringRedisTemplate.opsForValue().set("test2", "1");
        // 也可以向redis里存入数据和设置缓存时间
        stringRedisTemplate.opsForValue().set("test1", "hello redis", 10, TimeUnit.SECONDS);
    }

    /**
     * 批量插入，key值存在会覆盖原值
     *
     * multiSet(Map<? extends K,? extends V> map)
     */
    public void multiSet() {
        HashMap<String, String> hashMap = new HashMap<>();
        hashMap.put("testMultiSet1", "value0");
        hashMap.put("testMultiSet2", "value1");
        stringRedisTemplate.opsForValue().multiSet(hashMap);
    }

    /**
     * 批量插入，如果里面的所有key都不存在，则全部插入，返回true，如果其中一个在redis中已存在，全不插入，返回false
     *
     * multiSetIfAbsent(Map<? extends K,? extends V> map)
     */
    public void mutiSetIfAbsent() {
        HashMap<String, String> hashMap = new HashMap<>(16);
        hashMap.put("testMultiSet3", "value3");
        hashMap.put("testMultiSet4", "value4");
        Boolean multiSetIfAbsent = stringRedisTemplate.opsForValue().multiSetIfAbsent(hashMap);
        System.out.println(multiSetIfAbsent);
    }

    /**
     * 如果不存在则插入，返回true为插入成功,false失败
     *
     * setIfAbsent(K key, V value)
     */
    public void setIfAbsent() {
        Boolean setIfAbsent = stringRedisTemplate.opsForValue().setIfAbsent("test", "hello redis");
        System.out.println(setIfAbsent);
    }

    /**
     * 获取值,key不存在返回null
     *
     * get(Object key)
     */
    public void get() {
        System.out.println(stringRedisTemplate.opsForValue().get("test"));
    }

    /**
     * 批量获取，key不存在返回null
     *
     * multiGet(Collection<K> keys)
     */
    public void getmulti() {
        List<String> multiGet = stringRedisTemplate.opsForValue().multiGet(Arrays.asList("test", "test2"));
        if (multiGet != null) {
            System.out.println(multiGet.toString());
        }
    }

    /**
     * 获取指定字符串的长度。
     *
     * size(K key)
     */
    public void getLength() {
        Long size = stringRedisTemplate.opsForValue().size("maizi");
        System.out.println(size);
    }

    /**
     * 在原有的值基础上新增字符串到末尾。返回的事这个字符的长度
     *
     * append(K key, String value)
     */
    public void append() {
        Integer append = stringRedisTemplate.opsForValue().append("maizi", "today");
        System.out.println(append);
    }

    /**
     * 获取原来key键对应的值并重新赋新值
     *
     * getAndSet(K key, V value)
     */
    public void getAndSet() {
        String andSet = stringRedisTemplate.opsForValue().getAndSet("maizi", "today");
        System.out.println(andSet);
    }

    /**
     * 获取指定key的值进行减1，如果value不是integer类型，会抛异常，如果key不存在会创建一个，默认value为0
     *
     * decrement(k key)
     */
    public void decrement() {
        stringRedisTemplate.opsForValue().decrement("test5");
        stringRedisTemplate.opsForValue().decrement("maizi");

    }

    /**
     * 获取指定key的值进行加1，如果value不是integer类型，会抛异常，如果key不存在会创建一个，默认value为0
     * 
     * increment(k key)
     */
    public void increment() {
        stringRedisTemplate.opsForValue().set("test5", "10");
        stringRedisTemplate.opsForValue().increment("test5");
    }

    /**
     * 删除指定key,成功返回true，否则false
     * 
     * delete(k key)
     */
    public void delete() {
        Boolean delete = stringRedisTemplate.opsForValue().getOperations().delete("maizi");
        System.out.println(delete);
    }

    /**
     * 删除多个key，返回删除key的个数
     * 
     * delete(k ...keys)
     */
    public void deleteMulti() {
        Long count = stringRedisTemplate.opsForValue().getOperations()
                .delete(Arrays.asList("test5", "test2", "testMultiSet3", "testMultiSet4"));
        System.out.println(count);
    }
```



## opsForList(操作集合)



```java
/**
     * 在变量左边添加元素值。如果key不存在会新建，添加成功返回添加后的总个数
     * 
     * leftPush(K key, V value)
     */
    public void leftPush() {
        Long leftPush = stringRedisTemplate.opsForList().leftPush("list", "a");
        System.out.println(leftPush);

    }

    /**
     * 向左边批量添加参数元素，如果key不存在会新建，添加成功返回添加后的总个数
     * 
     * leftPushAll(K key, V... values)
     * 
     * 注意他是往左边添加的，这个时候，list的下标就是最后进来的。
     */
    public void leftPushAll() {
        Long leftPushAll = stringRedisTemplate.opsForList().leftPushAll("list", "b", "c", "d");
        System.out.println(leftPushAll);
    }

    /**
     * 向集合最右边添加元素。如果key不存在会新建，添加成功返回添加后的总个数
     * 
     * rightPush(K key, V value)
     */
    public void rightPush() {
        Long rightPush = stringRedisTemplate.opsForList().rightPush("list", "maizi");
        System.out.println(rightPush);
    }

    /**
     * 如果存在集合则添加元素,不存在就不添加
     * 
     * leftPushIfPresent(K key, V value)
     */
    public void leftPushIfPresent() {
        Long leftPushIfPresent = stringRedisTemplate.opsForList().leftPushIfPresent("list", "today");
        System.out.println(leftPushIfPresent);
    }

    /**
     * 向右边批量添加元素。返回当前集合元素总个数
     * 
     * rightPushAll(K key, V... values)
     */
    public void rightPushAll() {
        Long rightPushAll = stringRedisTemplate.opsForList().rightPushAll("list", "today-right-0", "today-right-2",
                "today-right-3");
        System.out.println(rightPushAll);
    }

    /**
     * 向已存在的集合中添加元素。返回集合总元素个数
     * 
     * rightPushIfPresent(K key, V value)
     */
    public void rightPushIfPresent() {
        Long aLong = stringRedisTemplate.opsForList().rightPushIfPresent("list", "e");
        System.out.println(aLong);
    }

    /**
     * 获取集合长度
     * 
     * size(K key)
     */
    public void size() {
        Long size = stringRedisTemplate.opsForList().size("list");
        System.out.println(size);
    }

    /**
     * 移除集合中的左边第一个元素。返回删除的元素，如果元素为空，该集合会自动删除
     * 
     * leftPop(K key)
     */
    public void leftPop() {
        String leftPop = stringRedisTemplate.opsForList().leftPop("list");
        System.out.println(leftPop);
    }

    /**
     * 移除集合中左边的元素在等待的时间里，如果超过等待的时间仍没有元素则退出。 这个也是移除左边的第一个元素，只是有一个等待的时间限制。
     * 
     * leftPop(K key, long timeout, TimeUnit unit)
     */
    public void leftPopWait() {
        String leftPop = stringRedisTemplate.opsForList().leftPop("list", 10, TimeUnit.SECONDS);
        System.out.println(leftPop);
    }

    /**
     * 移除集合中右边的元素。返回删除的元素，如果元素为空，该集合会自动删除
     * 
     * rightPop(K key)
     */
    public void rightPop() {
        String pop = stringRedisTemplate.opsForList().rightPop("list");
        System.out.println(pop);
    }

    /**
     * 移除集合中右边的元素在等待的时间里，如果超过等待的时间仍没有元素则退出。
     * 
     * rightPop(K key, long timeout, TimeUnit unit)
     */
    public void rightPopWait() {
        String pop = stringRedisTemplate.opsForList().rightPop("list", 10, TimeUnit.SECONDS);
        System.out.println(pop);
    }

    /**
     * 移除第一个集合右边的一个元素，插入到第二个集合左边插入这个元素, 如果第二个元素不在，会自动创建。
     * 
     * rightPopAndLeftPush(K sourceKey, K destinationKey)
     */
    public void rightPopAndLeftPush() {
        String rightPopAndLeftPush = stringRedisTemplate.opsForList().rightPopAndLeftPush("list", "cba");
        System.out.println(rightPopAndLeftPush);
    }

    /**
     * 在集合的指定位置插入元素,如果指定位置已有元素，则覆盖，没有则新增，超过集合下标+n则会报错。
     * 
     * set(K key, long index, V value)
     */
    public void setForList() {
        stringRedisTemplate.opsForList().set("list", 0, "today");
    }

    /**
     * 从存储在键中的列表中删除等于值的元素的第一个计数事件。count> 0：删除等于从左到右移动的值的第一个元素； count<
     * 0：删除等于从右到左移动的值的第一个元素；count = 0：删除等于value的所有元素
     * 
     * remove(K key, long count, Object value)
     * 
     * count : 表示一次性删除几个 value这个值。
     */
    public void remove() {
        Long remove = stringRedisTemplate.opsForList().remove("list", 3, "today-right-3");
        System.out.println(remove);
    }

    /**
     * 截取集合元素长度，保留长度内的数据。 注意：他是包含 end这个下标的值的。
     * 
     * trim(K key, long start, long end)
     */
    public void trim() {
        stringRedisTemplate.opsForList().trim("list", 0, 2);
    }

    /**
     * 获取集合指定位置的值。
     * 
     * index(K key, long index)
     */
    public void index() {
        String index = stringRedisTemplate.opsForList().index("list", 0);
        System.out.println(index);
    }

    /**
     * 获取指定区间的值。
     * 
     * range(K key, long start, long end)
     */
    public void range() {
        List<String> range = stringRedisTemplate.opsForList().range("list", 0, 10);
        System.out.println(range);
    }

    /**
     * 删除指定集合,返回true删除成功
     * 
     * delete(K key)
     */
    public void deleteForList() {
        stringRedisTemplate.opsForList().getOperations().delete("list");
    }

```



## opsForHash(操作hashMap)

```java
    /**
     * 新增hashMap值
     * 
     * put(H key, HK hashKey, HV value)
     */
    public void put() {
        stringRedisTemplate.opsForHash().put("hash", "hash-key-01", "hash-value-01");
        stringRedisTemplate.opsForHash().put("hash", "hash-key-02", "hash-value-02");
    }

    /**
     * 以map集合的形式添加键值对
     * 
     * putAll(H key, Map<? extends HK,? extends HV> m)
     */
    public void putAll() {
        HashMap<String, String> hashMap = new HashMap<String, String>();
        hashMap.put("hash-key-03", "hash-value-03");
        hashMap.put("hash-key-04", "hash-value-04");
        stringRedisTemplate.opsForHash().putAll("hash", hashMap);
    }

    /**
     * 如果变量值存在，在变量中可以添加不存在的的键值对，如果变量不存在，则新增一个变量，同时将键值对添加到该变量。添加成功返回true否则返回false
     * 
     * putIfAbsent(H key, HK hashKey, HV value), 如果已经有存在的，无法被覆盖
     */
    public void putIfAbsent() {
        Boolean putIfAbsent = stringRedisTemplate.opsForHash().putIfAbsent("hash", "hash-key-01", "value-001");
        Boolean putIfAbsent2 = stringRedisTemplate.opsForHash().putIfAbsent("hash", "hash-key-05", "hash-value-05");

        System.out.println(putIfAbsent);
        System.out.println(putIfAbsent2);

    }

    /**
     * 获取指定变量中的hashMap值。 所有HashMap中的所有val的值集合 和 key的集合
     * 
     * values(H Key)
     */
    public void values() {
        List<Object> values = stringRedisTemplate.opsForHash().values("hash");
        System.out.println(values.toString());

        Set<Object> keys = stringRedisTemplate.opsForHash().keys("hash");
        System.out.println(keys.toString());
    }

    /**
     * 获取变量中的键值对。
     * 
     * entries(H key)
     */
    public void entries() {
        Map<Object, Object> entries = stringRedisTemplate.opsForHash().entries("hash");
        System.out.println(entries.toString());
    }

    /**
     * 获取变量中的指定map键是否有值,如果存在该map键则获取值，没有则返回null。
     * 
     * get(H key, Object hashKey)
     */
    public void getForMap() {
        Object object = stringRedisTemplate.opsForHash().get("hash", "hash-key-01");
        System.out.println(object.toString());
    }

    /**
     * 获取变量的长度
     * 
     * size(H key)
     */
    public void sizeForMap() {
        Long size = stringRedisTemplate.opsForHash().size("hash");
        System.out.println(size);
    }

    /**
     * 使变量中的键以long值的大小进行自增长。值必须为Integer类型,否则异常 , 注意： 需要把这个key对应的值设置为字符串数字
     * 
     * increment(H key, HK hashKey, long data)
     */
    public void incrementForMap() {
        // stringRedisTemplate.opsForHash().put("hash", "cba", "10");
        Long increment = stringRedisTemplate.opsForHash().increment("hash", "cba", 1);
        System.out.println(increment);
    }

    /**
     * 以集合的方式获取变量中的值。
     * 
     * multiGet(H key, Collection<HK> hashKeys)
     */
    public void multiGet() {
        List<Object> multiGet = stringRedisTemplate.opsForHash().multiGet("hash",
                Arrays.asList("hash-key-02", "hash-key-03", "hash-key-04"));
        System.out.println(multiGet.toString());
    }

    /**
     * 匹配获取键值对，ScanOptions.NONE为获取全部键对，ScanOptions.scanOptions().match("hash-key2").build()匹配获取键位map1的键值对,不能模糊匹配。
     * 
     * scan(H key, ScanOptions options)
     */
    public void scan() {
        // Cursor<Entry<Object, Object>> scan =
        // stringRedisTemplate.opsForHash().scan("hash", ScanOptions.NONE);
        Cursor<Entry<Object, Object>> scan = stringRedisTemplate.opsForHash().scan("hash",
                ScanOptions.scanOptions().match("cba").build());
        while (scan.hasNext()) {
            Entry<Object, Object> next = scan.next();
            System.out.println(next.getKey() + " : " + next.getValue());
        }
    }

    /**
     * 删除变量中的键值对，可以传入多个参数，删除多个键值对。返回删除成功数量
     * 
     * delete(H key, Object... hashKeys)
     */
    public void deleteForMap() {
        Long delete = stringRedisTemplate.opsForHash().delete("hash", "hash-key-01", "hash-key-02");
        System.out.println(delete);
    }
```



## opsForSet(操作set集合)

```java
 /**
     * 向变量中批量添加值。返回添加的数量
     *
     * add(K key, V... values)
     */
    public void add() {
        Long add = stringRedisTemplate.opsForSet().add("set", "李白", "张三", "李四", "张三");
        System.out.println(add);
    }

    /**
     * 获取变量的值
     *
     * members(K key)
     */
    public void membered() {
        Set<String> members = stringRedisTemplate.opsForSet().members("set");
        System.out.println(members);
    }

    /**
     * 获取变量中值得长度
     *
     * size(k key)
     */
    public void sizeForSet() {
        Long size = stringRedisTemplate.opsForSet().size("set");
        System.out.println(size);
    }

    /**
     * 随机获取变量中的某个元素
     *
     * randomMember(k key)
     */
    public void randomRembers() {
        String randomMember = stringRedisTemplate.opsForSet().randomMember("set");
        System.out.println(randomMember);

        List<String> randomMembers = stringRedisTemplate.opsForSet().randomMembers("set", 2);
        System.out.println(randomMembers);
    }

    /**
     * 检查给定的元素是否在变量中,true为存在
     *
     * isMember(k key, object value)
     */
    public void isMember() {
        Boolean member = stringRedisTemplate.opsForSet().isMember("set", "李白");
        System.out.println(member);
    }

    /**
     * 转义变量的元素值到另一个变量中
     *
     * move(k key, v value, k targetKey)
     */
    public void move() {
        Boolean move = stringRedisTemplate.opsForSet().move("set", "张三", "set2");
        System.out.println(move);
    }

    /**
     * 弹出变量中的元素。当元素全部弹完,变量也会删除, 每次弹出第一个
     *
     * pop(k key)
     */
    public void pop() {
        String pop = stringRedisTemplate.opsForSet().pop("set");
        System.out.println(pop);
    }

    /**
     * 批量删除变量中的元素,返回删除的数量
     *
     * remove(k key, v ...values)
     */
    public void removeForSet() {
        Long remove = stringRedisTemplate.opsForSet().remove("set", "1", "2", "3");
        System.out.println(remove);
    }

    /**
     * 匹配获取键值对，ScanOptions.NONE为获取全部键值对；ScanOptions.scanOptions().match("C").build()匹配获取键位map1的键值对,不能模糊匹配。
     *
     * scan(K key, ScanOptions options)
     */
    public void scanForSet() {
        Cursor<String> scan = stringRedisTemplate.opsForSet().scan("set", ScanOptions.NONE);
        while (scan.hasNext()) {
            String next = scan.next();
            System.out.println(next);
        }
    }

    /**
     * 通过集合求差值。
     *
     * difference(k key, k otherKey)
     */
    public void difference() {
        Set<String> difference = stringRedisTemplate.opsForSet().difference("set", "set2");
        System.out.println(difference);
    }

    /**
     * 将求出来的差值元素保存
     *
     * differenceAndStore(K key, K otherKey, K targetKey)
     */
    public void differenceAndStore() {
        Long differenceAndStore = stringRedisTemplate.opsForSet().differenceAndStore("set", "set2", "set3");
        System.out.println(differenceAndStore);

    }

    /**
     * 获取去重的随机元素
     *
     * distinctRandomMembers(K key, long count)
     */
    public void distinctRandomMembers() {
        Set<String> distinctRandomMembers = stringRedisTemplate.opsForSet().distinctRandomMembers("set", 2);
        System.out.println(distinctRandomMembers);
    }

    /**
     * 获取两个变量中的交集
     *
     * intersect(K key, K otherKey)
     */
    public void intersect() {
        Set<String> intersect = stringRedisTemplate.opsForSet().intersect("set", "set2");
        System.out.println(intersect);
    }

    /**
     * 获取2个变量交集后保存到最后一个变量上。
     *
     * intersectAndStore(K key, K otherKey, K targetKey)
     */
    public void intersectAndStore() {
        Long intersectAndStore = stringRedisTemplate.opsForSet().intersectAndStore("set", "set2", "set4");
        System.out.println(intersectAndStore);
    }

    /**
     * 获取两个变量的合集
     *
     * union(K key, K otherKey)
     */
    public void union() {
        Set<String> union = stringRedisTemplate.opsForSet().union("set", "set2");
        System.out.println(union);
    }

    /**
     * 获取两个变量合集后保存到另一个变量中
     *
     * unionAndStore(K key, K otherKey, K targetKey)
     */
    public void unionAndStore() {
        Long unionAndStore = stringRedisTemplate.opsForSet().unionAndStore("set", "set2", "set7");
        System.out.println(unionAndStore);
    }

```



## opsForZset(操作有序set)

```java
/**
     * 添加元素到变量中同时指定元素的分值。
     *
     * add(K key, V value, double score)
     */
    public void addForZSet() {
        Boolean add = stringRedisTemplate.opsForZSet().add("zset", "110", 1);
        System.out.println(add);

        Boolean add2 = stringRedisTemplate.opsForZSet().add("zset", "120", 1.5);
        System.out.println(add2);
    }

    /**
     * 通过TypedTuple方式新增数据。
     *
     * add(K key, Set<ZSetOperations.TypedTuple<V>> tuples)
     */
    public void addByTypedTuple() {
        ZSetOperations.TypedTuple<String> typedTuple1 = new DefaultTypedTuple<>("E", 2.0);
        ZSetOperations.TypedTuple<String> typedTuple2 = new DefaultTypedTuple<>("F", 3.0);
        ZSetOperations.TypedTuple<String> typedTuple3 = new DefaultTypedTuple<>("G", 5.0);
        Set<ZSetOperations.TypedTuple<String>> typedTupleSet = new HashSet<>();
        typedTupleSet.add(typedTuple1);
        typedTupleSet.add(typedTuple2);
        typedTupleSet.add(typedTuple3);
        Long zset = stringRedisTemplate.opsForZSet().add("zset", typedTupleSet);
        System.out.println(zset);
    }

    /**
     * 获取指定区间的元素
     *
     * range(k key, long start, long end)，注意这个顺序，是从后面到前面来的。
     */
    public void rangeForZSet() {
        Set<String> zset = stringRedisTemplate.opsForZSet().range("zset", 0, 3);
        System.out.println(zset);
    }

    /**
     * 用于获取满足非score的排序取值。这个排序只有在有相同分数的情况下才能使用，如果有不同的分数则返回值不确定。
     *
     * rangeByLex(K key, RedisZSetCommands.Range range)
     */
    public void rangeByLex() {
        Set<String> rangeByLex = stringRedisTemplate.opsForZSet().rangeByLex("zset",
                RedisZSetCommands.Range.range().lt("E"));
        System.out.println(rangeByLex);
    }

    /**
     * 用于获取满足非score的设置 下标开始的长度排序取值。
     *
     * rangeByLex(k key, range range, limit limit)
     */
    public void rangeByLexAndLimit() {
        Set<String> zset = stringRedisTemplate.opsForZSet().rangeByLex("zset", RedisZSetCommands.Range.range().lt("E"),
                RedisZSetCommands.Limit.limit().offset(1).count(2));
        System.out.println(zset);
    }

    /**
     * 根据设置的score获取区间值。
     *
     * rangeByScore(K key, double min, double max)
     */
    public void rangeByScore() {
        Set<String> zset = stringRedisTemplate.opsForZSet().rangeByScore("zset", 1, 3);
        System.out.println(zset);
    }

    /**
     * 获取RedisZSetCommands.Tuples的区间值。
     *
     * rangeWithScores(K key, long start, long end)
     */
    public void rangeWithScores() {
        Set<ZSetOperations.TypedTuple<String>> zset = stringRedisTemplate.opsForZSet().rangeWithScores("zset", 1, 3);
        assert zset != null;
        for (ZSetOperations.TypedTuple<String> next : zset) {
            String value = next.getValue();
            Double score = next.getScore();
            System.out.println(value + "-->" + score);
        }
    }

    /**
     * 获取区间值的个数。
     *
     * count(k key, double min, double max)
     */
    public void count() {
        Long zset = stringRedisTemplate.opsForZSet().count("zset", 1, 3);
        System.out.println(zset);
    }

    /**
     * 获取变量中指定元素的索引,下标开始为0
     *
     * rank(k key, object o)
     */
    public void rank() {
        Long rank = stringRedisTemplate.opsForZSet().rank("zset", "G");
        System.out.println(rank);
    }

    /**
     * 匹配获取键值对，ScanOptions.NONE为获取全部键值对；ScanOptions.scanOptions().match("C").build()匹配获取键位map1的键值对,不能模糊匹配。
     *
     * scan(K key, ScanOptions options)
     */
    public void scanForZset() {
        Cursor<ZSetOperations.TypedTuple<String>> zset = stringRedisTemplate.opsForZSet().scan("zset",
                ScanOptions.NONE);
        while (zset.hasNext()) {
            ZSetOperations.TypedTuple<String> next = zset.next();
            System.out.println(next.getValue() + "-->" + next.getScore());
        }
    }

    /**
     * 获取指定元素的分值
     *
     * score(k key, object o)
     */
    public void score() {
        Double score = stringRedisTemplate.opsForZSet().score("zset", "a");
        System.out.println(score);
    }

    /**
     * 获取变量中元素的个数
     *
     * zCard(k key)
     */
    public void zCard() {
        Long zset = stringRedisTemplate.opsForZSet().zCard("zset");
        System.out.println(zset);
    }

    /**
     * 修改变量中元素的分值, 每次递增 + 2 ，在这里
     *
     * incrementScore(K key, V value, double delta)
     */
    public void incrementScore() {
        Double score = stringRedisTemplate.opsForZSet().incrementScore("zset", "a", 2);
        System.out.println(score);
    }

    /**
     * 索引倒序排列指定区间的元素， 注意： 这是索引倒序
     *
     * reverseRange(K key, long start, long end)
     */
    public void reverseRange() {
        Set<String> zset = stringRedisTemplate.opsForZSet().reverseRange("zset", 1, 3);
        System.out.println(zset);
    }

    /**
     * 倒序排列指定分值区间的元素
     *
     * reverseRangeByScore(K key, double min, double max)
     */
    public void reverseRangeByScore() {
        Set<String> zset = stringRedisTemplate.opsForZSet().reverseRangeByScore("zset", 1, 3);
        System.out.println(zset);
    }

    /**
     * 倒序排序获取RedisZSetCommands.Tuples的分值区间值
     *
     * reverseRangeByScore(K key, double min, double max, long offset, long count)
     */
    public void reverseRangeByScoreLength() {
        Set<String> zset = stringRedisTemplate.opsForZSet().reverseRangeByScore("zset", 1, 3, 1, 2);
        System.out.println(zset);
    }

    /**
     * 倒序排序获取RedisZSetCommands.Tuples的分值区间值。
     *
     * reverseRangeByScoreWithScores(K key, double min, double max)
     */
    public void reverseRangeByScoreWithScores() {
        Set<ZSetOperations.TypedTuple<String>> zset = stringRedisTemplate.opsForZSet()
                .reverseRangeByScoreWithScores("zset", 1, 5);
        assert zset != null;
        zset.iterator().forEachRemaining(e -> System.out.println(e.getValue() + "--->" + e.getScore()));
    }

    /**
     * 获取倒序排列的索引值
     *
     * reverseRank(k key, object o)
     */
    public void reverseRank() {
        Long aLong = stringRedisTemplate.opsForZSet().reverseRank("zset", "G");
        System.out.println(aLong);
    }

    /**
     * 获取2个变量的交集存放到第3个变量里面。
     *
     * intersectAndStore(K key, K otherKey, K destKey)
     */
    public void intersectAndStoreForZSet() {
        Long aLong = stringRedisTemplate.opsForZSet().intersectAndStore("zset", "zset2", "zset3");
        System.out.println(aLong);
    }

    /**
     * 获取2个变量的合集存放到第3个变量里面。 返回操作的数量
     *
     * unionAndStore(K key, K otherKey, K destKey)
     */
    public void unionAndStoreForZSet() {
        Long aLong = stringRedisTemplate.opsForZSet().unionAndStore("zset", "zset2", "zset3");
        System.out.println(aLong);
    }

    /**
     * 批量移除元素根据元素值。返回删除的元素数量
     *
     * remove(K key, Object... values)
     */
    public void removeForZSet() {
        Long remove = stringRedisTemplate.opsForZSet().remove("zset", "a", "b");
        System.out.println(remove);
    }

    /**
     * 根据分值移除区间元素。返回删除的数量
     *
     * removeRangeByScore(k key, double min, double max)
     */
    public void removeRangeByScore() {
        Long zset = stringRedisTemplate.opsForZSet().removeRangeByScore("zset", 1, 3);
        System.out.println(zset);
    }

    /**
     * 根据索引值移除区间元素。返回移除的元素集合
     *
     * removeRange(K key, long start, long end)
     */
    public void removeRange() {
        Set<String> zset = stringRedisTemplate.opsForZSet().reverseRange("zset", 0, 4);
        System.out.println(zset);
    }
```



# 五大常用数据类型使用场景



## String

缓存：将数据以字符串方式存储
计数器功能：比如视频播放次数，点赞次数。
共享session：数据共享的功能，redis作为单独的应用软件用来存储一些共享数据供多个实例访问。
字符串的使用空间非常大，可以结合字符串提供的命令充分发挥自己的想象力

## hash

字典。键值对集合，即编程语言中的Map类型。适合存储对象，并且可以像数据库中update一个属性一样只修改某一项属性值。适用于：存储、读取、修改用户属性。也可以用Hash做表数据缓存

## list

链表(双向链表)，增删快，提供了操作某一段元素的API。适用于：最新消息排行等功能；消息队列。

## set

集合。哈希表实现，元素不重复，为集合提供了求交集、并集、差集等操作。适用于：共同好友；利用唯一性，统计访问网站的所有独立ip；> 好友推荐时，根据tag求交集，大于某个阈值就可以推荐。

## sorted set

有序集合。将Set中的元素增加一个权重参数score，元素按score有序排列。数据插入集合时，已经进行天然排序。适用于：排行榜；带权重的消息队列。

# StringRedisTemplate与RedisTemplate的区别?

两者的关系是StringRedisTemplate继承RedisTemplate。
两者的数据是不共通的；也就是说StringRedisTemplate只能管理StringRedisTemplate里面的数据
RedisTemplate只能管RedisTemplate中的数据。
SDR默认采用的序列化策略有两种，一种是String的序列化策略，一种是JDK的序列化策略。
StringRedisTemplate默认采用的是String的序列化策略，保存的key和value都是采用此策略序列化保存的。
RedisTemplate默认采用的是JDK的序列化策略，保存的key和value都是采用此策略序列化保存的。

# 注意

**目前数据类型已经达到八种，本文只讲解了五种常用类型**