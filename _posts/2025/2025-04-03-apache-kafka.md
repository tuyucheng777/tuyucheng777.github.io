---
layout: post
title:  Apache Kafka简介
category: kafka
copyright: kafka
excerpt: Apache Kafka
---

## 1. 概述

在本教程中，我们将学习Kafka的基础知识-每个人都应该知道的用例和核心概念，然后我们可以找到并了解[有关Kafka的更详细的文章](https://www.baeldung.com/tag/kafka)。

## 2. 什么是Kafka？

[Kafka](https://kafka.apache.org/)是Apache软件基金会开发的开源流处理平台，我们可以将其用作消息系统，以解耦消息生产者和消费者。但与[ActiveMQ](https://activemq.apache.org/)等“经典”消息系统相比，它旨在处理实时数据流，并提供分布式、容错和高度可扩展的架构来处理和存储数据。

因此，我们可以在各种用例中使用它：

- 实时数据处理和分析
- 日志和事件数据聚合
- 监控和指标收集
- 点击流数据分析
- 欺诈检测
- 大数据管道中的流处理

## 3. 设置本地环境

如果我们第一次接触Kafka，我们可能希望在本地安装以体验其功能。借助Docker，我们可以快速实现这一点。

### 3.1 安装Kafka

我们下载[现有镜像](https://hub.docker.com/r/bashj79/kafka-kraft)并使用此命令运行容器实例：

```shell
docker run -p 9092:9092 -d bashj79/kafka-kraft
```

这将使所谓的Kafka代理在主机系统的端口9092上可用。现在，我们想使用Kafka客户端连接到代理，我们可以使用多个客户端。

### 3.2 使用Kafka CLI

Kafka CLI是安装的一部分，可在Docker容器内使用。我们可以通过连接到容器的bash来使用它。

首先，我们需要使用以下命令找出容器的名称：

```shell
docker ps

CONTAINER ID   IMAGE                    COMMAND                  CREATED        STATUS       PORTS                    NAMES
7653830053fa   bashj79/kafka-kraft      "/bin/start_kafka.sh"    8 weeks ago    Up 2 hours   0.0.0.0:9092->9092/tcp   awesome_aryabhata
```

在此示例中，名称为awesome_aryabhata。然后我们使用以下命令[连接到bash](https://www.baeldung.com/linux/shell-alpine-docker)：

```shell
docker exec -it awesome_aryabhata /bin/bash
```

现在，我们可以创建一个主题(我们稍后会澄清这个术语)并使用以下命令列出所有现有主题：

```shell
cd /opt/kafka/bin

# create topic 'my-first-topic'
sh kafka-topics.sh --bootstrap-server localhost:9092 --create --topic my-first-topic --partitions 1 --replication-factor 1

# list topics
sh kafka-topics.sh --bootstrap-server localhost:9092 --list

# send messages to the topic
sh kafka-console-producer.sh --bootstrap-server localhost:9092 --topic my-first-topic
>Hello World
>The weather is fine
>I love Kafka
```

### 3.3 使用KafkaIO GUI

[KafkaIO](https://kafkio.com)是一个用于管理Kafka的 GUI应用程序。我们可以在任何受支持的平台(Mac、Windows、Linux和Unix)上[下载](https://kafkio.com/download)它并快速安装。然后，为了创建连接我们指定Kafka代理的引导服务器：

![](/assets/images/2025/kafka/apachekafka01.png)

### 3.4 使用Apache Kafka的UI(Kafka UI)

Apache Kafka的UI[(Kafka UI)](https://github.com/kafbat/kafka-ui)是一个Web UI，使用Spring Boot和React实现，并以Docker镜像的形式提供，可使用以下命令[作为容器进行简单安装](https://hub.docker.com/r/provectuslabs/kafka-ui)：

```shell
docker run -it -p 8080:8080 -e DYNAMIC_CONFIG_ENABLED=true provectuslabs/kafka-ui
```

然后我们可以使用[http://localhost:8080](http://localhost:8080/)在浏览器中打开UI并定义一个集群，如下图所示：

![](/assets/images/2025/kafka/apachekafka02.png)

由于Kafka代理与Kafka UI的后端在不同的容器中运行，因此它无法访问localhost:9092。我们可以使用host.docker.internal:9092来寻址主机系统，但这只是引导URL。

不幸的是，Kafka本身会返回一个响应，导致再次重定向到localhost:9092，这不会起作用。如果我们不想配置Kafka(因为这会破坏其他客户端)，我们需要创建从Kafka UI的容器端口9092到主机系统端口9092的端口转发。以下草图说明了连接：

![](/assets/images/2025/kafka/apachekafka03.png)

我们可以设置这个容器内部的端口转发，例如使用socat。我们必须在容器(Alpine Linux)内安装它，因此我们需要以root权限连接到容器的bash。因此，我们需要这些命令，从主机系统的命令行开始：

```shell
# Connect to the container's bash (find out the name with 'docker ps')
docker exec -it --user=root <name-of-kafka-ui-container> /bin/sh
# Now, we are connected to the container's bash.
# Let's install 'socat'
apk add socat
# Use socat to create the port forwarding
socat tcp-listen:9092,fork tcp:host.docker.internal:9092
# This will lead to a running process that we don't kill as long as the container's running
```

因此，每次启动容器时，我们都需要运行socat。另一种可能性是为[Dockerfile](https://github.com/provectus/kafka-ui/blob/master/kafka-ui-api/Dockerfile)提供扩展。

现在，我们可以在Kafka UI中指定localhost：9092作为引导服务器，并且能够查看和创建主题，如下所示：

![](/assets/images/2025/kafka/apachekafka04.png)

### 3.5 使用Kafka Java客户端

我们必须向我们的项目添加以下[Maven依赖](https://mvnrepository.com/artifact/org.apache.kafka/kafka-clients)：

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>3.9.0</version>
</dependency>
```

然后我们可以连接到Kafka并消费我们之前生成的消息：

```java
// specify connection properties
Properties props = new Properties();
props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
props.put(ConsumerConfig.GROUP_ID_CONFIG, "MyFirstConsumer");
props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
// receive messages that were sent before the consumer started
props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
// create the consumer using props.
try (final Consumer<Long, String> consumer = new KafkaConsumer<>(props)) {
    // subscribe to the topic.
    final String topic = "my-first-topic";
    consumer.subscribe(Arrays.asList(topic));
    // poll messages from the topic and print them to the console
    consumer
        .poll(Duration.ofMinutes(1))
        .forEach(System.out::println);
}
```

当然，Spring中有[针对Kafka客户端的集成](https://www.baeldung.com/spring-kafka)。

## 4. 基本概念

### 4.1 生产者和消费者

**我们可以把Kafka的客户端区分为消费者和生产者**，生产者向Kafka发送消息，而消费者从Kafka接收消息，它们只是通过主动轮询的方式从Kafka接收消息，而Kafka本身是被动的，这样可以让每个消费者拥有独特的性能，而不会阻塞Kafka。

当然，可以同时有多个生产者和多个消费者。当然，一个应用程序可以同时包含生产者和消费者。

消费者是消费者组的一部分，Kafka通过一个简单的名称来识别它。**每个消费者组中只有一个消费者会收到消息**，这允许扩展消费者，并保证只传递一次消息。

下图展示了多个生产者和消费者通过Kafka一起工作的情况：

![](/assets/images/2025/kafka/apachekafka05.png)

### 4.2 消息

消息(根据用例，我们也可以将其称为“记录”或“事件”)是Kafka处理的基本数据单位，其有效负载可以是任何二进制格式，也可以是纯文本、[Avro](https://avro.apache.org/)、XML或JSON等文本格式。

每个生产者必须指定一个序列化器来将消息对象转换为二进制有效负载格式，每个消费者必须指定相应的反序列化器来将有效负载格式转换回其JVM内的对象。我们简称这些组件为SerDes，有[内置的SerDes](https://kafka.apache.org/documentation/streams/developer-guide/datatypes.html)，但我们也可以实现自定义SerDes。

下图展示了消息负载序列化和反序列化的过程：

![](/assets/images/2025/kafka/apachekafka06.png)

此外，消息可以具有以下可选属性：

- 键也可以是任何二进制格式，如果我们使用键，我们也需要SerDes，Kafka使用键进行分区(我们将在下一章中详细讨论这一点)。
- 时间戳表示消息的生成时间，Kafka使用时间戳来对消息进行排序或实施保留策略。
- 我们可以[应用标头](https://www.baeldung.com/java-kafka-custom-headers)将元数据与有效负载关联起来，例如，[Spring](https://www.baeldung.com/spring-kafka)默认添加用于序列化和反序列化的类型标头。

### 4.3 主题和分区

**主题是生产者发布消息的逻辑通道或类别**，消费者订阅主题以在其消费者组上下文中接收消息。

默认情况下，主题的保留策略为7天，即7天后，无论是否发送给消费者，Kafka都会自动删除消息。我们可以根据需要进行[配置](https://www.baeldung.com/kafka-message-retention)。

**主题由分区(至少一个)组成**，确切地说，消息存储在主题的一个分区中。在一个分区内，消息获得一个顺序号(偏移量)，这可以确保消息按照存储在分区中的顺序传递给消费者。而且，通过存储消费者组已经收到的偏移量，Kafka保证了仅一次传递。

通过处理多个分区，我们可以确定**Kafka可以在消费者进程池上提供排序保证和负载均衡**。

**当一个消费者订阅该主题时，它将被分配到一个分区**，例如使用Java Kafka客户端API，正如我们已经看到的：

```java
String topic = "my-first-topic";
consumer.subscribe(Arrays.asList(topic));
```

但是，对于消费者来说，可以选择要从中轮询消息的分区：

```java
TopicPartition myPartition = new TopicPartition(topic, 1);
consumer.assign(Arrays.asList(myPartition));
```

此变体的缺点是所有组消费者都必须使用它，因此自动将分区分配给组消费者无法与连接到特殊分区的单个消费者结合使用。此外，如果发生架构变化(例如向组中添加更多消费者)，则无法重新平衡。

**理想情况下，我们的消费者数量与分区数量一样多**，这样每个消费者都可以被分配到恰好一个分区，如下所示：

![](/assets/images/2025/kafka/apachekafka07.png)

如果消费者数量多于分区数量，那么这些消费者将不会从任何分区接收消息：

![](/assets/images/2025/kafka/apachekafka08.png)

如果消费者数量少于分区数量，消费者将从多个分区接收消息，这与最佳负载均衡相冲突：

![](/assets/images/2025/kafka/apachekafka09.png)

**生产者不一定只将消息发送到一个分区**，每条生成的消息都会自动分配到一个分区，遵循以下规则：

- 生产者可以指定分区作为消息的一部分，如果这样做，则该分区具有最高优先级
- 如果消息有键，则通过计算键的哈希值来进行分区，具有相同哈希值的键将存储在同一个分区中。理想情况下，我们的哈希值至少与分区数一样多
- 否则，[Sticky Partitioner](https://cwiki.apache.org/confluence/display/KAFKA/KIP-480%3A+Sticky+Partitioner)将消息分发到各个分区

**再次，将消息存储到同一个分区将保留消息顺序，而将消息存储到不同的分区将导致无序但并行处理**。

如果默认分区不符合我们的期望，我们可以简单地实现自定义分区器。因此，我们实现[Partitioner](https://kafka.apache.org/34/javadoc/org/apache/kafka/clients/producer/Partitioner.html)接口并在生产者初始化期间注册它：

```javascript
Properties producerProperties = new Properties();
// ...  
producerProperties.put(ProducerConfig.PARTITIONER_CLASS_CONFIG, MyCustomPartitioner.class.getName());
KafkaProducer<String, String> producer = new KafkaProducer<>(producerProperties);
```

下图展示了生产者和消费者以及他们与分区的连接：

![](/assets/images/2025/kafka/apachekafka10.png)

每个生产者都有自己的分区器，所以如果我们想确保消息在主题内一致地分区，我们必须确保所有生产者的分区器以相同的方式工作，或者我们应该只与单个生产者合作。

分区按消息到达Kafka代理的顺序存储消息，通常，生产者不会将每条消息作为单个请求发送，而是会在一个批次中发送多条消息。如果我们需要确保消息的顺序以及在一个分区内仅传递一次，我们需要具有[事务感知能力的生产者和消费者](https://www.baeldung.com/kafka-exactly-once)。

### 4.4 集群和分区副本

我们发现，Kafka使用主题分区来实现并行消息传递和消费者负载均衡。但Kafka本身必须具有可扩展性和容错性，**因此，我们通常不使用单个Kafka Broker，而是使用由多个Broker组成的集群**。这些Broker的行为并不完全相同，但每个Broker都分配有特殊任务，如果一个Broker发生故障，集群的其余部分可以接管这些任务。

为了理解这一点，我们需要扩展对主题的理解。创建主题时，我们不仅指定分区数，还指定使用同步共同管理分区的代理数，我们称之为复制因子。例如，使用Kafka CLI，我们可以创建一个包含6个分区的主题，每个分区在3个代理上同步：

```shell
sh kafka-topics.sh --bootstrap-server localhost:9092 --create --topic my-replicated-topic --partitions 6 --replication-factor 3
```

例如，复制因子为3意味着集群最多可以承受2个副本故障(N-1弹性)。**我们必须确保至少拥有与复制因子指定的一样多的代理**，否则，Kafka不会创建主题，直到代理数量增加。

**为了提高效率，分区的复制只在一个方向上进行**，Kafka通过将其中一个代理声明为分区领导者来实现这一点。生产者只向分区领导者发送消息，然后领导者与其他代理同步。消费者也将从分区领导者那里进行轮询，因为不断增加的消费者组的偏移量也必须同步。

**分区引导分布到多个代理**，Kafka尝试为不同的分区寻找不同的代理，让我们看一个有4个代理和2个分区且复制因子为3的示例：

![](/assets/images/2025/kafka/apachekafka11.png)

Broker 1是分区1的领导者，而Broker 4是分区2的领导者。因此，每个客户端在从这些分区发送或轮询消息时都会连接到这些代理。为了获取有关分区领导者和其他可用代理(元数据)的信息，有一个特殊的引导机制。总之，我们可以说每个代理都可以提供集群的元数据，因此客户端可以初始化与每个代理的连接，然后重定向到分区领导者，这就是我们可以指定多个代理作为[引导服务器](https://www.baeldung.com/java-kafka-bootstrap-server)的原因。

如果一个分区领导者代理发生故障，Kafka将声明其中一个仍在工作的代理为新的分区领导者。然后，所有客户端都必须连接到新的领导者。在我们的示例中，如果Broker 1发生故障，Broker 2将成为分区1的新领导者；然后，连接到Broker 1的客户端必须切换到Broker 2。

![](/assets/images/2025/kafka/apachekafka12.png)

Kafka使用[Kraft(早期版本中为Zookeeper)](https://www.baeldung.com/kafka-shift-from-zookeeper-to-kraft)来协调集群内的所有Broker。

### 4.5 整合所有内容

如果我们将生产者和消费者放在一起，由3个代理组成的集群管理一个具有3个分区和复制因子为3的单个主题，我们将得到这种架构：

![](/assets/images/2025/kafka/apachekafka13.png)

## 5. 生态系统

我们已经知道有多个客户端可用于连接Kafka，例如CLI、与Spring应用程序集成的基于Java的客户端以及多个GUI工具。当然，还有其他编程语言(例如[C/C++](https://docs.confluent.io/platform/current/clients/librdkafka/html/md_INTRODUCTION.html)、[Python](https://github.com/confluentinc/confluent-kafka-python)或[Javascript](https://kafka.js.org/))的客户端API，但这些都不属于Kafka项目。

在这些API之上，还有用于特殊用途的更多API。

### 5.1 Kafka Connect API

[Kafka Connect](https://www.baeldung.com/kafka-connectors-guide)是用于与第三方系统交换数据的API，[现有的连接器](https://docs.confluent.io/platform/current/connect/kafka_connectors.html)有AWS S3、JDBC等，甚至可用于在不同的Kafka集群之间交换数据。当然，我们也可以编写自定义连接器。

### 5.2 Kafka Streams API

[Kafka Streams](https://www.baeldung.com/java-kafka-streams)是一种用于实现流处理应用程序的API，该应用程序从Kafka主题获取输入，并将结果存储在另一个Kafka主题中。

### 5.3 KSQL

KSQL是一个基于Kafka Streams构建的类似SQL的接口，它不需要我们开发Java代码，但我们可以声明类似SQL的语法来定义与Kafka交换的消息的流处理。为此，我们使用连接到Kafka集群的[ksqlDB](https://www.baeldung.com/ksqldb)。我们可以使用CLI或Java客户端应用程序访问ksqlDB。

### 5.4 Kafka REST代理

[Kafka REST代理](https://github.com/confluentinc/kafka-rest)为Kafka集群提供了RESTful接口，这样，我们就不需要任何Kafka客户端，也无需使用原生Kafka协议。它允许Web前端与Kafka连接，并可以使用API网关或防火墙等网络组件。

### 5.5 适用于Kubernetes的Kafka Operators(Strimzi)

[Strimzi](https://strimzi.io/)是一个开源项目，它提供了一种在Kubernetes和OpenShift平台上运行Kafka的方法。它引入了自定义Kubernetes资源，使得以Kubernetes原生方式声明和管理与Kafka相关的资源变得更加容易。它遵循[Operator Pattern](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/)，即操作员自动执行Kafka集群的配置、扩展、滚动更新和监控等任务。

### 5.6 基于云托管的Kafka服务

Kafka作为托管服务在常用的云平台上提供：[Amazon Managed Streaming for Apache Kafka(MSK)](https://aws.amazon.com/msk/)、Managed Service – Apache Kafka on Azure和[Google Cloud Managed Service for Apache Kafka](https://console.cloud.google.com/managedkafka/clusters)。

## 6. 总结

在本文中，我们了解到Kafka的设计具有高可扩展性和容错性。生产者收集消息并分批发送，主题被划分为分区以允许并行消息传递和消费者负载均衡，并在多个代理上进行复制以确保容错性。