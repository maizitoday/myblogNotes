---
title:       "ElasticSearch-API-常用操作"
subtitle:    ""
description: "ElasticSearch的常用增删查改"
date:        2020-05-18
author:      "麦子"
image:       "https://img.zhaohuabing.com/in-post/istio-install_and_example/post-bg.jpg"
tags:        ["elasticSearch"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：https://juejin.im/post/5d9e79ac51882509701337c2         作者：虚无境**

# ElasticSearch JAVA API

 目前市面上有几种常见的ElasticSearch Java API架包，JestClient、SpringBoot整合的SpringData、Spring整合的ElasticsearchTemplate、Elasticsearch Bboss等一些开源架包，上述这些第三方整合的架包中，基本已经支持日常的使用，除了支持的ES版本会低一些而已。

本文介绍的是ElasticSearch官方的`Java High Level REST Client`的使用，`Java High Level REST Client`是ElasticSearch官方目前推荐使用的，适用于6.x以上的版本，要求JDK在1.8以上，可以很好的在大版本中进行兼容，并且该架包自身也包含`Java Low Level REST Client`中的方法，可以应对一些特需的情况进行特殊的处理， 它对于一些常用的方法封装Restful风格，可以直接对应操作名调用使用即可，支持同步和异步(Async)调用。



# pom.xml

```java
 <dependency>
      <groupId>org.elasticsearch.client</groupId>
      <artifactId>elasticsearch-rest-high-level-client</artifactId>
      <version>6.8.9</version>
 </dependency>
```



# 初始化连接

```java

RestHighLevelClient client = new RestHighLevelClient(RestClient.builder(new HttpHost("127.0.0.1", 9200)));

client.close(); //关闭
```

# CRUD操作

## 新增数据

ElasticSearch可以直接新增数据，只要你指定了index(索引库名称)和type(类型)即可。在新增的时候你可以自己指定主键ID，也可以不指定，由 ElasticSearch自身生成。Elasticsearch `Java High Level REST Client`新增数据提供了三种方法，这里我们就来看一下这三种写法吧。

 

### 通过jsonString进行创建

```java
  String index = "test1";
	String type = "_doc";
	// 唯一编号
	String id = "1";
	IndexRequest request = new IndexRequest(index, type, id);

	String jsonString = "{" + "\"uid\":\"1234\","+ "\"phone\":\"12345678909\","+ "\"msgcode\":\"1\"," + "\"sendtime\":\"2019-03-14 01:57:04\","
			+ "\"message\":\"xuwujing study Elasticsearch\"" + "}";
	request.source(jsonString, XContentType.JSON);
	IndexResponse indexResponse = client.index(request, RequestOptions.DEFAULT);

```



### 通过map创建,会自动转换成json的数据

```java
String index = "test1";
String type = "_doc";
// 唯一编号
String id = "2";
IndexRequest request = new IndexRequest(index, type, id);
Map<String, Object> jsonMap = new HashMap<>();
jsonMap.put("uid", 1234);
jsonMap.put("phone", 12345678909L);
jsonMap.put("msgcode", 1);
jsonMap.put("sendtime", "2019-03-14 01:57:04");
jsonMap.put("message", "xuwujing study Elasticsearch");
request.source(jsonMap);
IndexResponse indexResponse = client.index(request, RequestOptions.DEFAULT);
System.out.println(indexResponse.toString());
```

**推荐第二种，比较容易理解和使用。**



## 创建索引库

在上述示例中，我们通过直接通过创建数据从而创建了索引库，但是没有创建索引库而通过ES自身生成的这种并不友好，因为它会使用默认的配置，字段结构都是text(text的数据会分词，在存储的时候也会额外的占用空间)，分片和索引副本采用默认值，默认是5和1，ES的分片数在创建之后就不能修改，除非reindex，所以这里我们还是指定数据模板进行创建。 使用JAVA API 创建索引库的方法和上述中新增数据的一样，有三种方式，不过这里就只介绍一种。

```java
private static void createIndex(RestHighLevelClient client) throws IOException {
        String type = "_doc";
        String index = "maizi";
        // setting 的值
        Map<String, Object> setmapping = new HashMap<>();
        // 分区数、副本数、缓存刷新时间
        setmapping.put("number_of_shards", 10);
        setmapping.put("number_of_replicas", 1);
        setmapping.put("refresh_interval", "5s");
        Map<String, Object> keyword = new HashMap<>();
        // 设置类型
        keyword.put("type", "keyword");
        Map<String, Object> lon = new HashMap<>();
        // 设置类型
        lon.put("type", "long");
        Map<String, Object> date = new HashMap<>();
        // 设置类型
        date.put("type", "date");
        date.put("format", "yyyy-MM-dd HH:mm:ss");

        Map<String, Object> jsonMap2 = new HashMap<>();
        Map<String, Object> properties = new HashMap<>();
        // 设置字段message信息
        properties.put("uid", lon);
        properties.put("phone", lon);
        properties.put("msgcode", lon);
        properties.put("message", keyword);
        properties.put("sendtime", date);
        Map<String, Object> mapping = new HashMap<>();
        mapping.put("properties", properties);
        jsonMap2.put(type, mapping);

        GetIndexRequest getRequest = new GetIndexRequest();
        getRequest.indices(index);
        getRequest.local(false);
        getRequest.humanReadable(true);
        boolean exists2 = client.indices().exists(getRequest, RequestOptions.DEFAULT);
        // 如果存在就不创建了
        if (exists2) {
            System.out.println(index + "索引库已经存在!");
            return;
        }
        // 开始创建库
        CreateIndexRequest request = new CreateIndexRequest(index);
        try {
            // 加载数据类型
            request.settings(setmapping);
            // 设置mapping参数
            request.mapping(type, jsonMap2);
            // 设置别名
            request.alias(new Alias("pancm_alias"));
            CreateIndexResponse createIndexResponse = client.indices().create(request, RequestOptions.DEFAULT);
            boolean falg = createIndexResponse.isAcknowledged();
            if (falg) {
                System.out.println("创建索引库:" + index + "成功！");
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
```

**注：创建索引库的时候，一定要先判断索引库是否存在！！！** 这里创建索引库的时候顺便也指定了别名（alias），这个别名是一个好东西，使用恰当可以提升查询性能，这里我们留着下次在讲。



## 修改数据

ES提供修改API的时候，有两种方式，一种是直接修改，但是若数据不存在会抛出异常，另一种则是存在更新，不存着就插入。相比第一种，第二种会更加好用一些，不过在写入速度上是不如第一种的。

```java
private static void update(RestHighLevelClient client) throws IOException {
        String type = "_doc";
        String index = "test1";
        // 唯一编号
        String id = "1";
        UpdateRequest upateRequest = new UpdateRequest();
        upateRequest.id(id);
        upateRequest.index(index);
        upateRequest.type(type);

        // 依旧可以使用Map这种集合作为更新条件
        Map<String, Object> jsonMap = new HashMap<>();
        jsonMap.put("uid", 12345);
        jsonMap.put("phone", 123456789019L);
        jsonMap.put("msgcode", 2);
        jsonMap.put("sendtime", "2019-03-14 01:57:04");
        jsonMap.put("message", "xuwujing study Elasticsearch");
        upateRequest.doc(jsonMap);
        // upsert 方法表示如果数据不存在，那么就新增一条
        upateRequest.docAsUpsert(true);
        client.update(upateRequest, RequestOptions.DEFAULT);
        System.out.println("更新成功！");

    }
```

**注:  upsert 方法表示如果数据不存在，那么就新增一条，默认是false。**





## 查询数据

几个常用的查询API这里就简单的介绍下用法，然后再直接给出所有的查询语句代码。



### 查询API

- 等值（term查询：QueryBuilders.termQuery(name,value);
- 多值(terms)查询:QueryBuilders.termsQuery(name,value,value2,value3...);
- 范围（range)查询：QueryBuilders.rangeQuery(name).gte(value).lte(value);
- 存在(exists)查询:QueryBuilders.existsQuery(name);
- 模糊(wildcard)查询:QueryBuilders.wildcardQuery(name,*+value+*);
- 组合（bool）查询: BoolQueryBuilder boolQueryBuilder = new BoolQueryBuilder();



