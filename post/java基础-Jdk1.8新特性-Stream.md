---
title:       "jdk1.8新特性-stream"
subtitle:    ""
description: ""
date:        2019-07-02
author:      "麦子"
image:       "https://zhaohuabing.com//img/post-bg-unix-linux.jpg"
tags:        ["java基础", "stream"]
categories:  ["Tech" ]
---

[TOC]

**转载地址：https://blog.csdn.net/Young4Dream/article/details/76794659**

**转载地址：https://blog.csdn.net/chenhao_c_h/article/details/80691284**

**转载地址:  https://blog.csdn.net/Keith003/article/details/80252553**

# 为什么需要 Stream

它专注于对集合对象进行各种非常便利、高效的聚合操作（aggregate operation），或者大批量数据操作 (bulk data operation)。Stream API 借助于同样新出现的 Lambda 表达式，极大的提高编程效率和程序可读性。同时它提供串行和并行两种模式进行汇聚操作，并发模式能够充分利用多核处理器的优势，使用 fork/join 并行方式来拆分任务和加速处理过程。通常编写并行代码很难而且容易出错, 但使用 Stream API 无需编写一行多线程的代码，就可以很方便地写出高性能的并发程序。所以说，Java 8 中首次出现的 java.util.stream 是一个函数式语言+多核时代综合影响的产物。

**Stream 不是集合元素，它不是数据结构并不保存数据，它是有关算法和计算的，它更像一个高级版本的 Iterator。**

Stream 就如同一个迭代器（Iterator），单向，不可往复，数据只能遍历一次，遍历过一次后即用尽了，就好比流水从面前流过，一去不复返。

而和迭代器又不同的是，Stream 可以并行化操作，迭代器只能命令式地、串行化操作。顾名思义，当使用串行方式去遍历时，每个 item 读完后再读下一个 item。而使用并行去遍历时，数据会被分成多个段，其中每一个都在不同的线程中处理，然后将结果一起输出。

**Stream 的另外一大特点是，数据源本身可以是无限的。**

# 构造流的几种常见方法

```java
// 字符 
Stream stream = Stream.of("a", "b", "c");

// 数组
String [] strArray = new String[] {"a", "b", "c"};
stream = Stream.of(strArray);
stream = Arrays.stream(strArray);

// 集合
List<String> list = Arrays.asList(strArray);
stream = list.stream();
```

## 数值流的构造

需要注意的是，对于基本数值型，目前有三种对应的包装类型 Stream：IntStream、LongStream、DoubleStream`。当然我们也可以用 `Stream<Integer>、Stream<Long> >、Stream<Double>`，

```java
IntStream.of(new int[]{1, 2, 3}).forEach(System.out::println);
IntStream.range(1, 3).forEach(System.out::println);
IntStream.rangeClosed(1, 3).forEach(System.out::println);
```



# 流转换为其它数据结构

```java
// 1. Array
String[] strArray1 = stream.toArray(String[]::new);

// list
List<String> list1 = stream.collect(Collectors.toList());
List<String> list2 = stream.collect(Collectors.toCollection(ArrayList::new));

// Set
Set set1 = stream.collect(Collectors.toSet());
Stack stack1 = stream.collect(Collectors.toCollection(Stack::new));

