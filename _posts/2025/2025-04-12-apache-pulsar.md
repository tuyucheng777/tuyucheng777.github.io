---
layout: post
title:  Apache Pulsar简介
category: apache
copyright: apache
excerpt: Apache Pulsar
---

## 1. 简介

**[Apache Pulsar](https://pulsar.apache.org/)是雅虎开发的基于发布/订阅的分布式开源消息系统**。

它的创建是为了支持雅虎的关键应用程序，如雅虎邮箱、雅虎财经、雅虎体育等。然后，在2016年，它在Apache软件基金会下开源。

## 2. 架构

**Pulsar是一个多租户、高性能的服务器到服务器消息传递解决方案**，它由一组Broker和Bookie以及内置的[Apache ZooKeeper](https://zookeeper.apache.org/)组成，用于配置和管理。Bookie来自[Apache BookKeeper](https://bookkeeper.apache.org/)，它为消息提供存储，直到它们被消费为止。

在集群中我们将拥有：

- 多个集群代理处理来自生产者的传入消息并将消息发送给消费者
- Apache BookKeeper支持消息持久化
- Apache ZooKeeper用于存储集群配置

为了更好地理解这一点，让我们看一下[文档](https://pulsar.apache.org/docs/en/concepts-architecture-overview/)中的架构图：

![](/assets/images/2025/apache/apachepulsar01.png)

## 3. 主要特点

让我们首先快速了解一下一些主要功能：

- 内置对多个集群的支持
- 支持跨多个集群的地理复制消息
- 多种订阅模式
- 可扩展至数百万个主题
- 使用Apache BookKeeper来保证消息传递。
- 低延迟

现在，让我们详细讨论一些关键特性。

### 3.1 消息模型

该框架提供了灵活的消息传递模型。一般来说，消息传递架构有两种消息传递模型：队列模型和发布/订阅模型。发布/订阅模型是一种广播消息传递系统，消息会发送给所有消费者；而队列模型是一种点对点通信。

**Pulsar将这两个概念结合在一个通用API中**，发布者将消息发布到不同的主题，然后，这些消息被广播到所有订阅者。

消费者通过订阅来获取消息，该库允许消费者选择在同一订阅中使用不同的消息消费方式，包括独占、共享和故障转移，我们将在后面的部分详细讨论这些订阅类型。

### 3.2 部署模式

**Pulsar内置支持在不同环境中部署**，这意味着我们可以在标准的本地机器上使用它，也可以将其部署在Kubernetes集群、Google或AWS云中。

它可以作为单节点运行，用于开发和测试目的。在这种情况下，所有组件(Broker、BookKeeper和ZooKeeper)都在单个进程中运行。

### 3.3 地理复制

**该库提供了开箱即用的数据地理复制支持**，我们可以通过配置不同的地理区域来实现多个集群之间的消息复制。

消息数据几乎实时复制，即使跨集群网络发生故障，数据也始终安全存储在BookKeeper中。复制系统会持续重试，直至复制成功。

**地理复制功能还允许组织跨不同的云提供商部署Pulsar并复制数据，这有助于他们避免使用专有云提供商API**。

### 3.4 持久性

**Pulsar读取并确认数据后，保证不会丢失数据**，数据持久性与配置用于存储数据的磁盘数量有关。

Pulsar通过在存储节点上运行bookie(Apache BookKeeper实例)来确保持久性，每当bookie收到消息时，它会在内存中保存一份副本，并将数据写入WAL(预写日志)。此日志的工作方式与数据库WAL相同，Bookie遵循数据库事务原则运行，确保即使发生机器故障，数据也不会丢失。

除上述功能外，Pulsar还能承受多节点故障。该库会将数据复制到多个Bookie，然后向生产者发送确认消息，这种机制即使在发生多个硬件故障的情况下也能保证0数据丢失。

## 4. 单节点设置

现在让我们看看如何设置Apache Pulsar的单节点集群。

**Apache还提供了一个简单的[客户端API](https://pulsar.apache.org/docs/en/concepts-clients/)，其中包含Java、Python和C++的绑定**，稍后我们将创建一个简单的Java生产者和订阅者示例。

### 4.1 安装

Apache Pulsar现已提供二进制发行版，我们先下载它：

```shell
wget https://archive.apache.org/dist/incubator/pulsar/pulsar-2.1.1-incubating/apache-pulsar-2.1.1-incubating-bin.tar.gz
```

下载完成后，我们可以解压zip文件，解压后的发行版将包含bin、conf、example、licenses和lib文件夹。

之后，我们需要下载内置连接器，它们现在作为单独的软件包提供：

```shell
wget https://archive.apache.org/dist/incubator/pulsar/pulsar-2.1.1-incubating/apache-pulsar-io-connectors-2.1.1-incubating-bin.tar.gz
```

让我们解压连接器并复制Pulsar文件夹中的Connectors文件夹。

### 4.2 启动实例

要启动独立实例，我们可以执行：

```shell
bin/pulsar standalone
```

## 5. Java客户端

现在我们将创建一个Java项目来生成和消费消息，我们还将为不同的订阅类型创建示例。

### 5.1 设置项目

我们首先将[pulsar-client](https://mvnrepository.com/artifact/org.apache.pulsar/pulsar-client)依赖添加到项目中：

```xml
<dependency>
    <groupId>org.apache.pulsar</groupId>
    <artifactId>pulsar-client</artifactId>
    <version>2.1.1-incubating</version>
</dependency>
```

### 5.2 生产者

我们继续创建一个生产者示例，在这里，我们将创建一个主题和一个生产者。

**首先，我们需要创建一个PulsarClient对象，它将使用其自身的协议连接到特定主机和端口上的Pulsar服务**，多个生产者和消费者可以共享同一个客户端对象。

现在，我们将创建一个具有特定主题名称的生产者：

```java
private static final String SERVICE_URL = "pulsar://localhost:6650";
private static final String TOPIC_NAME = "test-topic";

PulsarClient client = PulsarClient.builder()
    .serviceUrl(SERVICE_URL)
    .build();

Producer<byte[]> producer = client.newProducer()
    .topic(TOPIC_NAME)
    .compressionType(CompressionType.LZ4)
    .create();
```

生产者会发送5条消息：

```java
IntStream.range(1, 5).forEach(i -> {
    String content = String.format("hi-pulsar-%d", i);

    Message<byte[]> msg = MessageBuilder.create()
        .setContent(content.getBytes())
        .build();
    MessageId msgId = producer.send(msg);
});
```

### 5.3 消费者

接下来，我们将创建消费者来获取生产者创建的消息，消费者也需要相同的PulsarClient来连接到我们的服务器：

```java
Consumer<byte[]> consumer = client.newConsumer()
    .topic(TOPIC_NAME)
    .subscriptionType(SubscriptionType.Shared)
    .subscriptionName(SUBSCRIPTION_NAME)
    .subscribe();
```

这里我们创建了一个共享订阅类型的客户端，这允许多个消费者连接到同一个订阅并获取消息。

### 5.4 消费者订阅类型

在上面的消费者示例中，**我们创建了一个共享类型的订阅，我们还可以创建独占订阅和故障转移订阅**。

独占订阅只允许一个消费者订阅。

另一方面，故障转移订阅允许用户定义后备消费者，以防一个消费者发生故障，如下Apache图所示：

![](/assets/images/2025/apache/apachepulsar02.png)

## 6. 总结

在本文中，我们重点介绍了Pulsar消息传递系统的功能，例如消息传递模型、地理复制和强大的持久化保证。

我们还学习了如何设置单个节点以及如何使用Java客户端。