### 查询所有代码示例

```java
 private static void allSearch() throws IOException {
    SearchRequest searchRequestAll = new SearchRequest();
    SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
    searchSourceBuilder.query(QueryBuilders.matchAllQuery());
    searchRequestAll.source(searchSourceBuilder);
    // 同步查询
    SearchResponse searchResponseAll = client.search(searchRequestAll, RequestOptions.DEFAULT);
    System.out.println("所有查询总数:" + searchResponseAll.getHits().getTotalHits());
}

```



###  一般查询代码示例

其实就是等值查询，只不过在里面加入了分页、排序、超时、路由等等设置，并且在查询结果里面增加了一些处理。

```java
   private static void genSearch() throws IOException {
    String type = "_doc";
    String index = "test1";
    // 查询指定的索引库
    SearchRequest searchRequest = new SearchRequest(index);
    searchRequest.types(type);
    SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
    // 设置查询条件
    sourceBuilder.query(QueryBuilders.termQuery("uid", "1234"));
    // 设置起止和结束
    sourceBuilder.from(0);
    sourceBuilder.size(5);
    sourceBuilder.timeout(new TimeValue(60, TimeUnit.SECONDS));
    // 设置路由
//		searchRequest.routing("routing");
    // 设置索引库表达式
    searchRequest.indicesOptions(IndicesOptions.lenientExpandOpen());
    // 查询选择本地分片，默认是集群分片
    searchRequest.preference("_local");

    // 排序
    // 根据默认值进行降序排序
//	sourceBuilder.sort(new ScoreSortBuilder().order(SortOrder.DESC));
    // 根据字段进行升序排序
//	sourceBuilder.sort(new FieldSortBuilder("id").order(SortOrder.ASC));

    // 关闭suorce查询
//	sourceBuilder.fetchSource(false);

    String[] includeFields = new String[]{"title", "user", "innerObject.*"};
    String[] excludeFields = new String[]{"_type"};
    // 包含或排除字段
//	sourceBuilder.fetchSource(includeFields, excludeFields);

    searchRequest.source(sourceBuilder);
	System.out.println("普通查询的DSL语句:"+sourceBuilder.toString());
    // 同步查询
    SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);

    // HTTP状态代码、执行时间或请求是否提前终止或超时
    RestStatus status = searchResponse.status();
    TimeValue took = searchResponse.getTook();
    Boolean terminatedEarly = searchResponse.isTerminatedEarly();
    boolean timedOut = searchResponse.isTimedOut();

    // 供关于受搜索影响的切分总数的统计信息，以及成功和失败的切分
    int totalShards = searchResponse.getTotalShards();
    int successfulShards = searchResponse.getSuccessfulShards();
    int failedShards = searchResponse.getFailedShards();
    // 失败的原因
    for (ShardSearchFailure failure : searchResponse.getShardFailures()) {
        // failures should be handled here
    }
    // 结果
    searchResponse.getHits().forEach(hit -> {
        Map<String, Object> map = hit.getSourceAsMap();
        System.out.println("普通查询的结果:" + map);
    });
    System.out.println("\n=================\n");
}

```



