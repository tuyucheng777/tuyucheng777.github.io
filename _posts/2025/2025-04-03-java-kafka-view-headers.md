---
layout: post
title:  使用Java查看Kafka标头
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 简介

[Apache Kafka](https://www.baeldung.com/apache-kafka)是一个分布式流传输平台，允许我们发布和订阅记录流(通常称为消息)。此外，Kafka标头提供了一种将元数据附加到Kafka消息的方法，从而为消息处理提供额外的上下文和灵活性。

在本教程中，我们将深入研究常用的Kafka标头，并学习如何使用Java查看和提取它们。

## 2. Kafka标头概述

**Kafka标头代表附加到Kafka消息的键值对，提供一种在主要消息内容旁边包含补充元数据的方法**。

例如，Kafka标头通过提供将消息定向到特定处理管道或消费者的数据来促进消息路由。此外，标头还能够灵活地携带根据应用程序处理逻辑定制的[自定义应用程序元数据](https://www.baeldung.com/java-kafka-custom-headers)。

## 3. Kafka默认标头

**Kafka会自动在Kafka生产者发送的消息中包含几个默认标头**，此外，这些标头还提供了有关消息的关键元数据和上下文。在本节中，我们将深入探讨一些常用的标头及其在Kafka消息处理领域的重要性。

### 3.1 生产者标头

在Kafka中生成消息时，生产者会自动包含几个默认标头，例如：

- [KafkaHeaders.TOPIC](https://www.baeldung.com/kafka-topics-partitions)：此标头包含消息所属主题的名称。
- [KafkaHeaders.KEY](https://www.baeldung.com/kafka-send-data-partition#2-key-based-approach)：如果消息是使用键生成的，Kafka会自动包含一个名为“key”的标头，其中包含序列化的键字节。
- KafkaHeaders.PARTITION：Kafka添加一个名为“partition”的标头，以指示消息所属的分区ID。
- KafkaHeaders.TIMESTAMP：Kafka在每条消息中附加一个名为“timestamp”的标头，指示生产者生成该消息的时间戳。

### 3.2 消费者标头

以RECEIVED_为前缀的标头由Kafka消费者在消费消息时添加，以提供有关消息接收过程的元数据：

- KafkaHeaders.RECEIVED_TOPIC：此标头包含接收消息的主题的名称。
- KafkaHeaders.RECEIVED_KEY：此标头允许消费者访问与消息关联的键。
- KafkaHeaders.RECEIVED_PARTITION：Kafka添加此标头以指示分配消息的分区的ID。
- KafkaHeaders.RECEIVED_TIMESTAMP：此标头反映消费者收到消息的时间。
- KafkaHeaders.OFFSET：偏移量表示消息在分区日志中的位置。

## 4. 消费带有标头的消息

首先，我们实例化一个KafkaConsumer对象，KafkaConsumer负责订阅Kafka主题并从中获取消息。实例化KafkaConsumer后，我们订阅要从中消费消息的Kafka主题。通过订阅主题，消费者可以接收在该主题上发布的消息。

一旦消费者订阅了主题，我们就开始从Kafka获取记录。**在此过程中，KafkaConsumer会从订阅的主题中检索消息及其相关标头**。

以下代码示例演示了如何消费带有标头的消息：

```java
@KafkaListener(topics = "my-topic")
public void listen(String message, @Headers Map<String, Object> headers) {
    System.out.println("Received message: " + message);
    System.out.println("Headers:");
    headers.forEach((key, value) -> System.out.println(key + ": " + value));
}
```

Kafka监听器容器在从指定主题(例如“my-topic”)接收消息时调用listen()方法，**@Headers注解表示该参数应填充收到的消息的标头**。

以下是示例输出：

```text
Received message: Hello Tuyucheng!
Headers:
kafka_receivedMessageKey: null
kafka_receivedPartitionId: 0
kafka_receivedTopic: my-topic
kafka_offset: 123
... // other headers
```

要访问特定标头，我们可以使用标头Map的get()方法，提供所需标头的键。以下是访问主题名称的示例：

```java
String topicName = headers.get(KafkaHeaders.TOPIC);
```

topicName应该返回my-topic。

此外，**在消费消息时，如果我们已经知道需要处理的标头，我们可以直接将其提取为方法参数**。这种方法提供了一种更简洁、更有针对性的方法来访问特定的标头值，而无需遍历所有标头。

以下代码示例演示了如何消费带有标头的消息，直接提取特定标头作为方法参数：

```java
@KafkaListener(topics = "my-topic")
public void listen(String message, @Header(KafkaHeaders.RECEIVED_PARTITION_ID) int partition) {
    System.out.println("Received message: " + message);
    System.out.println("Partition: " + partition);
}
```

在listen()方法中，我们使用@Header注解直接提取RECEIVED_PARTITION标头，此注解允许我们指定要提取的标头及其对应的类型。**将标头的值直接注入方法参数(在本例中为partition)可实现在方法主体内直接访问**。

以下是输出：

```text
Received message: Hello Tuyucheng!
Partition: 0
```

## 5. 总结

在本文中，我们探讨了Kafka标头在Apache Kafka中的消息处理中的重要性，我们探讨了生产者和消费者自动包含的默认标头。此外，我们还学习了如何提取和使用这些标头。