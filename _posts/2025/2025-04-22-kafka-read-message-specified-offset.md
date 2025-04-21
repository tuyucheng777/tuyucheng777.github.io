---
layout: post
title:  如何从Kafka中的特定偏移量读取消息
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 概述

[Kafka](https://www.baeldung.com/apache-kafka)是一款流行的开源分布式消息流中间件，它将消息生产者与消息消费者解耦，它使用[发布-订阅模式](https://www.baeldung.com/cs/publisher-subscriber-model)实现消息生产者和消费者的解耦。Kafka使用[主题(Topic)](https://www.baeldung.com/kafka-topics-partitions#what-is-a-kafka-topic)来分发信息，每个主题由不同的分片(Shard)组成，在Kafka术语中称为[分区(Partition)](https://www.baeldung.com/ops/kafka-fifo-queue#topic-partitions)，分区中的每条消息都有一个特定的[偏移量(Offset)](https://www.baeldung.com/kafka-commit-offsets#what-is-offset)。

在本教程中，我们将讨论如何使用[kafka-console-consumer.sh](https://docs.confluent.io/kafka/operations-tools/kafka-tools.html#kafka-console-consumer-sh)命令行工具从主题分区的特定偏移量读取数据，示例中使用的Kafka版本是3.7.0。

## 2. 分区和偏移量简述

**Kafka将写入主题的消息拆分到不同的分区**，所有具有相同键的消息都保存在同一个分区中。但是，如果没有键，Kafka会将消息发送到随机分区。

**Kafka保证分区内消息的顺序，但不保证跨分区消息的顺序**。分区中的每条消息都有一个ID，**此ID称为分区偏移量**，随着新消息被添加到分区，分区偏移量会不断增加。

**默认情况下，消费者会从分区中从低偏移量到高偏移量读取消息**。但是，我们可能需要从分区中的特定偏移量开始读取消息，我们将在下一节中了解如何实现此目标。

## 3. 示例

在本节中，我们将学习如何从特定偏移量读取数据。假设Kafka服务器正在运行，并且已使用[kafka-topics.sh](https://docs.confluent.io/kafka/operations-tools/kafka-tools.html#kafka-topics-sh)创建了一个名为test-topic的主题，**该主题包含3个分区**。

Kafka提供了我们在示例中使用的所有脚本。

### 3.1 写入消息

**我们使用[kafka-console-producer.sh](https://docs.confluent.io/kafka/operations-tools/kafka-tools.html#kafka-console-producer-sh)脚本启动生产者**：

```shell
$ kafka-console-producer.sh --bootstrap-server localhost:9092 --topic test-topic --producer-property partitioner.class=org.apache.kafka.clients.producer.RoundRobinPartitioner
>
```

Kafka服务器监听localhost和9092端口上的客户端连接，因此，–bootstrap-server localhost:9092选项用于连接到Kafka服务器。

不使用键写入主题时，主题只会发送到随机选择的分区之一。但是，在本例中，我们希望将主题平均分配到所有分区，因此**我们使用RoundRobinPartitioner策略，使生产者以轮询方式写入主题**，命令中的–producer-property partitioner.class=org.apache.kafka.clients.producer.RoundRobinPartitioner部分指定了此行为。

箭头符号>表示我们已准备好发送消息，现在让我们发送6条消息：

```shell
$ kafka-console-producer.sh --bootstrap-server localhost:9092 --topic test-topic --producer-property partitioner.class=org.apache.kafka.clients.producer.RoundRobinPartitioner
>Message1
>Message2
>Message3
>Message4
>Message5
>Message6
>
```

第一条消息是Message1，而最后一条消息是Message6，**我们有3个分区，因此由于采用循环分区，我们预计Message1和Message4应该位于同一个分区中。同样，Message2和Message5应该位于另外两个分区中，Message3和Message6也应该位于另外两个分区中**。

### 3.2 读取消息

现在，我们将从特定的偏移量读取消息，**我们使用kafka-console-consumer.sh启动一个消费者**：

```shell
$ kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test-topic --partition 0 --offset 0
Message2
Message5
```

这里，**–partition 0和–offset 0选项指定要使用的分区和偏移量**，分区和偏移量的编号都从0开始。

我们从第一个分区(从第一个偏移量开始)读取的消息是Message2和Message5，正如预期的那样，它们位于同一个分区中。kafka-console-consumer.sh不会退出，而是继续运行以读取新消息。

可以从第二个偏移量开始读取第一个分区中的消息：

```shell
$ kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test-topic --partition 0 --offset 1
Message5 
```

由于使用了–offset 1选项，本例中我们只读取了Message5，我们还可以指定要读取的消息数量：

```shell
$ kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test-topic --partition 0 --offset 0 --max-messages 1
Message2
Processed a total of 1 messages
```

**–max-messages选项指定退出前需要消费的消息数量**，由于我们向kafka-console-consumer.sh传递了–max-messages 1参数，因此本例中我们只读取了Message2。kafka-console-consumer.sh在读取到所需数量的消息后退出，否则，它会等待，直到读取到所需数量的消息。

读取另外两个分区中的消息的方式相同：

```shell
$ kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test-topic --partition 1 --offset 0 --max-messages 2
Message1
Message4
Processed a total of 2 messages
$ kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test-topic --partition 2 --offset 0 --max-messages 2
Message3
Message6
Processed a total of 2 messages
```

结果正如预期。

但是，如果使用–offset传递给kafka-console-consumer.sh的值大于分区中可用消息的数量，则kafka-console-consumer.sh会等到消息写入该分区并立即读取该消息。

## 4. 总结

在本文中，我们学习了如何使用kafka-console-consumer.sh命令行工具从主题分区的特定偏移量读取。

首先，我们了解到分区中的每条消息都有一个ID，称为分区偏移量(Partition Offset)，通常情况下，Kafka会从分区中偏移量最小的消息开始投递。

然后，我们看到可以分别使用kafka-console-consumer.sh的–partition和–offset选项从特定的分区和偏移量读取数据，此外，我们还了解到–max-messages选项指定了要读取的消息数量。