### 或查询

其实这个或查询也是bool查询中的一种，这里的查询语句相当于SQL语句中的

```sql
SELECT * FROM test1 where (uid = 1 or uid =2) and phone = 12345678919
```

代码示例:

```java
private static void orSearch() throws IOException {
    SearchRequest searchRequest = new SearchRequest();
    searchRequest.indices("test1");
    searchRequest.types("_doc");
    SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
    BoolQueryBuilder boolQueryBuilder = new BoolQueryBuilder();
    BoolQueryBuilder boolQueryBuilder2 = new BoolQueryBuilder();
 
      /**
     *  SELECT * FROM test1 where (uid = 1234 or uid =12345)  and phone = 12345678909
     * */
    boolQueryBuilder2.should(QueryBuilders.termQuery("uid", 1234));
    boolQueryBuilder2.should(QueryBuilders.termQuery("uid", 12345));
    boolQueryBuilder.must(boolQueryBuilder2);
    boolQueryBuilder.must(QueryBuilders.termQuery("phone", "12345678909"));
    searchSourceBuilder.query(boolQueryBuilder);
    System.out.println("或查询语句:" + searchSourceBuilder.toString());
    searchRequest.source(searchSourceBuilder);
    // 同步查询
    SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);

    searchResponse.getHits().forEach(documentFields -> {

        System.out.println("查询结果:" + documentFields.getSourceAsMap());
    });

}

```



