---
layout: post
title:  使用Kafka Consumer API从头读取数据
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 简介

[Apache Kafka](https://www.baeldung.com/spring-kafka)是一个开源分布式事件流处理系统，它基本上是一个事件流平台，可以发布、订阅、存储和处理记录流。

Kafka为实时数据处理提供了一个高吞吐量和低延迟的平台。基本上，**Kafka实现了发布者-订阅者模型，其中生产者应用程序将事件发布到Kafka，而消费者应用程序订阅这些事件**。

在本教程中，我们将学习如何使用Kafka Consumer API从Kafka主题的开头读取数据。

## 2. 设置

在开始之前，让我们首先设置依赖，初始化Kafka集群连接，并向Kafka发布一些消息。

Kafka提供了方便的Java客户端库，我们可以使用它对Kafka集群进行各种操作。

### 2.1 依赖

首先，让我们将Kafka Clients Java库的[Maven依赖](https://mvnrepository.com/artifact/org.apache.kafka/kafka-clients)添加到项目pom.xml文件中：

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>3.4.0</version>
</dependency>
```

### 2.2 集群和主题初始化

在整个指南中，我们假设Kafka集群在我们的本地系统上使用默认配置运行。

其次，我们需要创建一个Kafka主题，用于发布和消费消息。参考我们的[Kafka主题创建](https://www.baeldung.com/kafka-topic-creation)指南，创建一个名为“tuyucheng”的Kafka主题。

现在我们已经启动运行了Kafka集群，并创建了主题，让我们向Kafka发布一些消息。

### 2.3 发布消息

最后，让我们向Kafka主题“tuyucheng”发布一些虚拟消息。

为了发布消息，让我们创建一个KafkaProducer实例，并使用Properties实例定义的基本配置：

```java
Properties producerProperties = new Properties();
producerProperties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, KAFKA_CONTAINER.getBootstrapServers());
producerProperties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
producerProperties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

KafkaProducer<String, String> producer = new KafkaProducer<>(producerProperties);
```

我们使用KafkaProducer.send(ProducerRecord)方法将消息发布到Kafka主题“tuyucheng”：

```java
for (int i = 1; i <= 10; i++) {
    ProducerRecord<String, String> record = new ProducerRecord<>("tuyucheng", String.valueOf(i));
    producer.send(record);
}
```

在这里，我们向Kafka集群发布了10条消息，我们将消费这些消息来演示我们的消费者实现。

## 3. 从头开始消费消息

到目前为止，我们已经初始化了Kafka集群，并向Kafka主题发布了一些示例消息。接下来，让我们看看如何从头开始读取消息。

为了演示这一点，我们首先使用Properties实例定义的一组特定消费者属性初始化KafkaConsumer实例。然后，我们使用创建的KafkaConsumer实例来消费消息并再次返回到分区偏移量的开始位置。

让我们详细地看一下每个步骤。

### 3.1 消费者属性

为了从Kafka主题的开头消费消息，我们创建一个KafkaConsumer实例，该实例具有随机生成的消费者组ID。我们通过将消费者的“group.id”属性设置为随机生成的UUID来实现这一点：

```java
Properties consumerProperties = new Properties();
consumerProperties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
consumerProperties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
consumerProperties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
consumerProperties.put(ConsumerConfig.GROUP_ID_CONFIG, UUID.randomUUID().toString());
```

当我们为消费者生成新的消费者组ID时，消费者将始终属于由“group.id”属性标识的新消费者组，新消费者组不会有任何与之关联的偏移量。在这种情况下，**Kafka提供了一个属性“auto.offset.reset”，指示当Kafka中没有初始偏移量或服务器上不再存在当前偏移量时应该做什么**。

“auto.offset.reset”属性接收以下值：

- earliest：此值会自动将偏移量重置为最早的偏移量
- latest：此值会自动将偏移量重置为最新偏移量
- none：如果未找到消费者组的先前偏移量，则此值将向消费者抛出异常
- 其他任何值：如果设置了除前三个值以外的任何其他值，则会向消费者抛出异常

**由于我们想要从Kafka主题的开头读取，因此我们将“auto.offset.reset”属性的值设置为“earliest”**：

```java
consumerProperties.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
```

现在让我们使用消费者属性创建KafkaConsumer的实例：

```java
KafkaConsumer<String, String> consumer = new KafkaConsumer<>(consumerProperties);
```

我们使用这个KafkaConsumer实例从主题开始的地方使用消息。

### 3.2 消费消息

为了消费消息，我们首先订阅消费者来消费来自主题“tuyucheng”的消息：

```java
consumer.subscribe(Arrays.asList("tuyucheng"));
```

接下来，**我们使用KafkaConsumer.poll(Duration duration)方法轮询来自主题“tuyucheng”的新消息，直到Duration参数指定的时间**：

```java
ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(10));

for (ConsumerRecord<String, String> record : records) {
    logger.info(record.value());
}
```

至此，我们从“tuyucheng”主题开始就读取了所有消息。

此外，**为了重置现有消费者从主题的开头读取，我们使用KafkaConsumer.seekToBeginning(Collection<TopicPartition\> partitions)方法**。此方法接收TopicPartition的集合，并将消费者的偏移量指向分区的开头：

```java
consumer.seekToBeginning(consumer.assignment());
```

这里，我们将KafkaConsumer.assignment()的值传递给seekToBeginning()方法，KafkaConsumer.assignment()方法返回当前分配给消费者的分区集。

最后，再次轮询同一个消费者以获取消息，现在从分区的开头读取所有消息：

```java
ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(10));

for (ConsumerRecord<String, String> record : records) {
    logger.info(record.value());
}
```

## 4. 总结

在本文中，我们学习了如何使用Kafka Consumer API从Kafka主题的开头读取消息。

我们首先了解新消费者如何从Kafka主题的开头读取消息，以及其实现。然后，我们了解了已在消费的消费者如何查找其偏移量以从开头读取消息。