---
layout: post
title:  使用Java介绍Apache Flink
category: libraries
copyright: libraries
excerpt: Apache Flink
---

## 1. 概述

Apache Flink是一个大数据处理框架，允许程序员以非常高效和可扩展的方式处理大量数据。

在本文中，我们将介绍[Apache Flink](https://ci.apache.org/projects/flink/flink-docs-release-1.2/index.html) Java API中的一些核心API概念和标准数据转换。此API的流式风格使其易于与Flink的核心构造-分布式集合配合使用。

首先，我们将了解Flink的DataSet API转换，并使用它们来实现字数统计程序。然后，我们将简要介绍Flink的DataStream API，它允许你实时处理事件流。

## 2. Maven依赖

首先，我们需要将[flink-java](https://mvnrepository.com/artifact/org.apache.flink/flink-java)和[flink-test-utils](https://mvnrepository.com/artifact/org.apache.flink/flink-test-utils)库依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-java</artifactId>
    <version>1.16.1</version>
</dependency>
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-test-utils</artifactId>
    <version>1.16.1</version>
    <scope>test<scope>
</dependency>
```

## 3. 核心API概念

在使用Flink时，我们需要了解一些与其API相关的事项：

- **每个Flink程序都会对分布式数据集合执行转换**，它提供了各种用于转换数据的功能，包括过滤、映射、拼接、分组和聚合
- **Flink中的接收器操作触发流的执行以产生程序所需的结果**，例如将结果保存到文件系统或将其打印到标准输出
- **Flink转换是惰性的，这意味着它们直到调用接收器操作时才会执行**
- **Apache Flink API支持两种操作模式：批处理和实时**。如果你处理的是可以批处理模式处理的有限数据源，则可以使用DataSet API。如果你想要实时处理无限数据流，则需要使用DataStream API

## 4. DataSet API转换

Flink程序的入口点是[ExecutionEnvironment](https://ci.apache.org/projects/flink/flink-docs-release-1.2/api/java/org/apache/flink/api/scala/ExecutionEnvironment.html)类的一个实例-它定义了程序执行的上下文。

让我们创建一个ExecutionEnvironment来开始我们的处理：

```java
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
```

**请注意，当你在本地机器上启动应用程序时，它将在本地JVM上执行处理**。如果你希望在一组机器上开始处理，则需要在这些机器上安装[Apache Flink](https://nightlies.apache.org/flink/flink-docs-release-1.14//docs/try-flink/local_installation/)并相应地配置ExecutionEnvironment。

### 4.1 创建数据集

要开始执行数据转换，我们需要向程序提供数据。

让我们使用ExecutionEnvironment创建DataSet类的实例：

```java
DataSet<Integer> amounts = env.fromElements(1, 29, 40, 50);
```

你可以从多个来源创建数据集，例如Apache Kafka、CSV、文件或几乎任何其他数据源。

### 4.2 过滤和归约

一旦创建了DataSet类的实例，就可以对其应用转换。

假设你要过滤高于特定阈值的数字，然后将它们全部相加。你可以使用filter()和reduce()转换来实现此目的：

```java
int threshold = 30;
List<Integer> collect = amounts
    .filter(a -> a > threshold)
    .reduce((integer, t1) -> integer + t1)
    .collect();

assertThat(collect.get(0)).isEqualTo(90);
```

请注意，collect()方法是触发实际数据转换的接收操作。

### 4.3 映射

假设你有一个Person对象的DataSet：

```java
private static class Person {
    private int age;
    private String name;

    // standard constructors/getters/setters
}
```

接下来，让我们创建这些对象的DataSet：

```java
DataSet<Person> personDataSource = env.fromCollection(Arrays.asList(
    new Person(23, "Tom"),
    new Person(75, "Michael")));
```

假设你只想从集合的每个对象中提取age字段，你可以使用map()转换来仅获取Person类的特定字段：

```java
List<Integer> ages = personDataSource
    .map(p -> p.age)
    .collect();

assertThat(ages).hasSize(2);
assertThat(ages).contains(23, 75);
```

### 4.4 拼接

当你有两个数据集时，你可能希望在某些id字段上将它们拼接起来。为此，你可以使用join()转换。

让我们创建用户的交易和地址的集合：

```java
Tuple3<Integer, String, String> address
    = new Tuple3<>(1, "5th Avenue", "London");
DataSet<Tuple3<Integer, String, String>> addresses
    = env.fromElements(address);

Tuple2<Integer, String> firstTransaction 
    = new Tuple2<>(1, "Transaction_1");
DataSet<Tuple2<Integer, String>> transactions 
    = env.fromElements(firstTransaction, new Tuple2<>(12, "Transaction_2"));
```

两个元组中的第一个字段都是Integer类型，这是我们想要拼接两个数据集的id字段。

为了执行实际的连接逻辑，我们需要为地址和交易实现一个[KeySelector](https://ci.apache.org/projects/flink/flink-docs-release-1.2/api/java/org/apache/flink/api/java/functions/KeySelector.html)接口：

```java
private static class IdKeySelectorTransaction implements KeySelector<Tuple2<Integer, String>, Integer> {
    @Override
    public Integer getKey(Tuple2<Integer, String> value) {
        return value.f0;
    }
}

private static class IdKeySelectorAddress implements KeySelector<Tuple3<Integer, String, String>, Integer> {
    @Override
    public Integer getKey(Tuple3<Integer, String, String> value) {
        return value.f0;
    }
}
```

每个选择器仅返回应执行拼接的字段。

**不幸的是，这里无法使用Lambda表达式，因为Flink需要泛型类型信息**。

接下来，让我们使用这些选择器实现合并逻辑：

```java
List<Tuple2<Tuple2<Integer, String>, Tuple3<Integer, String, String>>>
    joined = transactions.join(addresses)
    .where(new IdKeySelectorTransaction())
    .equalTo(new IdKeySelectorAddress())
    .collect();

assertThat(joined).hasSize(1);
assertThat(joined).contains(new Tuple2<>(firstTransaction, address));
```

### 4.5 排序

假设你有以下Tuple2集合：

```java
Tuple2<Integer, String> secondPerson = new Tuple2<>(4, "Tom");
Tuple2<Integer, String> thirdPerson = new Tuple2<>(5, "Scott");
Tuple2<Integer, String> fourthPerson = new Tuple2<>(200, "Michael");
Tuple2<Integer, String> firstPerson = new Tuple2<>(1, "Jack");
DataSet<Tuple2<Integer, String>> transactions = env.fromElements(fourthPerson, secondPerson, thirdPerson, firstPerson);
```

如果要按元组的第一个字段对此集合进行排序，则可以使用sortPartitions()转换：

```java
List<Tuple2<Integer, String>> sorted = transactions
    .sortPartition(new IdKeySelectorTransaction(), Order.ASCENDING)
    .collect();

assertThat(sorted)
    .containsExactly(firstPerson, secondPerson, thirdPerson, fourthPerson);
```

## 5. 字数统计

字数统计问题是展示大数据处理框架能力的常用问题之一，基本解决方案是计算文本输入中的单词出现次数，让我们使用Flink来实现此问题的解决方案。

作为解决方案的第一步，我们创建一个LineSplitter类，将输入拆分为标记(单词)，为每个标记收集一个键值对的Tuple2。在每个元组中，键是文本中找到的单词，值是整数-(1)。

此类实现了[FlatMapFunction](https://ci.apache.org/projects/flink/flink-docs-release-1.1/api/java/org/apache/flink/api/common/functions/FlatMapFunction.html)接口，该接口以String作为输入并生成[Tuple2](https://nightlies.apache.org/flink/flink-docs-release-1.3/api/java/org/apache/flink/api/java/tuple/Tuple2.html)<String, Integer\>：

```java
public class LineSplitter implements FlatMapFunction<String, Tuple2<String, Integer>> {

    @Override
    public void flatMap(String value, Collector<Tuple2<String, Integer>> out) {
        Stream.of(value.toLowerCase().split("\\W+"))
                .filter(t -> t.length() > 0)
                .forEach(token -> out.collect(new Tuple2<>(token, 1)));
    }
}
```

我们调用[Collector](https://ci.apache.org/projects/flink/flink-docs-release-1.0/api/java/org/apache/flink/util/class-use/Collector.html)类上的collect()方法将数据在处理管道中向前推动。

我们的下一步也是最后一步是根据元组的第一个元素(单词)对其进行分组，然后对第二个元素执行求和聚合以计算单词出现的次数：

```java
public static DataSet<Tuple2<String, Integer>> startWordCount(ExecutionEnvironment env, List<String> lines) throws Exception {
    DataSet<String> text = env.fromCollection(lines);

    return text.flatMap(new LineSplitter())
        .groupBy(0)
        .aggregate(Aggregations.SUM, 1);
}
```

**我们使用三种类型的Flink转换：flatMap()、groupBy()和aggregate()**。

让我们编写一个测试来断言字数统计的实现是否按预期工作：

```java
List<String> lines = Arrays.asList(
    "This is a first sentence",
    "This is a second sentence with a one word");

DataSet<Tuple2<String, Integer>> result = WordCount.startWordCount(env, lines);

List<Tuple2<String, Integer>> collect = result.collect();
 
assertThat(collect).containsExactlyInAnyOrder(
    new Tuple2<>("a", 3), new Tuple2<>("sentence", 2), new Tuple2<>("word", 1),
    new Tuple2<>("is", 2), new Tuple2<>("this", 2), new Tuple2<>("second", 1),
    new Tuple2<>("first", 1), new Tuple2<>("with", 1), new Tuple2<>("one", 1));
```

## 6. DataStreamAPI

### 6.1 创建数据流

Apache Flink还支持通过其DataStream API处理事件流，如果我们想要开始使用事件，我们首先需要使用StreamExecutionEnvironment类：

```java
StreamExecutionEnvironment executionEnvironment
    = StreamExecutionEnvironment.getExecutionEnvironment();
```

接下来，我们可以使用来自各种来源的执行环境创建事件流。它可能是一些消息总线，如Apache Kafka，但在此示例中，我们将仅从几个字符串元素创建一个源：

```java
DataStream<String> dataStream = executionEnvironment.fromElements(
    "This is a first sentence", 
    "This is a second sentence with a one word");
```

我们可以像在普通DataSet类中一样对DataStream的每个元素应用转换：

```java
SingleOutputStreamOperator<String> upperCase = text.map(String::toUpperCase);
```

为了触发执行，我们需要调用一个接收器操作(例如print())，它将转换的结果打印到标准输出，然后调用StreamExecutionEnvironment类上的execute()方法：

```java
upperCase.print();
env.execute();
```

它将产生以下输出：

```text
1> THIS IS A FIRST SENTENCE
2> THIS IS A SECOND SENTENCE WITH A ONE WORD
```

### 6.2 事件窗口

当实时处理事件流时，有时可能需要将事件组合在一起并对这些事件的窗口应用一些计算。

假设我们有一个事件流，其中每个事件都是一对事件编号和将事件发送到我们系统的时间戳，并且我们可以容忍无序的事件，但前提是它们的延迟不超过20秒。

对于此示例，让我们首先创建一个模拟相隔几分钟的两个事件的流，并定义一个指定我们的延迟阈值的时间戳提取器：

```java
SingleOutputStreamOperator<Tuple2<Integer, Long>> windowed
        = env.fromElements(
                new Tuple2<>(16, ZonedDateTime.now().plusMinutes(25).toInstant().getEpochSecond()),
                new Tuple2<>(15, ZonedDateTime.now().plusMinutes(2).toInstant().getEpochSecond()))
        .assignTimestampsAndWatermarks(
                new BoundedOutOfOrdernessTimestampExtractor
                        <Tuple2<Integer, Long>>(Time.seconds(20)) {

                    @Override
                    public long extractTimestamp(Tuple2<Integer, Long> element) {
                        return element.f1 * 1000;
                    }
                });
```

接下来，让我们定义一个窗口操作，将事件分组为5秒窗口，并对这些事件应用转换：

```java
SingleOutputStreamOperator<Tuple2<Integer, Long>> reduced = windowed
    .windowAll(TumblingEventTimeWindows.of(Time.seconds(5)))
    .maxBy(0, true);
reduced.print();
```

它将获取每个5秒窗口的最后一个元素，因此打印出：

```java
1> (15,1491221519)
```

请注意，我们看不到第二个事件，因为它到达的时间晚于指定的延迟阈值。

## 7. 总结

在本文中，我们介绍了Apache Flink框架并研究了其API提供的一些转换。

我们使用Flink流式且功能强大的DataSet API实现了一个字数统计程序，然后我们研究了DataStream API，并在事件流上实现了一个简单的实时转换。