### 模糊查询

相当于SQL语句中的like查询。

```java
 private static void likeSearch() throws IOException {
        String type = "_doc";
        String index = "test1";
        SearchRequest searchRequest = new SearchRequest();
        searchRequest.indices(index);
        searchRequest.types(type);
        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
        BoolQueryBuilder boolQueryBuilder = new BoolQueryBuilder();

        /**
         * SELECT * FROM p_test where message like '%xu%';
         */
        boolQueryBuilder.must(QueryBuilders.wildcardQuery("message", "*xu*"));
        searchSourceBuilder.query(boolQueryBuilder);
        System.out.println("模糊查询语句:" + searchSourceBuilder.toString());
        searchRequest.source(searchSourceBuilder);
        // 同步查询
        SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);
        searchResponse.getHits().forEach(documentFields -> {
            System.out.println("模糊查询结果:" + documentFields.getSourceAsMap());
        });
        System.out.println("\n=================\n");
    }
```



### 多值查询

也就是相当于SQL语句中的in查询。

```java
 	 private static void inSearch() throws IOException {
        String type = "_doc";
        String index = "test1";
        // 查询指定的索引库
        SearchRequest searchRequest = new SearchRequest(index,type);
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        /**
         *  SELECT * FROM p_test where uid in (1,2)
         * */
        // 设置查询条件
        sourceBuilder.query(QueryBuilders.termsQuery("uid", 1, 2));
        searchRequest.source(sourceBuilder);
  		System.out.println("in查询的DSL语句:"+sourceBuilder.toString());
        // 同步查询
        SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);
        // 结果
        searchResponse.getHits().forEach(hit -> {
            Map<String, Object> map = hit.getSourceAsMap();
            String string = hit.getSourceAsString();
            System.out.println("in查询的Map结果:" + map);
            System.out.println("in查询的String结果:" + string);
        });

        System.out.println("\n=================\n");
    }

```



### 存在查询

判断是否存在该字段，用法和SQL语句中的exist类似。

```java
  private static void existSearch() throws IOException {
    String type = "_doc";
    String index = "test1";
    // 查询指定的索引库
    SearchRequest searchRequest = new SearchRequest(index);
    searchRequest.types(type);
    SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();

    // 设置查询条件
     sourceBuilder.query(QueryBuilders.existsQuery("msgcode"));
    searchRequest.source(sourceBuilder);
    System.out.println("存在查询的DSL语句:"+sourceBuilder.toString());
    // 同步查询
    SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);
    // 结果
    searchResponse.getHits().forEach(hit -> {
        Map<String, Object> map = hit.getSourceAsMap();
        String string = hit.getSourceAsString();
        System.out.println("存在查询的Map结果:" + map);
        System.out.println("存在查询的String结果:" + string);
    });
    System.out.println("\n=================\n");
}

```



