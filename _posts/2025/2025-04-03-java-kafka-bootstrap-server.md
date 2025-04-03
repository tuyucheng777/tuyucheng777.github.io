---
layout: post
title:  Kafka配置中的bootstrap-server
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 概述

每当我们实现[Kafka](https://www.baeldung.com/tag/kafka)生产者或消费者(例如，[使用Spring](https://www.baeldung.com/tag/kafka))时，我们需要配置的一件事就是“bootstrap.servers”属性。

在本教程中，我们将了解此设置的含义及其用途。

## 2. Kafka拓扑

Kafka的拓扑结构专为可扩展性和高可用性而设计，**这就是为什么有一组服务器(代理)处理代理之间复制的主题分区**。每个分区都有一个代理作为领导者，其他代理作为追随者。

生产者将消息发送到分区领导者，然后领导者将记录传播到每个副本。消费者通常也连接到分区领导者，因为消费消息会改变状态(消费者偏移量)。

副本数是复制因子，建议值为3，因为它在性能和容错能力之间提供了适当的平衡，并且云提供商通常会提供3个数据中心(可用区)作为区域的一部分进行部署。

举例来说，下图展示了一个由4个Broker组成的集群，该集群提供一个具有2个分区且因子为3的主题：

![](/assets/images/2025/kafka/javakafkabootstrapserver01.png)

**当一个分区领导者崩溃时，Kafka会选择另一个代理作为新的分区领导者**。然后，消费者和生产者(“客户端”)也必须切换到新的领导者。因此，如果Broker 1崩溃，情况可能会变为这样：

![](/assets/images/2025/kafka/javakafkabootstrapserver02.png)

## 3. Bootstrap

正如我们所见，整个集群是动态的，**客户端需要了解拓扑的当前状态，才能连接到正确的分区领导者来发送和接收消息，这就是Bootstrap发挥作用的地方**。

“bootstrap-servers”配置是“hostname:port”对的列表，用于寻址一个或多个(甚至所有)代理。客户端通过执行以下步骤使用此列表：

- 从列表中选择第一个代理
- 向代理发送请求以获取包含有关主题、分区以及每个分区的领导代理的信息的集群元数据(每个代理都可以提供此元数据)
- 连接到所选主题分区的领导代理

**当然，在列表中指定多个代理是有意义的，因为如果第一个代理不可用，客户端可以选择第二个代理进行引导**。

Kafka使用Kraft(早期是Zookeeper)来管理所有这类编排。

## 4. 示例

假设我们在开发环境中使用包含Kafka和Kraft的简单Docker镜像(例如[bashj79/kafka-kraft)](https://hub.docker.com/r/bashj79/kafka-kraft)，我们可以使用以下命令安装此Docker镜像：

```shell
docker run -p 9092:9092 -d bashj79/kafka-kraft
```

这将在容器内和主机上的端口9092上运行一个可用的Kafka实例。

### 4.1 使用Kafka CLI

连接到Kafka的一种可能性是[使用Kafka CLI](https://docs.confluent.io/kafka/operations-tools/kafka-tools.html#kafka-console-consumer-sh)，它在Kafka安装中可用。首先，让我们创建一个名为samples的主题。在容器的Bash中，我们可以运行以下命令：

```shell
$ cd /opt/kafka/bin
$ sh kafka-topics.sh --bootstrap-server localhost:9092 --create --topic samples --partitions 1 --replication-factor 1
```

如果我们想要开始消费该主题，我们需要再次指定引导服务器：

```shell
$ sh kafka-console-consumer.sh --bootstrap-server localhost:9092,another-host.com:29092 --topic samples
```

我们还可以将集群元数据作为一种虚拟文件系统进行探索，我们使用kafka-metadata-shell脚本连接到元数据：

```shell
$ sh kafka-metadata-shell.sh --snapshot /tmp/kraft-combined-logs/__cluster_metadata-0/00000000000000000167.log
```

![](/assets/images/2025/kafka/javakafkabootstrapserver03.png)

### 4.2 使用Java

在Java应用程序中，我们可以使用[Kafka客户端](https://docs.confluent.io/kafka-clients/java/current/overview.html)：

```java
static Consumer<Long, String> createConsumer() {
    final Properties props = new Properties();
    props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG,
            "localhost:9092,another-host.com:29092");
    props.put(ConsumerConfig.GROUP_ID_CONFIG,
            "MySampleConsumer");
    props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG,
            LongDeserializer.class.getName());
    props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG,
            StringDeserializer.class.getName());
    // Create the consumer using props.
    final Consumer<Long, String> consumer = new KafkaConsumer<>(props);
    // Subscribe to the topic.
    consumer.subscribe(Arrays.asList("samples"));
    return consumer;
}
```

**通过Spring Boot和Spring的[Kafka集成](https://spring.io/projects/spring-kafka)，我们可以简单地配置application.properties**：

```properties
spring.kafka.bootstrap-servers=localhost:9092,another-host.com:29092
```

## 5. 总结

在本文中，我们了解到Kafka是一个由多个Broker组成的分布式系统，这些Broker复制主题分区以确保高可用性、可扩展性和容错能力。

客户端需要从一个代理检索元数据，以找到要连接的当前分区领导者。然后，此代理将成为引导服务器，我们通常会提供引导服务器列表，以便在主代理无法访问时为客户端提供替代方案。