// 3. String
String str = stream.collect(Collectors.joining()).toString();
```

# 流的操作



## Intermediate(中间操作)

一个流可以后面跟随零个或多个 intermediate 操作。其目的主要是打开流，做出某种程度的数据映射/过滤，然后返回一个新的流，交给下一个操作使用。**这类操作都是惰性化的（lazy），就是说，仅仅调用到这类方法，并没有真正开始流的遍历。**

多个中间操作可以连接起来形成一个流水线，除非流水线上触发终止操作，否则中间操作不会执行任何的处理！而在终止操作时一次性全部处理，**称为“惰性求值”。**

### 常用方法

map (mapToInt, flatMap 等)、 filter、 distinct、 sorted、 peek、 limit、 skip、 parallel、 sequential、 unordered

#### 筛选与切片

![20180509140419898](/img/20180509140419898.png)

#### 映射

![20180509140353794](/img/20180509140353794.png)

#### 排序

![20180509140320474](/img/20180509140320474.png)



## Terminal(最终操作)

一个流只能有一个 terminal 操作，当这个操作执行后，流就被使用“光”了，无法再被操作。所以这必定是流的最后一个操作。Terminal 操作的执行，才会真正开始流的遍历，并且会生成一个结果，或者一个 side effect。

### 常用方法

forEach、 forEachOrdered、 toArray、 reduce、 collect、 min、 max、 count、 anyMatch、 allMatch、 noneMatch、 findFirst、 findAny、 iterator

#### 查找与匹配

![20180509140624786](/img/20180509140624786.png)

![20180509140651964](/img/20180509140651964.png)



#### 归约

![20180509140732558](/img/20180509140732558.png)

**注：map 和 reduce 的连接通常称为map-reduce 模式，因 Google 用它来进行网络搜索而出名**



#### 收集

![20180509140941161](/img/20180509140941161.png)

Collector 接口中方法的实现决定了如何对流执行收集操作(如收集到List、Set、Map)。但是Collectors 实用类提供了很多静态方法，可以方便地创建常见收集器实例，具体方法与实例如下表：

![20180509141507583](/img/20180509141507583.png)



## short-circuiting(环形)

- 对于一个**intermediate** 操作，如果它接受的是一个无限大（infinite/unbounded）的**Stream**，但返回一个有限的新**Stream**。
- 对于一个 **terminal** 操作，如果它接受的是一个无限大的 **Stream**，但能在有限的时间计算出结果。

当操作一个无限大的 **Stream**，而又希望在有限时间内完成操作，则在管道内拥有一个 short-circuiting 操作是必要非充分条件。

### 常用方法

anyMatch、 allMatch、 noneMatch、 findFirst、 findAny、 limit

# 常用实例代码



## Intermediate(中间操作)

```java
   /**
    对每个元素执行mapper指定的操作，并用所有mapper返回的Stream中的元素组成一个新的Stream作为最终返回结果，通俗易懂就是将原来的stream中的所有元素都展开组成一个新的stream, map 只能对应一维数据， flatMap主要对应的事二维数据，可以看出返回的对象是一个流， 所以可以在对流进行数据处理。
   */ 
    public static void flatMapTest() {
        List<School> childs1 = new ArrayList<School>();
        childs1.add(new School(10, "10号", "深圳", "开放"));
        childs1.add(new School(11, "11号", "湖南", "建设中"));

        List<School> childs2 = new ArrayList<School>();
        childs2.add(new School(12, "12号", "邵阳", "倒闭"));
        childs2.add(new School(13, "13号", "西乡", "倒闭"));

        List<School> childs3 = new ArrayList<School>();
        childs3.add(new School(14, "14号", "保安", "建设中"));

        List<List<School>> data = new ArrayList<>();
        data.add(childs1);
        data.add(childs2);
        data.add(childs3);

        data.stream().flatMap(temp -> temp.stream().filter(entity -> entity.getId() > 12)).forEach(System.out::println);
        ;

        // 执行结果如下
        // School(id=13, name=13号, address=西乡)
        // School(id=14, name=14号, address=保安)
    }

    // findFirst 获取第一个
    public static void findFirstTest(List<School> schoolList) {
        School school = schoolList.stream().findFirst().get();
        System.out.println(school);
    }

    // 接收一个方法作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素
    public static void mapTest(List<School> schoolList) {
        Function<School, School> fun = (school) -> {
            String name = school.getName();
            school.setName(name + "---" + school.getId());
            return school;
        };
        schoolList.stream().map(fun).forEach(System.out::println);
    }

    // distinct 去重
    public static void distinctTst(List<School> schoolList) {
        schoolList.stream().distinct().sorted((o1, o2) -> o1.getId() - o2.getId()).forEach(System.out::println);
    }

    // skip 和 limit是互斥的， skip是跳过前面几个， limit是截断， 顺序都是从前面作为起点。
    public static void skipTest(List<School> schoolList) {
        schoolList.stream().skip(2).forEach(System.out::println);
    }

    // 排序
    public static void sortedTest(List<School> schoolList) {
        schoolList.stream().sorted((o1, o2) -> o1.getId() - o2.getId()).forEach(System.out::println);
    }

    // filter
    public static void filterTest(List<School> schoolList) {
        schoolList.stream().filter(school -> school.getId() > 3).forEach(System.out::println);

    }

    // limit
    public static void limitTest(List<School> schoolList) {
        schoolList.stream().limit(2).forEach(System.out::println);
    }


    // max，min，count,sum，avg，
    public static void sqlTest(List<School> schoolList) {
        Optional<School> opt = schoolList.stream().max((o1, o2) -> o1.getId() - o2.getId());
        School school = opt.get();
        System.out.println(school);

        opt = schoolList.stream().min((o1, o2) -> o1.getId() - o2.getId());
        school = opt.get();
        System.out.println(school);

        long count = schoolList.stream().count();
        System.out.println(count);

        double avg = schoolList.stream().collect(Collectors.averagingInt(x -> x.getId()));
        System.out.println(avg);

        double sum = schoolList.stream().collect(Collectors.summingInt(x -> x.getId()));
        System.out.println(sum);
    }
