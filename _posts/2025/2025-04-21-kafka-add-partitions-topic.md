---
layout: post
title:  如何向Kafka中的现有主题添加分区
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 概述

[Kafka](https://www.baeldung.com/apache-kafka)是一个非常流行的消息队列，功能丰富。我们将消息存储在Kafka的主题(Topic)中，主题又被划分为分区(Partition)，消息实际存储在这些分区中。有时我们需要增加主题内的分区数量。在本教程中，我们将学习如何实现这种特殊场景。

## 2. 增加分区的原因

在讨论如何增加分区之前，有必要先讨论一下为什么要这样做。在某些情况下，我们需要增加Kafka的分区数量，以下列出了一些场景：

- 当生产者产生大量消息，Kafka分区无法跟上时
- 当我们在消费者组中添加新消费者来处理并行处理时
- 当某些分区处理不成比例的更多数据时
- 为了容错
- 考虑未来的需求，主动增加分区

所以，我们可以理解添加新分区的原因有很多，现在的问题是如何做到这一点？在下一节中，我们将学习两种实现此功能的方法。

## 3. 如何添加分区

Kafka提供了两种添加新分区的方法，一种方法是通过CLI运行Kafka脚本，另一种是通过[Kafka Admin API](https://www.baeldung.com/kafka-topic-creation)(一种以编程方式添加分区的方法)添加分区，让我们逐一学习如何使用这两种方法。

### 3.1 使用Kafka脚本

Kafka提供了一个kafka-topics.sh脚本，用于在主题中添加新的分区，以下是该脚本的CLI命令：

```shell
$ bin/kafka-topics.sh --bootstrap-server <broker:port> --topic <topic-name> --alter --partitions <number>
```

假设我们的代理运行在localhost:9092，主题名称为my-topic，现有分区为两个，让我们再添加一个分区：

```shell
$ bin/kafka-topics.sh --bootstrap-server localhost:9092 --topic my-topic --alter --partitions 3
```

值得注意的是，**这里的分区数是3，其中包括现有分区数和新分区数**，此命令确保代理总共有3个分区。

### 3.2 使用Kafka API

Kafka还提供了一种使用其Admin API以编程方式实现相同任务的方法，该API非常简单，只需要上述CLI命令所需的参数即可。

首先，我们需要将Kafka客户端库添加到项目中：

```xml
<dependency>
     <groupId>org.apache.kafka</groupId>
     <artifactId>kafka-clients</artifactId>
     <version>3.9.0</version>
</dependency>
```

可以从[Maven Central](https://mvnrepository.com/artifact/org.apache.kafka/kafka-clients)找到此依赖的最新版本。

现在，让我们了解如何以编程方式增加分区：

```java
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
try (AdminClient adminClient = AdminClient.create(props)) {
    adminClient.createPartitions(Collections.singletonMap("my-topic", NewPartitions.increaseTo(3)))
        .all()
        .get();
} catch (Exception e) {
   throw new RuntimeException(e);
}
```

如我们所见，AdminClient需要与代理地址、主题名称和分区总数相关的参数。同样，**我们可以从方法名称increaseTo()中看到，它的参数需要指定分区数，其中包括要添加的新分区以及现有分区的数量**。

## 4. 增加分区时的常见陷阱

虽然添加新分区看似简单，但实际上需要付出一些代价，其中涉及一些注意事项，我们将在本节中讨论，让我们讨论一下一些重要的陷阱。

### 4.1 对消息排序的影响

我们知道，Kafka确保分区内消息的有序性，但无法跨分区保证。当我们增加分区数时，键的哈希值可能会发生变化，导致某些键被重新分配到不同的分区，这可能会扰乱特定键的消息顺序。

如果在我们的系统中，消息排序至关重要，那么这种重哈希操作如果依赖于分区数量，则可能会导致问题。为了避免这种情况，我们不应该使用可能导致问题的键计算函数，我们的键计算函数和分区应该能够解决这个问题。

### 4.2 消费者重平衡

添加分区会触发消费者组重新平衡，这可能会暂时中断消息消费，在重新平衡期间，消费者可能会短暂停止处理。

因此，有必要在非高峰时段或计划维护时段执行分区增加操作。我们应该使用Kafka的优雅关闭和重新平衡设置来最大限度地减少中断。

### 4.3 增加Broker和集群负载

如果分区数量增加太多，可能会对Kafka集群造成压力，这是因为分区数量越多，意味着代理上元数据和管理开销也就越大。

这就是为什么我们需要确保代理在增加分区之前和之后拥有充足的资源(例如CPU、内存和磁盘使用率)，我们需要确保代理有足够的扩展能力来处理新的负载。

### 4.4 重分区复杂性

虽然Kafka允许动态添加分区，但它不会将现有数据重新分布到新分区。因此，只有新数据才会写入新增分区，这可能会导致数据分布不均匀。

因此，我们有责任重新处理旧数据，将其重新分布到新的分区中。避免过于频繁地添加分区非常重要，我们应该始终规划分区策略，以应对长期增长。

除了上述注意事项之外，我们还面临更多挑战，例如客户端配置问题、延迟问题、分区策略问题等等。因此，在非生产服务器上进行规划和测试以降低这些风险至关重要。

## 5. 总结

在本文中，我们了解了为什么需要在Kafka中添加新分区，我们还介绍了两种添加新分区的方法-通过CLI和Kafka Admin API。最后，我们讨论了添加新分区时可能遇到的一些陷阱以及如何预测它们。