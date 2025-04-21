---
layout: post
title:  使用Apache Kafka进行数据建模
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 概述

在本教程中，我们将使用Apache Kafka进入事件驱动架构的数据建模领域。

## 2. 设置

Kafka集群由多个在Zookeeper集群中注册的Kafka Broker组成，为了简单起见，**我们将使用[Confluent](https://docs.confluent.io/5.0.0/installation/docker/docs/installation/clustered-deployment.html#docker-compose-setting-up-a-three-node-kafka-cluster)发布的现成Docker镜像和[docker-compose](https://baeldung.com/docker-compose)配置**。

首先，让我们下载适用于3节点Kafka集群的docker-compose.yml：

```shell
$ BASE_URL="https://raw.githubusercontent.com/confluentinc/cp-docker-images/5.3.3-post/examples/kafka-cluster"
$ curl -Os "$BASE_URL"/docker-compose.yml
```

接下来，让我们启动Zookeeper和Kafka代理节点：

```shell
$ docker-compose up -d
```

最后，我们可以验证所有Kafka代理都已启动：

```shell
$ docker-compose logs kafka-1 kafka-2 kafka-3 | grep started
kafka-1_1      | [2020-12-27 10:15:03,783] INFO [KafkaServer id=1] started (kafka.server.KafkaServer)
kafka-2_1      | [2020-12-27 10:15:04,134] INFO [KafkaServer id=2] started (kafka.server.KafkaServer)
kafka-3_1      | [2020-12-27 10:15:03,853] INFO [KafkaServer id=3] started (kafka.server.KafkaServer)
```

## 3. 事件基础

在我们承担事件驱动系统的数据建模任务之前，我们需要了解一些概念，例如事件、事件流、生产者-消费者和主题。

### 3.1 事件

Kafka世界中的事件是领域世界中发生的事情的信息日志，它通过**将信息记录为键值对**消息以及一些其他属性(例如时间戳、元信息和标头)来实现。

假设我们正在模拟一场国际象棋游戏；那么事件可能是一个举动：

![](/assets/images/2025/kafka/apachekafkadatamodeling01.png)

我们可以注意到，**事件包含了参与者、动作以及发生时间等关键信息**。在本例中，Player1是参与者，动作是在2020/12/25 00:08:30将车从a1单元格移动到a5单元格。

### 3.2 消息流

Apache Kafka是一个流处理系统，它将事件捕获为消息流。在象棋游戏中，我们可以将事件流视为棋手走棋的记录。

每次事件发生时，棋盘都会有一个快照来代表其状态，通常使用传统的表结构来存储对象的最新静态状态。

另一方面，事件流可以帮助我们以事件的形式捕捉两个连续状态之间的动态变化，**如果我们执行一系列这些不可变的事件，就可以从一个状态转换到另一个状态**。这就是事件流与传统表之间的关系，通常被称为流表对偶性。

让我们用两个连续事件来形象化棋盘上的事件流：

![](/assets/images/2025/kafka/apachekafkadatamodeling02.png)

## 4. 主题

在本节中，我们将学习如何对通过Apache Kafka路由的消息进行分类。

### 4.1 分类

在Apache Kafka这样的消息系统中，任何产生事件的实体通常被称为生产者，而读取和消费这些消息的实体则被称为消费者。

在现实世界中，每个生产者都可以生成不同类型的事件，因此如果我们期望消费者过滤与他们相关的消息并忽略其余消息，那么将会浪费大量精力。

为了解决这个基本问题，**Apache Kafka使用了主题(Topic)，这些主题本质上是属于同一组的消息**。因此，消费者在消费事件消息时可以提高工作效率。

在我们的棋盘示例中，可以使用主题将所有动作分组到chess-moves主题下：

```shell
$ docker run \
  --net=host --rm confluentinc/cp-kafka:5.0.0 \
  kafka-topics --create --topic chess-moves \
  --if-not-exists \
  --partitions 1 --replication-factor 1 \
  --zookeeper localhost:32181
Created topic "chess-moves".
```

### 4.2 生产者-消费者

现在，让我们看看生产者和消费者如何使用Kafka的主题进行消息处理。我们将使用Kafka发行版自带的kafka-console-producer和kafka-console-consumer工具来演示这一点。

让我们[启动](https://docs.docker.com/engine/reference/commandline/run/)一个名为kafka-producer的容器，在其中我们将调用生产者实用程序：

```shell
$ docker run \
--net=host \
--name=kafka-producer \
-it --rm \
confluentinc/cp-kafka:5.0.0 /bin/bash
# kafka-console-producer --broker-list localhost:19092,localhost:29092,localhost:39092 \
--topic chess-moves \
--property parse.key=true --property key.separator=:
```

同时，我们可以启动一个名为kafka-consumer的容器，在其中调用消费者实用程序：

```shell
$ docker run \
--net=host \
--name=kafka-consumer \
-it --rm \
confluentinc/cp-kafka:5.0.0 /bin/bash
# kafka-console-consumer --bootstrap-server localhost:19092,localhost:29092,localhost:39092 \
--topic chess-moves --from-beginning \
--property print.key=true --property print.value=true --property key.separator=:
```

现在，让我们通过生产者记录一些游戏动作：

```shell
>{Player1 : Rook, a1->a5}
```

当消费者处于活跃状态时，它将接收以Player1为键的消息：

```shell
{Player1 : Rook, a1->a5}
```

## 5. 分区

接下来，让我们看看如何使用分区对消息进行进一步分类并提高整个系统的性能。

### 5.1 并发

**我们可以将一个主题划分为多个分区，并调用多个消费者来消费来自不同分区的消息**，通过启用这种并发行为，可以提高系统的整体性能。

**默认情况下，在创建主题时支持–bootstrap-server选项的Kafka版本会为每个主题创建一个分区，除非在创建主题时明确指定**。但是，对于现有主题，我们可以增加分区数，让我们将“chess-moves”主题的分区数设置为3：

```shell
$ docker run \
--net=host \
--rm confluentinc/cp-kafka:5.0.0 \
bash -c "kafka-topics --alter --zookeeper localhost:32181 --topic chess-moves --partitions 3"
WARNING: If partitions are increased for a topic that has a key, the partition logic or ordering of the messages will be affected
Adding partitions succeeded!
```

### 5.2 分区键

在主题中，Kafka使用分区键跨多个分区处理消息。一方面，生产者隐式地使用分区键将消息路由到其中一个分区。另一方面，每个消费者都可以从特定分区读取消息。

默认情况下，**生产者会生成一个键的哈希值，然后对其后的分区数进行模数运算**。之后，它会将消息发送到计算出的标识符所标识的分区。

让我们使用kafka-console-producer实用程序创建新的事件消息，但这次我们将记录两个玩家的动作：

```shell
# kafka-console-producer --broker-list localhost:19092,localhost:29092,localhost:39092 \
--topic chess-moves \
--property parse.key=true --property key.separator=:
>{Player1: Rook, a1 -> a5}
>{Player2: Bishop, g3 -> h4}
>{Player1: Rook, a5 -> e5}
>{Player2: Bishop, h4 -> g3}
```

现在，我们可以有两个消费者，一个从分区1读取，另一个从分区2读取：

```shell
# kafka-console-consumer --bootstrap-server localhost:19092,localhost:29092,localhost:39092 \
--topic chess-moves --from-beginning \
--property print.key=true --property print.value=true \
--property key.separator=: \
--partition 1
{Player2: Bishop, g3 -> h4}
{Player2: Bishop, h4 -> g3}
```

我们可以看到，Player2的所有动作都被记录到了分区1中。同样，我们可以检查Player1的所有动作是否被记录到了分区0中。

## 6. 扩展

如何概念化主题和分区对于水平扩展至关重要。一方面，**主题更像是一种预定义的数据分类**；另一方面，分区是一种动态的数据分类，会实时发生。

此外，我们在一个主题中可以配置的分区数量存在实际限制，这是因为每个分区都映射到代理节点文件系统中的一个目录，**当我们增加分区数量时，操作系统上打开的文件句柄数量也会增加**。

根据经验法则，[Confluent的专家建议](https://www.confluent.io/blog/how-choose-number-topics-partitions-kafka-cluster/)将每个代理的分区数量限制为100 x b x r，其中b是Kafka集群中的代理数量，r是复制因子。

## 7. 总结

在本文中，我们使用Docker环境介绍了使用Apache Kafka进行消息处理的系统的数据建模基础知识。在对事件、主题和分区有了基本的了解之后，我们现在可以概念化事件流并进一步使用这种架构范例。