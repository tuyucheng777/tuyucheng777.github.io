---
layout: post
title:  Kafka中的提交偏移量
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 概述

在[Kafka](https://www.baeldung.com/apache-kafka)中，消费者从分区读取消息。在读取消息时，需要考虑一些问题，例如确定从[分区](https://www.baeldung.com/apache-kafka#3-topics-amp-partitions)读取哪些消息，或者在发生故障时防止重复读取消息或丢失消息。解决这些问题的方法是使用偏移量。

在本教程中，我们将了解Kafka中的偏移量。我们将了解如何提交偏移量来管理消息消费，并讨论其方法和缺点。

## 2. 什么是偏移？

我们知道Kafka将消息存储在主题中，每个主题可以有多个分区，每个消费者从主题的一个分区读取消息。在这里，**Kafka借助偏移量跟踪消费者读取的消息**。偏移量是从0开始的整数，随着消息的存储而递增1。

假设一个消费者已从分区读取了5条消息，然后，根据配置，Kafka将偏移量4标记为已提交(从0开始的序列)。消费者下次尝试读取消息时，将消费偏移量5及以上的消息。

**如果没有偏移量，就无法避免重复处理或数据丢失，这就是它如此重要的原因**。

我们可以将其与数据库存储进行类比。在数据库中，我们在执行完SQL语句后提交以持久化更改。同样，在从分区读取后，我们会提交偏移量来标记已处理消息的位置。

## 3. 提交偏移量的方式

**提交偏移量有4种方法**，我们将详细介绍每种方法，并讨论它们的用例、优点和缺点。

让我们首先在pom.xml中添加Kafka客户端API[依赖](https://mvnrepository.com/artifact/org.apache.kafka/kafka-clients)：

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>3.6.1</version>
</dependency>

```

### 3.1 自动提交

这是提交偏移量最简单的方法，**默认情况下，Kafka使用自动提交-每5秒提交poll()方法返回的最大偏移量**。poll()返回一组消息，超时时间为10秒，如代码所示：

```java
KafkaConsumer<Long, String> consumer = new KafkaConsumer<>(props);
consumer.subscribe(KafkaConfigProperties.getTopic());
ConsumerRecords<Long, String> messages = consumer.poll(Duration.ofSeconds(10));
for (ConsumerRecord<Long, String> message : messages) {
    // processed message
}
```

自动提交的问题在于，如果应用程序发生故障，数据丢失的几率非常高。当[poll](https://www.baeldung.com/java-kafka-consumer-api-read#2-consuming-messages)()返回消息时，**Kafka可能会在处理消息之前提交最大的偏移量**。

假设poll()返回100条消息，而消费者在自动提交时处理了60条消息。然后，由于某些故障，消费者崩溃了。当新的消费者上线读取消息时，它从偏移量101开始读取，导致61到100之间的消息丢失。

因此，我们需要其他方法来避免这种缺点；答案是手动提交。

### 3.2 手动同步提交

在手动提交中，无论是同步还是异步，都需要通过将默认属性(enabled.auto.commit)设置为false来禁用自动提交：

```java
Properties props = new Properties();
props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "false");
```

禁用手动提交后，**我们现在了解commitSync()的用法**：

```java
KafkaConsumer<Long, String> consumer = new KafkaConsumer<>(props);
consumer.subscribe(KafkaConfigProperties.getTopic());
ConsumerRecords<Long, String> messages = consumer.poll(Duration.ofSeconds(10));
    // process the messages
consumer.commitSync();
```

**此方法仅在处理完消息后才提交偏移量，从而防止数据丢失**。但是，如果消费者在提交偏移量之前崩溃，则无法防止重复读取。除此之外，它还会影响应用程序性能。

commitSync()会阻塞代码，直到完成为止。此外，如果出现错误，它会继续重试。这会降低应用程序的吞吐量，这是我们不希望看到的。因此，Kafka提供了另一种解决方案，即异步提交，可以解决这些缺点。

### 3.3 手动异步提交

Kafka提供commitAsync()来异步提交偏移量，**它通过在不同线程中提交偏移量来克服手动同步提交的性能开销**。让我们实现一个异步提交来理解这一点：

```java
KafkaConsumer<Long, String> consumer = new KafkaConsumer<>(props); 
consumer.subscribe(KafkaConfigProperties.getTopic()); 
ConsumerRecords<Long, String> messages = consumer.poll(Duration.ofSeconds(10));
  // process the messages
consumer.commitAsync();
```

异步提交的问题在于，如果失败，它不会重试。它依赖于commitAsync()的下一次调用，该调用将提交最新的偏移量。

假设300是我们想要提交的最大偏移量，但我们的commitAsync()由于某些问题而失败。在重试之前，另一个commitAsync()调用可能会提交最大偏移量400，因为它是异步的。当失败的commitAsync()重试时，如果它成功提交了偏移量300，它将覆盖之前的400提交，从而导致重复读取。这就是commitAsync()不重试的原因。

### 3.4 提交特定偏移量

有时，我们需要对偏移量进行更多控制。假设我们正在小批量处理消息，并希望在处理消息后立即提交偏移量，**我们可以使用commitSync()和commitAsync()的重载方法，它们接收Map参数来提交特定偏移量**：

```java
KafkaConsumer<Long, String> consumer = new KafkaConsumer<>(props);
consumer.subscribe(KafkaConfigProperties.getTopic());
Map<TopicPartition, OffsetAndMetadata> currentOffsets = new HashMap<>();
int messageProcessed = 0;
while (true) {
    ConsumerRecords<Long, String> messages = consumer.poll(Duration.ofSeconds(10));
    for (ConsumerRecord<Long, String> message : messages) {
        // processed one message
        messageProcessed++;
        currentOffsets.put(
            new TopicPartition(message.topic(), message.partition()),
            new OffsetAndMetadata(message.offset() + 1));
        if (messageProcessed%50==0){
            consumer.commitSync(currentOffsets);
        }
    }
}
```

在这段代码中，我们管理一个currentOffsets Map，该Map以TopicPartition为键，以OffsetAndMetadata为值。我们将消息处理过程中已处理消息的TopicPartition和OffsetAndMetadata插入到currentOffsets Map中，当已处理消息数达到50条时，我们使用currentOffsets Map调用commitSync()以将这些消息标记为已提交。

这种方式的行为与同步和异步提交相同，唯一的区别是，这里我们决定要提交的偏移量，而不是Kafka。

## 4. 总结

在本文中，我们了解了偏移量及其在Kafka中的重要性。此外，我们探讨了提交偏移量的4种方式，包括手动和自动。最后，我们分析了它们各自的优缺点。我们可以得出总结，在Kafka中没有绝对的最佳提交方式；相反，这取决于具体的用例。