```





## Terminal(最终操作)

```java
    // 归约---可以将流中元素反复结合起来，得到一个值
    // map 和 reduce 的连接通常称为map-reduce 模式，因 Google 用它来进行网络搜索而出名
    public static void reduceTest(List<School> schoolList) {

        schoolList.stream().map(x -> x.getId()).forEach(System.out::println);

        Optional<Integer> op = schoolList.stream().map(x -> x.getId()).reduce((x, y) -> x + y);
        System.out.println(op.get());

        // 相对于一个参数的方法来说，它多了一个T类型的参数；实际上就相当于需要计算的值在Stream的基础上多了一个初始化的值。
        int count = schoolList.stream().map(x -> x.getId()).reduce(2, (x, y) -> x + y);
        System.out.println(count);
    }
```



### 集合常用方法

```java

        // 一次性获取 avg， sum 等操作
        DoubleSummaryStatistics doubleSummaryStatistics = schoolList.stream()
                .collect(Collectors.summarizingDouble(School::getId));
        System.out.println(doubleSummaryStatistics.getSum());
        System.out.println(doubleSummaryStatistics.getAverage());
        System.out.println(doubleSummaryStatistics.getMax());
        System.out.println(doubleSummaryStatistics.getMin());
        System.out.println(doubleSummaryStatistics.getCount());

        // 连接
        String str = schoolList.stream().map(School::getName).collect(Collectors.joining(",", "$$$", "###"));
        System.out.println(str);

        List<String> dataIdList = schoolList.stream().map(School::getName).collect(Collectors.toList());
        dataIdList.forEach(System.out::println);
        System.out.println("-----------------");

        Set<String> dataSet = schoolList.stream().map(School::getName).collect(Collectors.toSet());
        dataSet.forEach(System.out::println);
        System.out.println("-----------------");

        // 这里转换成其他数据结构
        HashSet<String> hashSet = schoolList.stream().map(School::getName)
                .collect(Collectors.toCollection(HashSet::new));
        hashSet.forEach(System.out::println);

        // 总数
        Long count = schoolList.stream().collect(Collectors.counting());
        System.out.println("总数:" + count);

        // 平均值
        Double avg = schoolList.stream().collect(Collectors.averagingDouble(School::getId));
        System.out.println("平均值:" + avg);

        // 总和
        Double sum = schoolList.stream().collect(Collectors.summingDouble(School::getId));
        System.out.println("总和:" + sum);

        // 最大值
        Optional<School> max = schoolList.stream()
                .collect(Collectors.maxBy((x, y) -> Double.compare(x.getId(), y.getId())));
        System.out.println("最大值" + max.get());

        // 最小值
        Optional<School> min = schoolList.stream()
                .collect(Collectors.minBy((x, y) -> Double.compare(x.getId(), y.getId())));
        System.out.println("最小值" + min.get());

        // 分组
        Map<String, List<School>> groupData = schoolList.stream().collect(Collectors.groupingBy(x -> x.getStatus()));
        System.out.println("分组:" + groupData);
        /***
         * 
         * 
         * 分组:{
         * 
         * 倒闭=[School(id=3, name=3号, address=邵阳, status=倒闭), School(id=2, name=2号,
         * address=西乡, status=倒闭)],
         * 
         * 开放=[School(id=0, name=5号, address=深圳, status=开放), School(id=4, name=4号,
         * address=保安, status=开放), School(id=50, name=5号, address=深圳, status=开放)],
         * 
         * 建设中=[School(id=1, name=1号, address=湖南, status=建设中)]
         * 
         * }
         * 
         * 
         * 
         */

        // 多级分组 先按状态分组，在按id分组, 这种分组可以无限分组
        Map<String, Map<String, List<School>>> data = schoolList.stream()
                .collect(Collectors.groupingBy(School::getStatus, Collectors.groupingBy((x) -> {
                    School currentSchool = (School) x;
                    if (currentSchool.getId() > 3) {
                        return "幸运号码";
                    } else {
                        return "通吃号码";
                    }

                })));
        System.out.println("多级分组:" + data);
        /***
         * 
         * {
         * 
         * 倒闭={通吃号码=[School(id=3, name=3号, address=邵阳, status=倒闭), School(id=2, name=2号,
         * address=西乡, status=倒闭)]},
         * 
         * 开放={通吃号码=[School(id=0, name=5号, address=深圳, status=开放)], 幸运号码=[School(id=4,
         * name=4号, address=保安, status=开放), School(id=50, name=5号, address=深圳,
         * status=开放)]},
         * 
         * 建设中={通吃号码=[School(id=1, name=1号, address=湖南, status=建设中)]}
         * 
         * }
         * 
         */

        // 分区
        System.out.println("分区----------------------------");
        Map<Boolean, List<School>> map = schoolList.stream().collect(Collectors.partitioningBy((x) -> x.getId() > 4));
        System.out.println(map);
        /**
         * 
         * 
         * {
         * 
         * false=[School(id=0, name=5号, address=深圳, status=开放), School(id=1, name=1号,
         * address=湖南, status=建设中), School(id=3, name=3号, address=邵阳, status=倒闭),
         * School(id=2, name=2号, address=西乡, status=倒闭), School(id=4, name=4号,
         * address=保安, status=开放)],
         * 
         * true=[School(id=50, name=5号, address=深圳, status=开放)] }
         * 
         */

    }
