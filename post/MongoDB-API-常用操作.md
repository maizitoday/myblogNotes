---
title:       "MongoDB-API-常用操作"
subtitle:    ""
description: ""
date:        2020-02-27
author:      "麦子"
image:       "https://c.pxhere.com/images/a1/23/147c2090573fcd3fbc38e7b7b5a7-1593571.jpg!d"
tags:        ["mongodb","MongoDB-API-常用操作"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：https://www.cnblogs.com/laoqing/p/11792578.html**

# 

```xml
  <dependency>
      <groupId>org.mongodb</groupId>
      <artifactId>mongo-java-driver</artifactId>
      <version>3.12.4</version>
   </dependency>
```

# 连接和关闭

```java
 private static com.mongodb.client.MongoClient mongoClient;

    /**
     * 关于MongoDB Client的初始化和关闭
     * 
     * 
     */
    public static MongoClient connection() {
        if (mongoClient == null) {
            mongoClient = MongoClients.create("mongodb://localhost:27017");
            System.out.println(mongoClient);
            System.out.println("连接成功");
        }
        return mongoClient;
    }

    /**
     * client连接到一个 Replica Set：
     */
    public static void initByReplicaSet() {
        mongoClient = MongoClients.create("mongodb://host1:27017,host2:27017,host3:27017");
        MongoClients.create("mongodb://host1:27017,host2:27017,host3:27017/?replicaSet=myReplicaSet");
        mongoClient = MongoClients.create(MongoClientSettings.builder()
                .applyToClusterSettings(builder -> builder.hosts(Arrays.asList(new ServerAddress("host1", 27017),
                        new ServerAddress("host2", 27017), new ServerAddress("host3", 27017))))
                .build());
    }

    /**
     * 连接关闭
     */
    public static void close() {
        if (mongoClient != null) {
            mongoClient.close();
            mongoClient = null;
        }
    }

```

# 创建database

```java
 /**
     * 
     * 获取database，如果没有会自动创建
     * 
     * @param databaseName
     * @return
     */
    public static MongoDatabase getDatabase(String databaseName) {
        MongoDatabase database = mongoClient.getDatabase(databaseName);
        System.out.println("获取数据库成功   " + database);
        return database;
    }
```

# 创建Collection

```java
 /**
     * 创建Collection
     * 
     * @param db
     * @param collectionName
     * @return
     */
    public static MongoCollection<Document> getCollection(MongoDatabase db, String collectionName) {
        MongoCollection<Document> collection = db.getCollection(collectionName);
        //db.createCollection(collectionName);
        System.out.println("创建collection成功   " + collection);
        return collection;
    }
```

# 注意

**上面的操作，如果没有Document的数据的时候， 客户端是看不到这个数据库的。** 

# 同步操作API

## pom.xml

```xml
<dependency>
      <groupId>org.mongodb</groupId>
      <artifactId>mongo-java-driver</artifactId>
      <version>3.12.4</version>
</dependency>
```



## 新增

```java
/**
     * 新增一条或者多条
     * 
     * @param collection
     */
    public static void insertDocument(MongoCollection<Document> collection) {
        Document document = new Document();
        document.append("java", "长沙");
        document.append("age", "8个月");
        // 新增一条
        collection.insertOne(document);

        // 新增多条
        List<Document> documents = new ArrayList<Document>();
        for (int i = 0; i < 5; i++) {
            Document newDocument = new Document();
            newDocument.append("java", "长沙-" + i);
            newDocument.append("age", "时间-" + i);
            documents.add(newDocument);
        }
        collection.insertMany(documents);
    }

    /**
     * MongoDB 的bulkWrite操作 （批量写入），对于数据很多时，效率很高
     * 
     * @param dataBaseName
     * @param collectionName
     * @param listData
     * @return
     */
    public BulkWriteResult bulkWrite(String dataBaseName, String collectionName,
            List<? extends WriteModel<? extends Document>> listData) {
        return getCollectionByName(dataBaseName, collectionName).bulkWrite(listData);
    }
```



## 查询

```java
/**
     * 查询数据库中有多少的collection
     * 
     * @param dataBaseName
     * @return
     */
    public static List<String> listCollectionNames(String dataBaseName) {
        List<String> stringList = new ArrayList<String>();
        mongoClient.getDatabase(dataBaseName).listCollectionNames().forEach((Consumer<? super String>) t -> {
            stringList.add(t);
        });
        return stringList;
    }

    /**
     * 查询数据库中的 对应的 collection
     * 
     * @param dataBaseName
     * @param collectionName
     * @return
     */
    public static MongoCollection<Document> getCollectionByName(String dataBaseName, String collectionName) {
        return getDatabase(dataBaseName).getCollection(collectionName);
    }

    /**
     * 精确查询，模糊查询，范围查询， 排序，返回条数
     * 
     * @param dataBaseName
     * @param collectionName
     * @param id
     * @return
     */
    public static FindIterable<Document> findMongoDbDocById(String dataBaseName, String collectionName, String id) {
        BasicDBObject searchDoc = new BasicDBObject();
        // 通过id（objectid）精确查询
        // searchDoc.put("_id", new ObjectId(id));

        // 模糊查询
        searchDoc.put("java", new BasicDBObject("$regex", "长沙"));

        // searchDoc.put("java", new BasicDBObject("$gte", startId).append("$lte",
        // endId));

        //
        return getCollectionByName(dataBaseName, collectionName).find(searchDoc).limit(10)
                .sort(new Document().append("age", 1));
    }
```



## 修改

```java
/**
     * 
     * 更新一条， 批量更新
     * 
     */
    public static void update() {
        MongoDatabase database = App.getDatabase("maiziDB");
        MongoCollection<Document> workCollection = database.getCollection("work");
        Bson eqBson = Filters.eq("java", "长沙");
        // 要修改的内容
        Document updateBson = new Document("$set", new Document("age", 25).append("school", "一度教育"));
        UpdateResult updateMany = workCollection.updateMany(eqBson, updateBson);
        System.out.println(updateMany.toString());

        // workCollection.updateMany(filter, update)
    }
```



## 删除

```java
/**
     * 
     * 根据多个条件删除单个或者是多个
     * 
     */
    public static void remove() {
        MongoDatabase database = App.getDatabase("maiziDB");
        MongoCollection<Document> workCollection = database.getCollection("work");
        // 单一条件删除
        Bson delBson = Filters.eq("age", "时间-0");
        DeleteResult deleteMany = workCollection.deleteMany(delBson);
        System.out.println(deleteMany.toString());

        // 多个条件合并，删除多个
        Bson delBson1 = Filters.gte("age", 20);
        // 构造删除条件（age<=30）
        Bson delBson2 = Filters.lte("age", 30);
        // 合并删除条件
        Bson delBsonCount = Filters.and(delBson1, delBson2);
        deleteMany = workCollection.deleteMany(delBsonCount);
        System.out.println(deleteMany.toString());
    }
```

 

## 替换操作

```java
/**
     * 存在就替换，不存在的话就插入
     * 
     * @param dataBaseName
     * @param collectionName
     * @param var1
     * @param var2
     * @param var3
     * @return
     */
    public UpdateResult replaceDoc(String dataBaseName, String collectionName, Bson var1, Document var2,
            ReplaceOptions var3) {
        return getCollectionByName(dataBaseName, collectionName).replaceOne(var1, var2, var3);
    }

    /**
     * 设置存在就替换，不存在的话就插入
     * 
     * @param dataBaseName
     * @param collectionName
     * @param var1
     * @param var2
     * @return
     */
    public UpdateResult replaceDoc(String dataBaseName, String collectionName, Bson var1, Document var2) {
        Document documentQuery = new Document("_id", new ObjectId("id"));
        Document document = new Document("_id", new ObjectId("id"));
        ReplaceOptions replaceOptions = new ReplaceOptions();
        replaceOptions.upsert(true);
        return replaceDoc(dataBaseName, collectionName, documentQuery, document, replaceOptions);
    }
```



# 异步操作API

mongodb异步驱动程序提供了异步api，可以利用netty或java 7的asynchronoussocketchannel实现快速、无阻塞的i/o，maven依赖

## pom.xml

```xml
  <dependency>
      <groupId>org.mongodb</groupId>
      <artifactId>mongodb-driver-async</artifactId>
      <version>3.12.4</version>
    </dependency>
```

**异步操作必然会涉及到回调，回调时采用ResultCallback<Document>**



## 异步insert操作

```java
collection.insertMany(documents, new SingleResultCallback<Void>() {
    @Override
    public void onResult(final Void result, final Throwable t) {
        System.out.println("Documents inserted!");
    }
});
```



## 异步统计操作

```java
collection.countDocuments(
  new SingleResultCallback<Long>() {
      @Override
      public void onResult(final Long count, final Throwable t) {
          System.out.println(count);
      }
  });
```



## 异步更新操作

```java
collection.updateOne(eq("i", 10), set("i", 110),
    new SingleResultCallback<UpdateResult>() {
        @Override
        public void onResult(final UpdateResult result, final Throwable t) {
            System.out.println(result.getModifiedCount());
        }
    });
```



## 异步删除操作

```java
collection.deleteMany(gte("i", 100), new SingleResultCallback<DeleteResult>() {
    @Override
    public void onResult(final DeleteResult result, final Throwable t) {
        System.out.println(result.getDeletedCount());
    }
});
```





# MongoDB Reactive Streams 操作API

官方的MongoDB reactive streams Java驱动程序，为MongoDB提供异步流处理和无阻塞处理。

完全实现reactive streams api，以提供与jvm生态系统中其他reactive streams的互操作，一般适合于大数据的处理，比如spark，flink，storm等。

## pom.xml

```xml
<dependencies>
    <dependency>
        <groupId>org.mongodb</groupId>
        <artifactId>mongodb-driver-reactivestreams</artifactId>
        <version>1.12.0</version>
    </dependency>
</dependencies>
```



## 会包含如下三部分

1. Publisher：Publisher 是数据的发布者。Publisher 接口只有一个方法 subscribe，用于添加数据的订阅者，也就是 Subscriber。
2. Subscriber： 是数据的订阅者。Subscriber 接口有4个方法，都是作为不同事件的处理器。在订阅者成功订阅到发布者之后，其 onSubscribe(Subscription s) 方法会被调用。
3. Subscription：表示的是当前的订阅关系。



## 代码示例

```java
//建立连接
MongoClient mongoClient = MongoClients.create(mongodbUrl);
//获得数据库对象
MongoDatabase database = client.getDatabase(databaseName);
//获得集合
MongoCollection collection = database.getCollection(collectionName);
 
//异步返回Publisher
FindPublisher publisher = collection.find();
 
//订阅实现
publisher.subscribe(new Subscriber() {
    @Override
    public void onSubscribe(Subscription str) {
        System.out.println("start...");
        //执行请求
        str.request(Integer.MAX_VALUE);
    }
    @Override
    public void onNext(Document document) {
        //获得文档
        System.out.println("Document:" + document.toJson());
    }
 
    @Override
    public void onError(Throwable t) {
        System.out.println("error occurs.");
    }
 
    @Override
    public void onComplete() {
        System.out.println("finished.");
    }
});
```

