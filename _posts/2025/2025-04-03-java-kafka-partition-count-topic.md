---
layout: post
title:  获取Kafka中主题的分区数
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 简介

在本教程中，我们将了解检索[Kafka主题](https://www.baeldung.com/spring-kafka)的分区总数的不同方法。在简要介绍什么是Kafka分区以及我们为什么需要检索此信息之后，我们将编写Java代码来执行此操作。然后我们将了解如何使用CLI获取此信息。

## 2. Kafka分区

[Kafka主题](https://www.baeldung.com/kafka-topic-creation)可以分为多个分区，拥有多个分区的目的是为了能够同时消费来自同一主题的消息。由于消费者数量多于现有分区是没有意义的，因此**主题中的Kafka分区数代表了消费的最大并行度**。因此，提前了解给定主题有多少个分区对于正确调整各个消费者的大小非常有用。

## 3. 使用Java检索分区号

要使用Java检索特定主题的分区数，我们可以依赖[KafkaProducer.partitionFor(topic)](https://kafka.apache.org/26/javadoc/org/apache/kafka/clients/producer/KafkaProducer.html#partitionsFor-java.lang.String-)方法，此方法将返回给定主题的分区元数据：

```java
Properties producerProperties = new Properties();
// producerProperties.put("key","value") ... 
KafkaProducer<String, String> producer = new KafkaProducer<>(producerProperties)
List<PartitionInfo> info = producer.partitionsFor(TOPIC);
Assertions.assertEquals(3, info.size());
```

该方法返回的PartitionInfo列表的大小将恰好等于为特定主题配置的分区数。

如果我们无法访问Producer，我们可以使用KafkaAdminClient以稍微复杂的方式实现相同的结果：

```java
Properties props = new Properties();
// props.put("key","value") ...
AdminClient client = AdminClient.create(props)){
DescribeTopicsResult describeTopicsResult = client.describeTopics(Collections.singletonList(TOPIC));
Map<String, KafkaFuture> values = describeTopicsResult.values();
KafkaFuture topicDescription = values.get(TOPIC);
Assertions.assertEquals(3, topicDescription.get().partitions().size());
```

在本例中，我们依赖[KafkaClient.describeTopic(topic)](https://kafka.apache.org/23/javadoc/org/apache/kafka/clients/admin/AdminClient.html#describeTopics-java.util.Collection-)方法，该方法返回一个DescribeTopicsResult对象，其中包含要执行的未来任务的Map。在这里，我们仅检索所需主题的TopicDescription，最后检索分区数。

## 4. 使用CLI检索分区数量

我们有几个选项可以通过CLI检索给定主题的分区数。

首先，我们可以依赖每次安装和运行Kafka时附带的shell脚本：

```shell
$ kafka-topics --describe --bootstrap-server localhost:9092 --topic topic_name
```

此命令将输出指定主题的完整描述：

```text
Topic:topic_name        PartitionCount:3        ReplicationFactor:1     Configs: ... 
```

另一个选择是使用[Kafkacat](https://docs.confluent.io/platform/current/clients/kafkacat-usage.html)，它是Kafka的非基于JVM的消费者和生产者。使用元数据列表模式(-L)，此shell实用程序显示Kafka集群的当前状态，包括其所有主题和分区。要显示特定主题的元数据信息，我们可以运行以下命令：

```shell
$ kafkacat -L -b localhost:9092 -t topic_name
```

该命令的输出将是：

```text
Metadata for topic topic_name (from broker 1: mybroker:9092/1):
  topic "topic_name" with 3 partitions:
    partition 0, leader 3, replicas: 1,2,3, isrs: 1,2,3
    partition 1, leader 1, replicas: 1,2,3, isrs: 1,2,3
    partition 2, leader 1, replicas: 1,2, isrs: 1,2
```

我们可以看到这个shell实用程序命令如何显示有关特定主题及其分区的有用详细信息。

## 5. 总结

在这个简短的教程中，我们了解了如何使用Java和CLI检索特定Kafka主题的总分区数。

我们首先了解了为什么检索这些信息很有用，然后我们使用了KafkaProducer和KafkaAdmin。最后，我们将shell命令与Kafka脚本实用程序和KafkaCat结合使用。