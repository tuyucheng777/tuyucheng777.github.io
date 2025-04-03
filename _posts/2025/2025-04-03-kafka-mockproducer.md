---
layout: post
title:  使用Kafka MockProducer
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 概述

Kafka是一个围绕分布式消息队列构建的消息处理系统，它提供了一个Java库，以便应用程序可以将数据写入Kafka主题或从中读取数据。

现在，**由于大多数业务领域逻辑都是通过单元测试进行验证的，因此应用程序通常会在JUnit中Mock所有I/O操作**，Kafka也提供了MockProducer来Mock生产者应用程序。

在本教程中，我们将首先实现一个Kafka生产者应用程序。然后，我们将实现一个单元测试，以使用MockProducer验证常见的生产操作。

## 2. Maven依赖

在我们实现生产者应用程序之前，我们将为[kafka-clients](https://mvnrepository.com/artifact/org.apache.kafka/kafka-clients)添加Maven依赖：

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>3.4.0</version>
</dependency>
```

## 3. MockProducer

kafka-clients库包含一个用于在Kafka中发布和消费消息的Java库，生产者应用程序可以使用这些API将键值记录发送到Kafka主题：

```java
public class KafkaProducer {

    private final Producer<String, String> producer;

    public KafkaProducer(Producer<String, String> producer) {
        this.producer = producer;
    }

    public Future<RecordMetadata> send(String key, String value) {
        ProducerRecord record = new ProducerRecord("topic_sports_news", key, value);
        return producer.send(record);
    }
}
```

任何Kafka生产者都必须在客户端库中实现Producer接口。Kafka还提供了KafkaProducer类，它是对Kafka代理执行I/O操作的具体实现。

此外，Kafka提供了一个MockProducer，它实现了相同的Producer接口并模拟了KafkaProducer中实现的所有I/O操作：

```java
@Test
void givenKeyValue_whenSend_thenVerifyHistory() {

    MockProducer mockProducer = new MockProducer<>(true, new StringSerializer(), new StringSerializer());

    kafkaProducer = new KafkaProducer(mockProducer);
    Future<RecordMetadata> recordMetadataFuture = kafkaProducer.send("soccer",
            "{\"site\" : \"tuyucheng\"}");

    assertTrue(mockProducer.history().size() == 1);
}
```

尽管此类I/O操作也可以使用[Mockito](https://www.baeldung.com/mockito-series)进行Mock，但**MockProducer允许我们使用许多需要在Mock之上实现的功能**。其中一个功能是history()方法，MockProducer会缓存调用send()的记录，从而允许我们验证生产者的发布行为。

此外，我们还可以验证元数据，例如主题名称、分区、记录键或值：

```java
assertTrue(mockProducer.history().get(0).key().equalsIgnoreCase("data"));
assertTrue(recordMetadataFuture.get().partition() == 0);
```

## 4. Mock Kafka集群

到目前为止，在我们模拟的测试中，我们假设主题只有一个分区。但是，为了实现生产者和消费者线程之间的最大并发性，Kafka主题通常会被分成多个分区。

这允许生产者将数据写入多个分区，这通常是通过基于键对记录进行分区并将特定键映射到特定分区来实现的：

```java
public class EvenOddPartitioner extends DefaultPartitioner {

    @Override
    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
        if (((String)key).length() % 2 == 0) {
            return 0;
        }
        return 1;
    }
}
```

因此，所有偶数长度的键都将发布到分区“0”，同样，奇数长度的键将发布到分区“1”。

**MockProducer使我们能够通过Mock具有多个分区的Kafka集群来验证此类分区分配算法**：

```java
@Test
void givenKeyValue_whenSendWithPartitioning_thenVerifyPartitionNumber() throws ExecutionException, InterruptedException {
    PartitionInfo partitionInfo0 = new PartitionInfo(TOPIC_NAME, 0, null, null, null);
    PartitionInfo partitionInfo1 = new PartitionInfo(TOPIC_NAME, 1, null, null, null);
    List<PartitionInfo> list = new ArrayList<>();
    list.add(partitionInfo0);
    list.add(partitionInfo1);

    Cluster cluster = new Cluster("kafkab", new ArrayList<Node>(), list, emptySet(), emptySet());
    this.mockProducer = new MockProducer<>(cluster, true, new EvenOddPartitioner(),
            new StringSerializer(), new StringSerializer());

    kafkaProducer = new KafkaProducer(mockProducer);
    Future<RecordMetadata> recordMetadataFuture = kafkaProducer.send("partition",
            "{\"site\" : \"tuyucheng\"}");

    assertTrue(recordMetadataFuture.get().partition() == 1);
}
```

我们Mock了一个有2个分区(0和1)的集群，然后我们可以验证EvenOddPartitioner是否将记录发布到分区1。

## 5. 使用MockProducer Mock错误 

到目前为止，我们仅Mock生产者成功将记录发送到Kafka主题。但是，如果在写入记录时出现异常，会发生什么？

应用程序通常通过重试或向客户端抛出异常来处理此类异常。

MockProducer允许我们在send()过程中Mock异常，以便我们可以验证异常处理代码：

```java
@Test
void givenKeyValue_whenSend_thenReturnException() {
    MockProducer<String, String> mockProducer = new MockProducer<>(false, new StringSerializer(), new StringSerializer())

    kafkaProducer = new KafkaProducer(mockProducer);
    Future<RecordMetadata> record = kafkaProducer.send("site", "{\"site\" : \"tuyucheng\"}");
    RuntimeException e = new RuntimeException();
    mockProducer.errorNext(e);

    try {
        record.get();
    } catch (ExecutionException | InterruptedException ex) {
        assertEquals(e, ex.getCause());
    }
    assertTrue(record.isDone());
}
```

**这段代码中有两点值得注意**。

首先，我们调用MockProducer构造函数，并将autoComplete设置为false，这告诉MockProducer在完成send()方法之前等待输入。

其次，我们将调用mockProducer.errorNext(e)，以便MockProducer对最后的send()调用返回一个异常。

## 6. 使用MockProducer Mock事务写入 

Kafka 0.11引入了Kafka代理、生产者和消费者之间的事务。这允许Kafka中实现端到端的[Exactly-Once消息传递语义](https://www.baeldung.com/kafka-exactly-once)。简而言之，这意味着事务生产者只能使用[两阶段提交](https://www.baeldung.com/transactions-intro)协议将记录发布到代理。

MockProducer也支持事务写入并允许我们验证此行为：

```java
@Test
void givenKeyValue_whenSendWithTxn_thenSendOnlyOnTxnCommit() {
    MockProducer<String, String> mockProducer = new MockProducer<>(true, new StringSerializer(), new StringSerializer())

    kafkaProducer = new KafkaProducer(mockProducer);
    kafkaProducer.initTransaction();
    kafkaProducer.beginTransaction();
    Future<RecordMetadata> record = kafkaProducer.send("data", "{\"site\" : \"tuyucheng\"}");

    assertTrue(mockProducer.history().isEmpty());
    kafkaProducer.commitTransaction();
    assertTrue(mockProducer.history().size() == 1);
}
```

**由于MockProducer也支持与具体KafkaProducer相同的API，因此它仅在我们提交事务后才更新history记录**，这种Mock行为可以帮助应用程序验证是否为每个事务调用了commitTransaction()。

## 7. 总结

在本文中，我们研究了kafka-client库的MockProducer类。我们讨论了MockProducer实现与具体KafkaProducer相同的层次结构，因此我们可以使用Kafka代理Mock所有I/O操作。

**我们还讨论了一些复杂的Mock场景，并能够使用MockProducer测试异常、分区和事务**。