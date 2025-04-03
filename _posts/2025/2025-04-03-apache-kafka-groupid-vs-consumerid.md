---
layout: post
title:  Apache Kafka中GroupId和ConsumerId之间的区别
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 简介

在本教程中，我们将阐明[Apache Kafka](https://www.baeldung.com/apache-kafka)中的GroupId和ConsumerId之间的区别，这对于理解如何正确设置消费者非常重要。此外，我们将介绍ClientId和ConsumerId之间的区别，并了解它们之间的关系。

## 2. 消费者组

在探讨Apache Kafka中标识符类型之间的差异之前，让我们先了解消费者组。

**消费者组由多个消费者组成，他们共同消费来自一个或多个主题的消息，完成并行消息处理**。它们在分布式Kafka环境中实现了可扩展性、容错性和高效的并行消息处理。

至关重要的是，**组内的每个消费者只负责处理其主题的一个子集，称为分区**。

## 3. 理解标识符

接下来，让我们从高层次上定义本教程中考虑的所有标识符：

- GroupId唯一标识一个消费者组
- ClientId唯一标识传递给服务器的请求
- ConsumerId分配给消费者组内的单个消费者，是client.id消费者属性和消费者唯一标识符的组合

![](/assets/images/2025/kafka/apachekafkagroupidvsconsumerid01.png)

## 4. 标识符的用途

接下来，让我们了解每个标识符的用途。

**GroupId是负载均衡机制的核心，它支持在消费者之间分配分区**。消费者组管理同一组内消费者之间的协调、负载均衡和分区分配。Kafka确保在任何给定时间只有一个消费者可以访问每个分区，如果组内的消费者发生故障，Kafka会无缝地将分区重新分配给其他消费者，以保持消息处理的连续性。

**Kafka使用ConsumerIds来确保组内的每个消费者在与Kafka代理交互时都是唯一可识别的**，此标识符完全由Kafka管理，用于管理消费者偏移量和跟踪处理分区消息的进度。

**最后，ClientId允许开发人员配置将包含在服务器端请求日志中的逻辑应用程序名称，从而跟踪请求的来源(而不仅仅是IP/端口)**。因为我们可以控制此值，所以我们可以创建两个具有相同ClientId的独立客户端。但是，在这种情况下，Kafka生成的ConsumerId将有所不同。

## 5. 配置GroupId和ConsumerId

### 5.1 使用Spring Kafka

让我们在[Spring Kafka](https://www.baeldung.com/spring-kafka)中为消费者定义GroupId和ConsumerId，我们将通过利用@KafkaListener注解来实现这一点：

```java
@KafkaListener(topics = "${kafka.topic.name:test-topic}", clientIdPrefix = "neo", groupId = "${kafka.consumer.groupId:test-consumer-group}", concurrency = "4")
public void receive(@Payload String payload, Consumer<String, String> consumer) {
    LOGGER.info("Consumer='{}' received payload='{}'", consumer.groupMetadata()
            .memberId(), payload);
    this.payload = payload;

    latch.countDown();
}
```

请注意我们如何将groupId属性指定为我们选择的任意值。

此外，我们已将clientIdPrefix属性设置为包含自定义前缀，让我们检查应用程序日志以验证ConsumerId是否包含此前缀：

```text
c.t.t.s.kafka.groupId.MyKafkaConsumer      : Consumer='neo-1-bae916e4-eacb-485a-9c58-bc22a0eb6187' received payload='Test 123...'
```

consumerId(也称为memberId)的值遵循特定模式，它以clientIdPrefix开头，然后是基于组中消费者数量的计数器，最后是UUID。

### 5.2 使用Kafka CLI

我们还可以通过CLI配置GroupId和ConsumerId，我们将使用kafka-console-consumer.sh脚本。让我们启动一个控制台消费者，将group.id设置为test-consumer-group，将client.id属性设置为neo-<sequence_number\>：

```shell
$ kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic Test --group test-consumer-group --consumer-property "client.id=neo-1"
```

在这种情况下，我们必须确保为每个客户端分配一个唯一的client.id。此行为与Spring Kafka不同，在Spring Kafka中，我们设置clientIdPrefix，框架会为其添加一个序列号。如果我们describe消费者组，我们将看到Kafka为每个消费者生成的ConsumerId：

```shell
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group test-consumer-group --describe
GROUP               TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID                                HOST            CLIENT-ID
test-consumer-group Test            0          0               0               0               neo-1-975feb3f-9e5a-424b-9da3-c2ec3bc475d6 /127.0.0.1      neo-1
test-consumer-group Test            1          0               0               0               neo-1-975feb3f-9e5a-424b-9da3-c2ec3bc475d6 /127.0.0.1      neo-1
test-consumer-group Test            2          0               0               0               neo-1-975feb3f-9e5a-424b-9da3-c2ec3bc475d6 /127.0.0.1      neo-1
test-consumer-group Test            3          0               0               0               neo-1-975feb3f-9e5a-424b-9da3-c2ec3bc475d6 /127.0.0.1      neo-1
test-consumer-group Test            7          0               0               0               neo-3-09b8d4ee-5f03-4386-94b1-e068320b5e6a /127.0.0.1      neo-3
test-consumer-group Test            8          0               0               0               neo-3-09b8d4ee-5f03-4386-94b1-e068320b5e6a /127.0.0.1      neo-3
test-consumer-group Test            9          0               0               0               neo-3-09b8d4ee-5f03-4386-94b1-e068320b5e6a /127.0.0.1      neo-3
test-consumer-group Test            4          0               0               0               neo-2-6a39714e-4bdd-4ab8-bc8c-5463d78032ec /127.0.0.1      neo-2
test-consumer-group Test            5          0               0               0               neo-2-6a39714e-4bdd-4ab8-bc8c-5463d78032ec /127.0.0.1      neo-2
test-consumer-group Test            6          0               0               0               neo-2-6a39714e-4bdd-4ab8-bc8c-5463d78032ec /127.0.0.1      neo-2
```

## 6. 比较

让我们总结一下我们讨论过的3个标识符之间的主要区别：

| 方面   | GroupId                                          | ConsumerId                                 | ClientId                                  |
|------| ----------------------------------------------------- | -------------------------------------------- |-------------------------------------------|
| 表示什么 | 消费者组                                              | 消费者组中的单个消费者                       | 消费者组中的单个消费者                               |
| 值的来源 | 开发者设置GroupId                                     |Kafka根据client.id消费者属性生成ConsumerId | 开发者设置client.id消费者属性                       |
| 是否唯一 | 如果两个消费者组有相同的GroupId，那么它们实际上是一个 |Kafka确保每个消费者都有唯一的值             | 它不必是唯一的，根据用例，可以为两个消费者赋予相同的client.id消费者属性值 |

## 7. 总结

在本文中，我们了解了与Kafka消费者相关的一些关键标识符：GroupId、ClientId和ConsumerId，并介绍了它们的用途以及如何配置它们。