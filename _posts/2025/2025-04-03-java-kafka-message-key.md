---
layout: post
title:  向Kafka发送消息时是否需要键
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 简介

[Apache Kafka](https://www.baeldung.com/spring-kafka)是一个开源的分布式流处理系统，具有容错能力，吞吐量高。**Kafka基本上是一个实现发布者-订阅者模型的消息系统**。Kafka的消息传递、存储和流处理功能使我们能够大规模存储和分析实时数据流。

在本教程中，我们首先了解Kafka消息中键的重要性。然后，我们将了解如何使用键将消息发布到Kafka主题。

## 2. Kafka消息中Key的意义

我们知道，Kafka按照我们生成记录的顺序有效地存储记录流。

**当我们将消息发布到Kafka主题时，该消息会以循环方式分发到可用分区中**。因此，在Kafka主题中，消息的顺序在分区内得到保证，但不能跨分区保证。

当我们将带有键的消息发布到Kafka主题时，Kafka保证将所有具有相同键的消息存储在同一分区中。因此，如果我们想保持具有相同键的消息的顺序，Kafka消息中的键就很有用。

**总而言之，键不是向Kafka发送消息的必需部分。基本上，如果我们希望使用相同的键保持消息的严格顺序，那么我们绝对应该使用带有键的消息。对于所有其他情况，使用空键将在分区之间提供更好的消息分布**。

接下来，让我们直接深入研究一些带有键的Kafka消息的实现代码。

## 3. 设置

在开始之前，让我们首先初始化一个Kafka集群，设置依赖关系，并初始化与Kafka集群的连接。

Kafka的Java库提供了易于使用的生产者和消费者API，我们可以使用它来发布和消费来自Kafka的消息。

### 3.1 依赖

首先，让我们将Kafka Clients Java库的[Maven依赖](https://mvnrepository.com/artifact/org.apache.kafka/kafka-clients)添加到项目pom.xml文件中：

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>3.4.0</version>
</dependency>
```

### 3.2 集群和主题初始化

其次，我们需要一个正在运行的Kafka集群，以便我们可以连接并执行各种Kafka操作。本指南假设Kafka集群在我们的本地系统上运行，并采用默认配置。

最后，我们将创建一个具有多个分区的Kafka主题，可用于发布和使用消息。参考我们的[Kafka主题创建](https://www.baeldung.com/kafka-topic-creation)指南，让我们创建一个名为“tuyucheng”的主题：

```java
Properties adminProperties = new Properties();
adminProperties.put(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");

Admin admin = Admin.create(adminProperties);
```

这里我们创建了一个KafkaAdmin实例，其基本配置由[Properties](https://www.baeldung.com/java-properties)实例定义。接下来，我们将使用此Admin实例创建一个名为“tuyucheng”的主题，其中包含5个分区：

```java
admin.createTopics(Collections.singleton(new NewTopic("tuyucheng", 5, (short) 1)));
```

现在我们已经使用主题初始化了Kafka集群设置，让我们使用键发布一些消息。

## 4. 使用键发布消息

为了演示我们的编码示例，我们首先创建一个KafkaProducer实例，该实例由Properties实例定义一些基本的生产者属性。接下来，我们将使用创建的KafkaProducer实例发布带有键的消息并验证主题分区。

让我们深入详细地了解每个步骤。

### 4.1 初始化生产者

首先，让我们创建一个新的Properties实例，该实例保存生产者的属性，以便连接到我们的本地代理：

```java
Properties producerProperties = new Properties();
producerProperties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
producerProperties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
producerProperties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
```

此外，让我们使用创建的生产者的Properties实例来创建KafkaProducer的实例：

```java
KafkaProducer <String, String> producer = new KafkaProducer<>(producerProperties);
```

KafkaProducer类的构造函数接收Properties对象(或Map)并返回KafkaProducer的实例。

### 4.2 发布消息

Kafka Publisher API提供了多个构造函数来创建带有键的ProducerRecord实例，**我们使用ProducerRecord<K,V\>(String topic, K key, V value)构造函数来创建带有键的消息**：

```java
ProducerRecord<String, String> record = new ProducerRecord<>("tuyucheng", "message-key", "Hello World");
```

在这里，我们用一个键为“tuyucheng”主题创建了一个ProducerRecord实例。

现在，让我们向Kafka主题发布一些消息并验证分区：

```java
for (int i = 1; i <= 10; i++) {
    ProducerRecord<String, String> record = new ProducerRecord<>("tuyucheng", "message-key", String.valueOf(i));
    Future<RecordMetadata> future = producer.send(record);
    RecordMetadata metadata = future.get();

    logger.info(String.valueOf(metadata.partition()));
}
```

我们使用KafkaProducer.send(ProducerRecord<String, String\> record)方法向Kafka发布消息，该方法返回RecordMetadata类型的[Future](https://www.baeldung.com/java-future)实例。然后，我们使用对Future<RecordMetadata\>.get()方法的阻塞调用，该方法在消息发布时返回RecordMetadata实例。

接下来，我们使用RecordMetadata.partition()方法并获取消息的分区。

上述代码片段产生以下记录结果：

```text
1
1
1
1
1
1
1
1
1
1
```

通过这种方式，我们验证了使用相同键发布的消息是否发布到了同一个分区。

## 5. 总结

在本文中，我们了解了Kafka消息中键的意义。

我们首先了解了如何将带有键的消息发布到主题，然后我们讨论了如何验证具有相同键的消息是否发布到同一分区。