---
layout: post
title:  使用Kafka、Spark Streaming和Cassandra构建数据管道
category: apache
copyright: apache
excerpt: Apache Spark
---

## 1. 概述

[Apache Kafka](https://kafka.apache.org/)是一个可扩展、高性能、低延迟的平台，**允许像消息传递系统一样读取和写入数据流**。我们可以相当轻松地[在Java中开始使用Kafka](https://www.baeldung.com/spring-kafka)。

[Spark Streaming](https://spark.apache.org/streaming/)是[Apache Spark](https://spark.apache.org/docs/3.5.3/streaming-programming-guide.html)平台的一部分，**可实现可扩展、高吞吐量、容错的数据流处理**。虽然Spark是用Scala编写的，但它也提供了[Java API](https://www.baeldung.com/apache-spark)来与之配合使用。

[Apache Cassandra](http://cassandra.apache.org/)是一个**分布式、宽列NoSQL数据存储**。有关[Cassandra的更多详细信息](https://www.baeldung.com/cassandra-with-java)，请参阅我们之前的文章。

在本教程中，我们将结合这些内容**为实时数据流创建高度可扩展且容错的数据管道**。

## 2. 安装

首先，我们需要在本地机器上安装Kafka、Spark和Cassandra来运行该应用程序，我们将逐步了解如何使用这些平台开发[数据管道](https://www.baeldung.com/cs/data-pipelines)。

但是，我们将保留所有安装的所有默认配置(包括端口)，这将有助于使教程顺利运行。

### 2.1 Kafka

在本地机器上安装Kafka相当简单，可以在[官方文档](https://kafka.apache.org/quickstart)中找到，我们将使用Kafka 2.1.0版本。

此外，**Kafka需要[Apache Zookeeper](https://zookeeper.apache.org/)才能运行**，但出于本教程的目的，我们将利用与Kafka打包的单节点Zookeeper实例。

一旦我们按照官方指南在本地启动了Zookeeper和Kafka，我们就可以继续创建名为“messages”的主题：

```shell
 $KAFKA_HOME$\bin\windows\kafka-topics.bat --create \
  --zookeeper localhost:2181 \
  --replication-factor 1 --partitions 1 \
  --topic messages
```

请注意，上述脚本适用于Windows平台，但也有类似的脚本可用于类Unix平台。

### 2.2 Spark

Spark使用Hadoop的HDFS和YARN客户端库，因此，**整合所有这些库的兼容版本可能非常棘手**。不过，[Spark的官方下载](https://spark.apache.org/docs/3.5.3/streaming-programming-guide.html)已预装了常用的Hadoop版本。在本教程中，我们将使用“为Apache Hadoop 2.7及更高版本预构建”的2.3.0版本软件包。

一旦解压了正确的Spark包，就可以使用可用的脚本提交应用程序，稍后我们将在Spring Boot中开发应用程序时看到这一点。

### 2.3 Cassandra

DataStax为包括Windows在内的不同平台提供了Cassandra社区版，我们可以按照[官方文档](https://www.datastax.com/what-is/cassandra)轻松下载并安装到本地机器上，本文将使用3.9.0版本。

在本地机器上安装并启动Cassandra后，我们就可以创建键空间和表了，这可以使用安装包中自带的CQL Shell来完成：

```shell
CREATE KEYSPACE vocabulary
    WITH REPLICATION = {
        'class' : 'SimpleStrategy',
        'replication_factor' : 1
    };
USE vocabulary;
CREATE TABLE words (word text PRIMARY KEY, count int);
```

请注意，我们创建了一个名为“vocabulary”的命名空间，并在其中创建了一个名为“words”的表，该表包含两列：word和count。

## 3. 依赖

我们可以通过Maven将Kafka和Spark依赖集成到应用程序中，我们将从Maven Central中提取这些依赖：

- [Core Spark](https://mvnrepository.com/artifact/org.apache.spark/spark-core)
- [SQL Spark](https://mvnrepository.com/artifact/org.apache.spark/spark-sql)
- [Streaming Spark](https://mvnrepository.com/artifact/org.apache.spark/spark-streaming)
- [Streaming Kafka Spark](https://mvnrepository.com/artifact/org.apache.spark/spark-streaming-kafka-0-10)
- [Cassandra Spark](https://mvnrepository.com/artifact/com.datastax.spark/spark-cassandra-connector)
- [Cassandra Java Spark](https://mvnrepository.com/artifact/com.datastax.spark/spark-cassandra-connector-java)

我们可以相应地将它们添加到pom中：

```xml
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-core_2.11</artifactId>
    <version>2.3.0</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-sql_2.11</artifactId>
    <version>2.3.0</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-streaming_2.11</artifactId>
    <version>2.3.0</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-streaming-kafka-0-10_2.11</artifactId>
    <version>2.3.0</version>
</dependency>
<dependency>
    <groupId>com.datastax.spark</groupId>
    <artifactId>spark-cassandra-connector_2.11</artifactId>
    <version>2.3.0</version>
</dependency>
<dependency>
    <groupId>com.datastax.spark</groupId>
    <artifactId>spark-cassandra-connector-java_2.11</artifactId>
    <version>1.5.2</version>
</dependency>
```

**请注意，其中一些依赖在范围内标记为“provided”**，这是因为这些依赖将在Spark安装过程中提供，我们将使用spark-submit提交应用程序以供执行。

## 4. Spark Streaming-Kafka集成策略

说到这里，值得简单聊一下Spark和Kafka的集成策略。

**Kafka在0.8和0.10版本之间引入了新的消费者API**，因此，两个Broker版本均提供相应的Spark Streaming软件包，根据可用的Broker和所需的功能选择合适的软件包至关重要。

### 4.1 Spark Streaming Kafka 0.8

0.8版本是稳定的集成API，**提供基于Receiver或直接方法两种选择**。关于这些方法的详细信息，我们不再赘述，详情请参阅[官方文档](https://downloads.apache.org/spark/docs/2.2.2/streaming-kafka-0-8-integration.html)。需要注意的是，此软件包兼容Kafka Broker 0.8.2.1或更高版本。

### 4.2 Spark Streaming Kafka 0.10

该软件包目前处于实验状态，仅与Kafka Broker 0.10.0或更高版本兼容。**此软件包仅提供直接方法，现在使用新的Kafka消费者API**。我们可以在[官方文档](https://spark.apache.org/docs/latest/streaming-kafka-0-10-integration.html)中找到更多详细信息，重要的是，**它与旧版Kafka Broker不向后兼容**。

请注意，本教程将使用0.10版本的软件包，上一节中提到的依赖仅指此版本。

## 5. 开发数据管道

我们将使用Spark用Java创建一个简单的应用程序，它将与我们之前创建的Kafka主题集成。该应用程序将读取已发布的消息，并计算每条消息中单词的频率。然后，这些数据将更新到我们之前创建的Cassandra表中。

让我们快速直观地看一下数据如何流动：

![](/assets/images/2025/apache/kafkasparkdatapipeline01.png)

### 5.1 获取JavaStreamingContext

首先，我们将初始化**JavaStreamingContext，它是所有Spark Streaming应用程序的入口点**：

```java
SparkConf sparkConf = new SparkConf();
sparkConf.setAppName("WordCountingApp");
sparkConf.set("spark.cassandra.connection.host", "127.0.0.1");

JavaStreamingContext streamingContext = new JavaStreamingContext(sparkConf, Durations.seconds(1));
```

### 5.2 从Kafka获取DStream

现在，我们可以从JavaStreamingContext连接到Kafka主题：

```java
Map<String, Object> kafkaParams = new HashMap<>();
kafkaParams.put("bootstrap.servers", "localhost:9092");
kafkaParams.put("key.deserializer", StringDeserializer.class);
kafkaParams.put("value.deserializer", StringDeserializer.class);
kafkaParams.put("group.id", "use_a_separate_group_id_for_each_stream");
kafkaParams.put("auto.offset.reset", "latest");
kafkaParams.put("enable.auto.commit", false);
Collection<String> topics = Arrays.asList("messages");

JavaInputDStream<ConsumerRecord<String, String>> messages = KafkaUtils.createDirectStream(
    streamingContext, 
    LocationStrategies.PreferConsistent(), 
    ConsumerStrategies.<String, String> Subscribe(topics, kafkaParams));
```

请注意，我们必须在此处为键和值提供反序列化器，**对于像String这样的常见数据类型，反序列化器默认可用**。但是，如果我们希望检索自定义数据类型，则必须提供自定义反序列化器。

这里我们获取到了JavaInputDStream，**它是离散流(Discretized Streams，简称DStreams)的一个实现，是Spark Streaming提供的基本抽象**，DStreams内部其实就是一系列连续的RDD。

### 5.3 处理获取的DStream

现在我们将对JavaInputDStream执行一系列操作来获取消息中的词频：

```java
JavaPairDStream<String, String> results = messages
    .mapToPair( 
        record -> new Tuple2<>(record.key(), record.value())
    );
JavaDStream<String> lines = results
    .map(
        tuple2 -> tuple2._2()
    );
JavaDStream<String> words = lines
    .flatMap(
        x -> Arrays.asList(x.split("\\s+")).iterator()
    );
JavaPairDStream<String, Integer> wordCounts = words
    .mapToPair(
        s -> new Tuple2<>(s, 1)
    ).reduceByKey(
        (i1, i2) -> i1 + i2
        );
```

### 5.4 将处理过的DStream持久化到Cassandra中

最后，我们可以遍历已处理过的JavaPairDStream并将它们插入到Cassandra表中：

```java
wordCounts.foreachRDD(
    javaRdd -> {
        Map<String, Integer> wordCountMap = javaRdd.collectAsMap();
        for (String key : wordCountMap.keySet()) {
            List<Word> wordList = Arrays.asList(new Word(key, wordCountMap.get(key)));
            JavaRDD<Word> rdd = streamingContext.sparkContext().parallelize(wordList);
            javaFunctions(rdd).writerBuilder("vocabulary", "words", mapToRow(Word.class)).saveToCassandra();
        }
    }
  );
```

### 5.5 运行应用程序

由于这是一个流处理应用程序，我们希望保持其运行：

```java
streamingContext.start();
streamingContext.awaitTermination();
```

## 6. 利用检查点

在流处理应用程序中，**保留正在处理的数据批次之间的状态通常很有用**。

例如，在我们之前的尝试中，我们只能存储单词的当前频率。如果我们想存储累积频率怎么办？**Spark Streaming通过名为“检查点”的概念实现了这一点**。

我们现在将修改之前创建的管道以利用检查点：

![](/assets/images/2025/apache/kafkasparkdatapipeline02.png)

请注意，我们仅在数据处理会话中使用检查点，这不提供容错功能，**但是，检查点也可以用于容错**。

为了利用检查点，我们需要对应用程序进行一些更改，这包括为JavaStreamingContext提供检查点位置：

```java
streamingContext.checkpoint("./.checkpoint");
```

这里我们使用本地文件系统来存储检查点，但是，为了保证稳定性，检查点应该存储在HDFS、S3或Kafka等位置。更多相关信息，请参阅[官方文档](https://spark.apache.org/docs/2.2.0/streaming-programming-guide.html#checkpointing)。

接下来，我们必须获取检查点并在使用映射函数处理每个分区时创建单词的累积计数：

```java
JavaMapWithStateDStream<String, Integer, Integer, Tuple2<String, Integer>> cumulativeWordCounts = wordCounts
    .mapWithState(
        StateSpec.function( 
            (word, one, state) -> {
                int sum = one.orElse(0) + (state.exists() ? state.get() : 0);
                Tuple2<String, Integer> output = new Tuple2<>(word, sum);
                state.update(sum);
                return output;
            }
          )
        );
```

一旦我们获得累积字数，我们就可以像以前一样继续迭代并将它们保存在Cassandra中。

请注意，**虽然数据检查点对于状态处理很有用，但它会带来延迟成本**。因此，有必要明智地使用它以及最佳的检查点间隔。

## 7. 理解偏移量

如果我们回想一下之前设置的一些Kafka参数：

```java
kafkaParams.put("auto.offset.reset", "latest");
kafkaParams.put("enable.auto.commit", false);
```

**这些基本上意味着我们不想自动提交偏移量，而是希望在每次初始化消费者组时选择最新的偏移量**。因此，我们的应用程序只能消费在其运行期间发布的消息。

如果我们想要消费所有发布的消息(无论应用程序是否正在运行)，并且还想要跟踪已经发布的消息，**我们必须适当地配置偏移量以及保存偏移量状态**，尽管这有点超出本教程的范围。

**这也是Spark Streaming提供类似“恰好一次”特定级别保证的方式**，这意味着发布到Kafka主题的每条消息只会被Spark Streaming处理一次。

## 8. 部署应用程序

我们可以**使用Spark安装中预先打包的Spark-submit脚本来部署我们的应用程序**：

```shell
$SPARK_HOME$\bin\spark-submit \
  --class cn.tuyucheng.taketoday.data.pipeline.WordCountingAppWithCheckpoint \
  --master local[2] 
  \target\spark-streaming-app-1.0.0-jar-with-dependencies.jar
```

请注意，我们使用Maven创建的jar应该包含范围未标记为provided的依赖。

一旦我们提交此应用程序并在我们之前创建的Kafka主题中发布一些消息，我们就应该看到在我们之前创建的Cassandra表中发布的累积字数。

## 9. 总结

在本教程中，我们学习了如何使用Kafka、Spark Streaming和Cassandra创建一个简单的数据管道，并学习了如何利用Spark Streaming中的检查点来维护批次之间的状态。