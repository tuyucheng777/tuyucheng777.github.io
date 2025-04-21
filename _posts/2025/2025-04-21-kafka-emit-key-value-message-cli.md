---
layout: post
title:  从Kafka的命令行发送键/值消息
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 概述

在本教程中，我们将学习两种从[Kafka](https://www.baeldung.com/apache-kafka#2-use-kafka-cli)命令行发送键/值消息的方法。

确保特定主题上[消息的排序](https://www.baeldung.com/kafka-message-ordering)是处理金融交易、订单、在线购物等现实生活中的事件驱动系统中的常见要求，在这些场景中，我们应该对发送到这些主题的事件使用[Kafka消息键](https://www.baeldung.com/java-kafka-message-key#significance-of-a-key-in-a-kafka-message)。

## 2. 先决条件

在从命令行发送键/值消息之前，我们需要检查一些事项。

**首先，我们需要一个正在运行的Kafka实例**，如果没有，可以参考[Kafka Docker](https://www.baeldung.com/ops/kafka-docker-setup)或[Kafka快速入门](https://www.baeldung.com/ops/kafka-list-topics#setting-up-kafka)指南来搭建工作环境，以下章节假设我们已经有一个可以通过kafka-server:9092访问的Kafka环境。

接下来，我们假设从命令行发送的消息是支付系统的一部分，对应的模型类如下：

```java
public class PaymentEvent {
    private String reference;
    private BigDecimal amount;
    private Currency currency;

    // standard getters and setters
}
```

**另一个先决条件是能够访问Kafka CLI工具**，这是一个简单的过程，我们需要下载[Kafka版本](https://kafka.apache.org/downloads)，解压下载的文件，然后导航到解压后的文件夹。Kafka CLI工具现在位于bin文件夹中，我们将假设以下部分中的所有CLI命令均在解压后的Kafka文件夹位置执行。

接下来，让我们创建用于发送消息的payments主题：

```shell
bin/kafka-topics.sh --create --topic payments --bootstrap-server kafka-server:9092
```

我们应该在控制台中看到以下消息，表明主题已成功创建：

```shell
Created topic payments.
```

最后，我们还在payments主题上创建一个Kafka消费者来测试消息是否正确发送：

```shell
bin/kafka-console-consumer.sh --topic payments --bootstrap-server kafka-server:9092 --property "print.key=true" --property "key.separator=="
```

请注意上一个命令末尾的print.key属性，消费者如果不显式将该属性设置为true，就不会打印消息键。我们还覆盖了key.separator属性的默认值(\\t制表符)，以便与后续章节中我们生成消息的方式保持一致。

现在，我们准备开始从命令行发送键/值消息。

## 3. 从命令行发送键/值消息

**我们使用[Kafka控制台生产者](https://www.baeldung.com/apache-kafka-data-modeling#producer-consumer)从命令行发送键/值消息**：

```shell
bin/kafka-console-producer.sh --topic payments --bootstrap-server kafka-server:9092 --property "parse.key=true" --property "key.separator=="
```

当我们想要从CLI提供消息键以及消息有效负载时，需要上一个命令末尾提供的parse.key和key.separator属性。

运行上述命令后，会出现一个提示，我们可以在其中提供消息键和消息有效负载：

```shell
>KEY1={"reference":"P000000001", "amount": "37.75", "currency":"EUR"}
>KEY2={"reference":"P000000002", "amount": "2", "currency":"EUR"}
```

我们可以从消费者输出中看到消息键和消息有效负载都从命令行正确发送：

```text
KEY1={"reference":"P000000001", "amount": "37.75", "currency":"EUR"}
KEY2={"reference":"P000000002", "amount": "2", "currency":"EUR"}
```

## 4. 从文件发送键/值消息

**从命令行发送键/值消息的另一种方法是使用文件**，让我们看看它是如何工作的。

首先，让我们创建包含以下内容的payment-events.txt文件：

```text
KEY3={"reference":"P000000003", "amount": "80", "currency":"SEK"}
KEY4={"reference":"P000000004", "amount": "77.8", "currency":"GBP"}
```

现在，让我们启动控制台生产者并使用payment-events.txt文件作为输入：

```shell
bin/kafka-console-producer.sh --topic payments --bootstrap-server kafka-server:9092 --property "parse.key=true" --property "key.separator==" < payment-events.txt
```

查看消费者输出，我们可以看到这次消息键和消息有效负载也都正确发送了：

```text
KEY3={"reference":"P000000003", "amount": "80", "currency":"SEK"}
KEY4={"reference":"P000000004", "amount": "77.8", "currency":"GBP"}
```

## 5. 总结

在本文中，我们学习了如何在Kafka中通过命令行发送键值消息，我们还了解了使用现有文件发送批量事件的另一种方法。当我们想要确保特定主题的消息传递，同时保持消息的顺序时，这些方法非常有用。