### 范围查询

和SQL语句中<>使用方法一样,其中gt是大于，lt是小于，gte是大于等于，lte是小于等于。

```java
private static void rangeSearch() throws IOException{
    String type = "_doc";
    String index = "test1";
    SearchRequest searchRequest = new SearchRequest(index);
    searchRequest.types(type);
    SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();

    // 设置查询条件
    sourceBuilder.query(QueryBuilders.rangeQuery("sendtime").gte("2019-01-01 00:00:00").lte("2019-12-31 23:59:59"));
    searchRequest.source(sourceBuilder);
     System.out.println("范围查询的DSL语句:"+sourceBuilder.toString());
    // 同步查询
    SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);
    // 结果
    searchResponse.getHits().forEach(hit -> {
        String string = hit.getSourceAsString();
        System.out.println("范围查询的String结果:" + string);
    });
    System.out.println("\n=================\n");
}

```



### 正则查询

ES可以使用正则进行查询，查询方式也非常的简单，代码示例如下：

```java
 private static void regexpSearch() throws IOException{
    String type = "_doc";
    String index = "test1";
    // 查询指定的索引库
    SearchRequest searchRequest = new SearchRequest(index);
    searchRequest.types(type);
    SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
    // 设置查询条件
    sourceBuilder.query(QueryBuilders.regexpQuery("message","xu[0-9]"));
    searchRequest.source(sourceBuilder);
	 System.out.println("正则查询的DSL语句:"+sourceBuilder.toString());
    // 同步查询
    SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);
    // 结果
    searchResponse.getHits().forEach(hit -> {
        Map<String, Object> map = hit.getSourceAsMap();
        String string = hit.getSourceAsString();
        System.out.println("正则查询的Map结果:" + map);
        System.out.println("正则查询的String结果:" + string);
    });

    System.out.println("\n=================\n");
}

```





## 删除数据

```java
 private static void delete(RestHighLevelClient client) throws IOException {
        String type = "_doc";
        String index = "test1";
        // 唯一编号
        String id = "1";
        DeleteRequest deleteRequest = new DeleteRequest();
        deleteRequest.id(id);
        deleteRequest.index(index);
        deleteRequest.type(type);
        // 设置超时时间
        deleteRequest.timeout(TimeValue.timeValueMinutes(2));
        // 设置刷新策略"wait_for"
        // 保持此请求打开，直到刷新使此请求的内容可以搜索为止。此刷新策略与高索引和搜索吞吐量兼容，但它会导致请求等待响应，直到发生刷新
        deleteRequest.setRefreshPolicy(WriteRequest.RefreshPolicy.WAIT_UNTIL);
        // 同步删除
        DeleteResponse deleteResponse = client.delete(deleteRequest, RequestOptions.DEFAULT);
    }
```

ES根据条件进行删除：

```java
private static void deleteByQuery() throws IOException {
	String type = "_doc";
	String index = "test1";
	DeleteByQueryRequest request = new DeleteByQueryRequest(index,type);
	// 设置查询条件
	request.setQuery(QueryBuilders.termsQuery("uid",1234));
	// 同步执行
	BulkByScrollResponse bulkResponse = client.deleteByQuery(request, RequestOptions.DEFAULT);
}

```



# 命令行显示数据格式

```
GET  test1/_search
```

数据格式

```json
{
  "took" : 5,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 2,
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "test1",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 1.0,
        "_source" : {
          "msgcode" : 1,
          "uid" : 1234,
          "phone" : 12345678909,
          "sendtime" : "2019-03-14 01:57:04",
          "message" : "xuwujing study Elasticsearch"
        }
      },
      {
        "_index" : "test1",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "msgcode" : 2,
          "uid" : 12345,
          "phone" : 123456789019,
          "sendtime" : "2019-03-14 01:57:04",
          "message" : "xuwujing study Elasticsearch"
        }
      }
    ]
  }
}

```

