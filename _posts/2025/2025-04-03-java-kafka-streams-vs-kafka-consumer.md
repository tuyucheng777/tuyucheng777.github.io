---
layout: post
title:  Kafka Streams与Kafka Consumer
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 简介

[Apache Kafka](https://kafka.apache.org/)是最流行的开源分布式容错流处理系统。Kafka Consumer提供了处理消息的基本功能，**[Kafka Streams](https://www.baeldung.com/java-kafka-streams)还在Kafka Consumer客户端之上提供实时流处理**。

在本教程中，我们将解释Kafka Streams的功能，以使流处理体验变得简单而轻松。

## 2. Streams和Consumer API之间的区别

### 2.1 Kafka Consumer API

简而言之，[Kafka Consumer API](https://kafka.apache.org/documentation/#consumerapi)允许应用程序处理来自主题的消息。**它提供了与它们交互的基本组件，包括以下功能**：

- 消费者和生产者之间的责任分离
- 单一处理
- 批处理支持
- 仅支持无状态，客户端不保留先前的状态，并单独评估流中的每条记录
- 编写应用程序需要大量代码
- 不使用线程或并行
- 可以在多个Kafka集群中写入

![](/assets/images/2025/kafka/javakafkastreamsvskafkaconsumer01.png)

### 2.2 Kafka Streams API

[Kafka Streams](https://kafka.apache.org/10/documentation/streams/architecture)大大简化了主题的流处理，**它建立在Kafka客户端库之上，提供数据并行性、分布式协调、容错和可扩展性**。它将消息作为无界、连续和实时的记录流来处理，具有以下特点：

- 单一Kafka Stream用于消费和生产
- 执行复杂的处理
- 不支持批处理
- 支持无状态和有状态的操作
- 编写应用程序只需几行代码
- 多线程和并行
- 仅与单个Kafka集群交互
- 将分区和任务作为存储和传输消息的逻辑单元进行流式处理

![](/assets/images/2025/kafka/javakafkastreamsvskafkaconsumer02.png)

**Kafka Streams使用分区和任务的概念作为与主题分区紧密相关的逻辑单元**。此外，它使用线程在应用程序实例内并行处理。支持的另一个重要功能是状态存储，Kafka Streams使用它来存储和查询来自主题的数据。最后，Kafka Streams API与集群交互，但它并不直接在集群上运行。

在接下来的部分中，我们将重点关注与基本Kafka客户端不同的4个方面：流表二元性、Kafka Streams领域特定语言(DSL)、Exactly-Once处理语义(EOS)和交互式查询。

### 2.3 依赖

为了实现示例，我们只需将[Kafka Consumer API](https://mvnrepository.com/artifact/org.apache.kafka/kafka-clients)和[Kafka Streams API](https://mvnrepository.com/artifact/org.apache.kafka/kafka-streams)依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>3.4.0</version>
</dependency>
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-streams</artifactId>
    <version>3.4.0</version>
 </dependency>
```

## 3. 流表二元性

**Kafka Streams支持流，也支持可以双向转换的表，这就是所谓的[流表二元性](https://www.confluent.io/blog/kafka-streams-tables-part-1-event-streaming/#stream-table-duality)**。表是一组不断发展的事实，每个新事件都会覆盖旧事件，而流是不可变事实的集合。

流处理来自主题的完整数据流，表通过聚合来自流的信息来存储状态。让我们想象一下玩[Kafka数据建模](https://www.baeldung.com/apache-kafka-data-modeling#event-basics)中描述的国际象棋游戏，连续移动的流被聚合到一个表中，我们可以从一个状态转换到另一个状态：

![](/assets/images/2025/kafka/javakafkastreamsvskafkaconsumer03.png)

### 3.1 KStream、KTable和GlobalKTable

**Kafka Streams为Streams和Tables提供了两种抽象**。KStream处理记录流，另一方面，KTable使用给定键的最新状态管理变更日志流，每个数据记录代表一次更新。

对于未分区表，还有另一种抽象。我们可以使用GlobalKTables将信息广播到所有任务，或者进行连接，而无需对输入数据进行重新分区。

我们可以将主题作为流读取和反序列化：

```java
StreamsBuilder builder = new StreamsBuilder();
KStream<String, String> textLines = builder.stream(inputTopic, Consumed.with(Serdes.String(), Serdes.String()));
```

还可以读取主题以表格形式跟踪收到的最新单词：

```java
KTable<String, String> textLinesTable = builder.table(inputTopic, Consumed.with(Serdes.String(), Serdes.String()));
```

最后，我们可以使用全局表来读取主题：

```java
GlobalKTable<String, String> textLinesGlobalTable = builder.globalTable(inputTopic, Consumed.with(Serdes.String(), Serdes.String()));
```

## 4. Kafka Streams DSL

**Kafka Streams DSL是一种声明式和函数式编程风格**，它建立在[Streams Processor API](https://kafka.apache.org/documentation/streams/developer-guide/processor-api.html)之上，该语言为上一节提到的流和表提供了内置的抽象。

此外，它还支持无状态(map、filter等)和有状态的转换(aggregation、join和windowing)，因此只需几行代码就可以实现流处理操作。

### 4.1 无状态转换

[无状态转换](https://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#stateless-transformations)不需要状态进行处理。同样，流处理器中也不需要状态存储。示例操作包括filter、map、flatMap或groupBy。

现在让我们看看如何将值映射为大写，从主题中过滤它们并将它们存储为流：

```java
KStream<String, String> textLinesUpperCase = textLines
    .map((key, value) -> KeyValue.pair(value, value.toUpperCase()))
    .filter((key, value) -> value.contains("FILTER"));
```

### 4.2 状态转换

[状态转换](https://kafka.apache.org/documentation/streams/developer-guide/dsl-api.html#stateful-transformations)依赖于状态来完成处理操作，一条消息的处理依赖于其他消息(状态存储)的处理。换句话说，任何表或状态存储都可以使用变更日志主题进行恢复。

状态转换的一个例子是字数统计算法：

```java
KTable<String, Long> wordCounts = textLines
    .flatMapValues(value -> Arrays.asList(value
        .toLowerCase(Locale.getDefault()).split("\\W+")))
    .groupBy((key, word) -> word)
        .count(Materialized.<String, Long, KeyValueStore<Bytes, byte[]>> as("counts-store"));
```

我们将把这两个字符串发送到主题：

```java
String TEXT_EXAMPLE_1 = "test test and test";
String TEXT_EXAMPLE_2 = "test filter filter this sentence";
```

结果是：

```text
Word: and -> 1
Word: test -> 4
Word: filter -> 2
Word: this -> 1
Word: sentence -> 1
```

DSL涵盖了多种转换功能，我们可以拼接或合并具有相同键的两个输入流/表以生成新的流/表。我们还可以聚合或将来自流/表的多个记录合并为新表中的单个记录。最后，可以在拼接或聚合函数中应用windowing以对具有相同键的记录进行分组。

使用5s窗口进行拼接的示例将按键分组的来自两个流的记录合并到一个流中：

```java
KStream<String, String> leftRightSource = leftSource.outerJoin(rightSource, (leftValue, rightValue) -> "left=" + leftValue + ", right=" + rightValue, JoinWindows.of(Duration.ofSeconds(5))).groupByKey()
        .reduce(((key, lastValue) -> lastValue))
  .toStream();
```

因此，我们输入左流value=left和key=1，右流value=right和key=2。结果如下：

```text
(key= 1) -> (left=left, right=null)
(key= 2) -> (left=null, right=right)
```

对于聚合示例，我们将计算字数统计算法，但使用每个单词的前两个字母作为键：

```java
KTable<String, Long> aggregated = input
    .groupBy((key, value) -> (value != null && value.length() > 0)
        ? value.substring(0, 2).toLowerCase() : "",
        Grouped.with(Serdes.String(), Serdes.String()))
    .aggregate(() -> 0L, (aggKey, newValue, aggValue) -> aggValue + newValue.length(),
        Materialized.with(Serdes.String(), Serdes.Long()));
```

包含以下条目：

```text
"one", "two", "three", "four", "five"
```

输出为：

```text
Word: on -> 3
Word: tw -> 3
Word: th -> 5
Word: fo -> 4
Word: fi -> 4
```

## 5. 精确一次处理语义(EOS)

在某些情况下，我们需要确保消费者只读取一次消息。Kafka引入了将消息包含在事务中的功能，以便使用[Transactional API](https://www.baeldung.com/kafka-exactly-once)实现EOS。从0.11.0版本开始，Kafka Streams就涵盖了同样的功能。

要在Kafka Streams中配置EOS，我们将包含以下属性：

```java
streamsConfiguration.put(StreamsConfig.PROCESSING_GUARANTEE_CONFIG, StreamsConfig.EXACTLY_ONCE);
```

## 6.交互式查询

**[交互式查询](https://kafka.apache.org/documentation/streams/developer-guide/interactive-queries.html)允许在分布式环境中查询应用程序的状态**，这意味着能够从本地存储中提取信息，也可以从多个实例上的远程存储中提取信息。基本上，我们将收集所有存储并将它们组合在一起以获取应用程序的完整状态。

让我们看一个使用交互式查询的示例。首先，我们将定义处理拓扑，在我们的例子中是字数统计算法：

```java
KStream<String, String> textLines = builder.stream(TEXT_LINES_TOPIC, Consumed.with(Serdes.String(), Serdes.String()));

final KGroupedStream<String, String> groupedByWord = textLines
    .flatMapValues(value -> Arrays.asList(value.toLowerCase().split("\\W+")))
    .groupBy((key, word) -> word, Grouped.with(stringSerde, stringSerde));
```

接下来，我们将为所有计算的字数创建一个状态存储(键值)：

```java
groupedByWord
    .count(Materialized.<String, Long, KeyValueStore<Bytes, byte[]>>as("WordCountsStore")
    .withValueSerde(Serdes.Long()));
```

然后，我们可以查询键值存储：

```java
ReadOnlyKeyValueStore<String, Long> keyValueStore = streams.store(StoreQueryParameters.fromNameAndType(
    "WordCountsStore", QueryableStoreTypes.keyValueStore()));

KeyValueIterator<String, Long> range = keyValueStore.all();
while (range.hasNext()) {
    KeyValue<String, Long> next = range.next();
    System.out.println("count for " + next.key + ": " + next.value);
}
```

该示例的输出如下：

```text
Count for and: 1
Count for filter: 2
Count for sentence: 1
Count for test: 4
Count for this: 1
```

## 7. 总结

在本教程中，我们展示了Kafka Streams如何简化从Kafka主题检索消息时的处理操作。它大大简化了处理Kafka中的流时的实现，不仅适用于无状态处理，也适用于有状态转换。

当然，不使用Kafka Streams也可以完美地构建消费者应用程序，但我们需要手动实现免费提供的一系列额外功能。