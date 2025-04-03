---
layout: post
title:  KafkaStreams简介
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 概述

在本文中，我们将研究[KafkaStreams库](https://kafka.apache.org/documentation/streams/)。

KafkaStreams由Apache Kafka的创建者设计，该软件的主要目标是允许程序员创建可以作为微服务运行的高效、实时、流式应用程序。

**KafkaStreams使我们能够从Kafka主题中消费，分析或转换数据，并可能将其发送到另一个Kafka主题**。

为了演示KafkaStreams，我们将创建一个简单的应用程序，它从主题中读取句子，计算单词的出现次数并打印每个单词的计数。

需要注意的是，KafkaStreams库不是响应式的，不支持异步操作和背压处理。

## 2. Maven依赖

要开始使用KafkaStreams编写流处理逻辑，我们需要添加[kafka-streams](https://mvnrepository.com/artifact/org.apache.kafka/kafka-streams)和[kafka-clients](https://mvnrepository.com/artifact/org.apache.kafka/kafka-clients)依赖：

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-streams</artifactId>
    <version>3.4.0</version>
</dependency>
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>3.4.0</version>
</dependency>
```

我们还需要安装并启动Apache Kafka，因为我们将使用Kafka主题，此主题将成为我们的流式传输作业的数据源。

我们可以从[官方网站](https://www.confluent.io/download/)下载Kafka和其他所需的依赖。

## 3. 配置KafkaStreams输入

**我们要做的第一件事是定义输入的Kafka主题**。

我们可以使用下载的Confluent工具-它包含一个Kafka服务器。它还包含kafka-console-producer，我们可以使用它来将消息发布到Kafka。

首先运行我们的Kafka集群：

```shell
./confluent start
```

一旦Kafka启动，我们就可以使用APPLICATION_ID_CONFIG定义我们的数据源和应用程序的名称：

```java
String inputTopic = "inputTopic";
Properties streamsConfiguration = new Properties();
streamsConfiguration.put(
    StreamsConfig.APPLICATION_ID_CONFIG, 
    "wordcount-live-test");
```

一个关键的配置参数是BOOTSTRAP_SERVER_CONFIG，这是我们刚刚启动的本地Kafka实例的URL：

```java
private String bootstrapServers = "localhost:9092";
streamsConfiguration.put(
    StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, 
    bootstrapServers);
```

接下来，我们需要传递将从inputTopic中消费消息的键和值的类型：

```java
streamsConfiguration.put(
    StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, 
    Serdes.String().getClass().getName());
streamsConfiguration.put(
    StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, 
    Serdes.String().getClass().getName());
```

**流处理往往是有状态的，当我们想要保存中间结果时，需要指定STATE_DIR_CONFIG参数**。

在我们的测试中，我们使用本地文件系统：

```java
this.stateDirectory = Files.createTempDirectory("kafka-streams");
streamsConfiguration.put(StreamsConfig.STATE_DIR_CONFIG, this.stateDirectory.toAbsolutePath().toString());
```

## 4. 构建流拓扑

**一旦我们定义了输入主题，我们就可以创建流拓扑-这是如何处理和转换事件的定义**。

在我们的示例中，我们想实现一个字数统计器。对于发送到inputTopic的每个句子，我们希望将其拆分成单词并计算每个单词的出现次数。

我们可以使用KStreamsBuilder类的实例来开始构建拓扑：

```java
StreamsBuilder builder = new StreamsBuilder();
KStream<String, String> textLines = builder.stream(inputTopic);
Pattern pattern = Pattern.compile("\\W+", Pattern.UNICODE_CHARACTER_CLASS);

KTable<String, Long> wordCounts = textLines
    .flatMapValues(value -> Arrays.asList(pattern.split(value.toLowerCase())))
    .groupBy((key, word) -> word)
    .count();
```

**要实现字数统计，首先我们需要使用正则表达式来拆分值**。

split方法返回一个数组，我们使用flatMapValues()来将其展平。否则，我们最终会得到一个数组列表，使用这种结构编写代码会很不方便。

最后，我们汇总每个单词的值并调用count()来计算特定单词的出现次数。

## 5. 处理结果

我们已经计算了输入消息的字数，**现在让我们使用foreach()方法将结果打印到标准输出上**：

```java
wordCounts.toStream()
    .foreach((word, count) -> System.out.println("word: " + word + " -> " + count));
```

在生产中，这种流式作业通常会将输出发布到另一个Kafka主题。

我们可以使用to()方法来实现这一点：

```java
String outputTopic = "outputTopic";
wordCounts.toStream()
    .to(outputTopic, Produced.with(Serdes.String(), Serdes.Long()));
```

Serde类为我们提供了Java类型的预配置序列化器，用于将对象序列化为字节数组。然后，字节数组将被发送到Kafka主题。

我们使用String作为主题的键，使用Long作为实际计数的值。to()方法会将结果数据保存到outputTopic。

## 6. 启动KafkaStream作业

至此，我们构建了一个可以执行的拓扑。但是，作业尚未开始。

**我们需要通过调用KafkaStreams实例上的start()方法来明确启动我们的作业**：

```java
Topology topology = builder.build();
KafkaStreams streams = new KafkaStreams(topology, streamsConfiguration);
streams.start();

Thread.sleep(30000);
streams.close();
```

请注意，我们正在等待30秒才能完成该作业。在实际情况下，该作业将一直运行，处理来自Kafka的事件。

**我们可以通过向Kafka主题发布一些事件来测试我们的作业**。

让我们启动一个kafka-console-producer并手动向我们的inputTopic发送一些事件：

```shell
./kafka-console-producer --topic inputTopic --broker-list localhost:9092
>"this is a pony"
>"this is a horse and pony"
```

这样，我们就向Kafka发布了两个事件，应用程序将消费这些事件并打印以下输出：

```text
word:  -> 1
word: this -> 1
word: is -> 1
word: a -> 1
word: pony -> 1
word:  -> 2
word: this -> 2
word: is -> 2
word: a -> 2
word: horse -> 1
word: and -> 1
word: pony -> 2
```

我们可以看到，当第一条消息到达时，pony这个词只出现了一次。但是当我们发送第二条消息时，pony这个词出现了第二次，打印结果为：“word: pony -> 2”。

## 7. 总结

本文讨论如何使用Apache Kafka作为数据源并使用KafkaStreams库作为流处理库来创建主要的流处理应用程序。