```

# 注意

在对于一个 Stream 进行多次转换操作 (Intermediate 操作)，每次都对 Stream 的每个元素进行转换，而且是执行多次，这样时间复杂度就是 N（转换次数）个 for 循环里把所有操作都做掉的总和吗？其实不是这样的，转换操作都是 lazy 的，多个转换操作只会在 Terminal 操作的时候融合起来，一次循环完成。我们可以这样简单的理解，Stream 里有个操作函数的集合，每次转换操作就是把转换函数放入这个集合中，在 Terminal 操作的时候循环 Stream 对应的集合，然后对每个元素执行所有的函数。

# Stream 的特性可以归纳

- 不是数据结构
- 它没有内部存储，它只是用操作管道从 source（数据结构、数组、generator function、IO channel）抓取数据。
- **它也绝不修改自己所封装的底层数据结构的数据**。例如 Stream 的 filter 操作会产生一个不包含被过滤元素的新 Stream，而不是从 source 删除那些元素。
- 所有 Stream 的操作必须以 lambda 表达式为参数
- 不支持索引访问
- 很容易生成数组或者 List
- Intermediate 操作永远是惰性化的。
- 并行能力
- 当一个 Stream 是并行化的，就不需要再写多线程代码，所有对它的操作会自动并行进行的。
- 可以是无限的
- 集合有固定大小，Stream 则不必。limit(n) 和 findFirst() 这类的 short-circuiting 操作可以对无限的 Stream 进行运算并很快完成。