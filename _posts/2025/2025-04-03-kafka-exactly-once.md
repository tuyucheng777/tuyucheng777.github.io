---
layout: post
title:  使用Java在Kafka中进行精确一次处理
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 概述

在本教程中，我们将了解K**afka如何通过新引入的Transactional API确保生产者和消费者应用程序之间的一次性交付**。

此外，我们将使用此API来实现事务生产者和消费者，以在WordCount示例中实现端到端的一次性交付。

## 2. Kafka中的消息传递

由于各种故障，消息传递系统无法保证生产者和消费者应用程序之间的消息传递。根据客户端应用程序与此类系统的交互方式，可能出现以下消息语义：

- 如果一个消息系统永远不会重复一条消息，但可能会偶尔漏掉一条消息，我们称之为最多一次
- 或者，如果它永远不会错过任何消息，但可能会重复偶尔的消息，我们称之为至少一次
- **但是，如果它始终传递所有消息而不重复，那么这就是恰好一次**

最初，Kafka仅支持最多一次和至少一次消息传递。

但是，**在Kafka代理和客户端应用程序之间引入事务可确保Kafka中的一次性交付**。为了更好地理解它，让我们快速回顾一下事务客户端API。

## 3. Maven依赖

要使用事务API，我们需要在pom中添加[Kafka的Java客户端](https://mvnrepository.com/artifact/org.apache.kafka/kafka-clients)：

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>3.9.0</version>
</dependency>
```

## 4. 事务性的消费-转换-生产循环

为了举个例子，我们将消费来自输入主题“sentences”的消息。

然后，对于每个句子，我们将计算每个单词，并将各个单词的计数发送到输出主题counts。

在示例中，我们假设sentences主题中已经有事务数据。

### 4.1 具有事务意识的生产者

因此，我们首先添加一个典型的Kafka生产者。

```java
Properties producerProps = new Properties();
producerProps.put("bootstrap.servers", "localhost:9092");
```

另外，我们还需要指定transactional.id并启用idempotence：

```java
producerProps.put("enable.idempotence", "true");
producerProps.put("transactional.id", "prod-1");

KafkaProducer<String, String> producer = new KafkaProducer(producerProps);
```

因为我们已经启用了幂等性，所以Kafka将使用此事务ID作为其算法的一部分来对该生产者发送的任何消息进行重复数据删除，从而确保幂等性。

简单来说，如果生产者不小心多次向Kafka发送相同的消息，这些设置可以让它注意到。

**我们只需要确保每个生产者的事务ID都是不同的**，但在重启时保持一致。

### 4.2 启用生产者事务

一旦我们准备就绪，我们还需要调用initTransaction来准备生产者使用事务：

```java
producer.initTransactions();
```

这将向代理注册生产者，使其可以使用事务，并**通过其transactional.id和序列号(或epoch)对其进行标识**。反过来，代理将使用这些将任何操作预先写入事务日志。

因此，**代理将从该日志中删除属于具有相同事务ID和更早时期的生产者的任何操作**，假定它们来自已失效的事务。

### 4.3 具有事务意识的消费者

当我们消费时，我们可以按顺序读取主题分区上的所有消息。但是，**我们可以通过isolation.level指示我们应该等待，直到相关事务提交后才能读取事务消息**：

```java
Properties consumerProps = new Properties();
consumerProps.put("bootstrap.servers", "localhost:9092");
consumerProps.put("group.id", "my-group-id");
consumerProps.put("enable.auto.commit", "false");
consumerProps.put("isolation.level", "read_committed");
KafkaConsumer<String, String> consumer = new KafkaConsumer<>(consumerProps);
consumer.subscribe(singleton(“sentences”));
```

**使用read_committed的值可确保我们在事务完成之前不会读取任何事务消息**。

isolation.level的默认值是read_uncommitted。

### 4.4 通过事务进行消费和转换

现在我们已经将生产者和消费者都配置为以事务方式写入和读取，我们可以从输入主题中消费记录并计算每条记录中的每个单词：

```java
ConsumerRecords<String, String> records = consumer.poll(ofSeconds(60));
Map<String, Integer> wordCountMap = records.records(new TopicPartition("input", 0))
    .stream()
    .flatMap(record -> Stream.of(record.value().split(" ")))
    .map(word -> Tuple.of(word, 1))
    .collect(Collectors.toMap(tuple -> tuple.getKey(), t1 -> t1.getValue(), (v1, v2) -> v1 + v2));
```

请注意，上述代码不是事务性的。但**由于我们使用了read_committed，这意味着该消费者不会读取在同一事务中写入输入主题的消息，直到它们全部写入为止**。

现在，我们可以将计算出的字数发送到输出主题。

让我们看看如何以事务的方式产生结果。

### 4.5 Send API

为了将计数作为新消息发送，但在同一事务中，我们调用beginTransaction：

```java
producer.beginTransaction();
```

然后，我们可以将每一个写入我们的“counts”主题，其中键是单词，计数是值：

```java
wordCountMap.forEach((key,value) -> 
    producer.send(new ProducerRecord<String,String>("counts",key,value.toString())));
```

请注意，由于生产者可以按键对数据进行分区，**因此事务消息可以跨多个分区，每个分区由不同的消费者读取**。因此，Kafka代理将存储事务的所有更新分区的列表。

还要注意，在一个事务中，生产者可以使用多个线程并行发送记录。

### 4.6 提交偏移量

最后，我们需要提交刚刚消费完的偏移量。**通过事务，我们将偏移量提交回我们从中读取它们的输入主题，就像平常一样。我们还将它们发送到生产者的事务**。

我们可以在一次调用中完成所有这些操作，但我们首先需要计算每个主题分区的偏移量：

```java
Map<TopicPartition, OffsetAndMetadata> offsetsToCommit = new HashMap<>();
for (TopicPartition partition : records.partitions()) {
    List<ConsumerRecord<String, String>> partitionedRecords = records.records(partition);
    long offset = partitionedRecords.get(partitionedRecords.size() - 1).offset();
    offsetsToCommit.put(partition, new OffsetAndMetadata(offset + 1));
}
```

**请注意，我们对事务承诺的是即将到来的偏移量，这意味着我们需要加1**。

然后，我们可以将计算出的偏移量发送到事务：

```java
producer.sendOffsetsToTransaction(offsetsToCommit, new ConsumerGroupMetadata("my-group-id"));
```

### 4.7 提交或中止事务

最后，我们可以提交事务，它将原子地将偏移量写入consumer_offsets主题以及事务本身：

```java
producer.commitTransaction();
```

这会将任何缓冲的消息刷新到相应的分区。此外，Kafka代理还会将该事务中的所有消息提供给消费者。

当然，如果我们在处理过程中出现任何问题，例如捕获到异常，我们可以调用abortTransaction：

```java
try {
    // ... read from input topic
    // ... transform
    // ... write to output topic
    producer.commitTransaction();
} catch ( Exception e ) {
    producer.abortTransaction();
}
```

删除所有缓冲的消息并从代理中删除事务。

**如果我们在代理配置的max.transaction.timeout.ms之前既不提交也不中止，Kafka代理将自行中止事务**。此属性的默认值为900000毫秒或15分钟。

## 5. 其他消费-转化-生产循环

我们刚刚看到了对同一个Kafka集群进行读写的基本消费-转换-生产循环。

相反，**必须读写不同Kafka集群的应用程序必须使用较旧的commitSync和commitAsync API**。通常，应用程序会将消费者偏移量存储到其外部状态存储中以维持事务性。

## 6. 总结

对于数据关键型应用程序，端到端的一次性处理通常是必不可少的。

在本教程中，我们了解了如何使用Kafka来实现这一点，即使用事务，并且我们实现了一个基于事务的字数统计示例来说明该原理。