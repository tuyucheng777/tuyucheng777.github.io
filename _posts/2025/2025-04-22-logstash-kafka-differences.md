---
layout: post
title:  Logstash与Kafka
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 概述

Logstash和Kafka是管理实时数据流的两个强大工具，Kafka作为分布式事件流平台表现出色，而Logstash则是一个数据处理管道，用于提取、过滤数据并将其转发到各种输出。

在本教程中，我们将更详细地研究Kafka和Logstash之间的区别，并提供它们的使用示例。

## 2. 要求

在了解Logstash和Kafka之间的区别之前，让我们确保已经安装了几个先决条件，并具备所涉及技术的基本知识。首先，我们需要安装[Java 8或更高版本](https://docs.oracle.com/en/java/javase/17/install/overview-jdk-installation.html)。

Logstash是[ELK堆栈](https://www.elastic.co/guide/en/elastic-stack/current/installing-elastic-stack.html)(Elasticsearch、Logstash、Kibana)的一部分，但可以独立安装和使用。对于Logstash，我们可以访问[Logstash官方下载页面](https://www.elastic.co/downloads/logstash)，并下载适合我们操作系统(Linux、macOS或 Windows)的软件包。

我们还需要安装[Kafka](https://www.baeldung.com/apache-kafka)，并对[发布者-订阅者模型](https://www.baeldung.com/cs/publisher-subscriber-model)有一定理解。

## 3. Logstash

让我们看一下主要的Logstash组件和处理日志文件的命令行示例。

### 3.1 Logstash组件

**Logstash是ELK Stack中的一个开源数据处理管道，用于收集、处理和转发来自多个来源的数据**。它由几个核心组件组成，这些组件协同工作以收集、转换和输出数据：

1. **输入**：这些输入将数据从各种来源(例如日志文件、数据库、消息队列(如Kafka)或云服务)导入Logstash，输入定义了原始数据的来源。
2. **过滤器**：这些组件负责处理和转换数据，常见的过滤器包括用于解析非结构化数据的[Grok](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html)、用于修改字段的mutate以及用于时间戳格式化的date，过滤器允许在将数据发送到最终目的地之前进行深度定制和数据准备。
3. **输出**：处理后，输出将数据发送到Elasticsearch、数据库、消息队列或本地文件等目标位置，Logstash支持多个并行输出，非常适合将数据分发到各个端点。
4. **[编解码器](https://www.elastic.co/guide/en/logstash/current/codec-plugins.html)**：编解码器对数据流进行编码和解码，例如将JSON转换为结构化对象或读取纯文本，它们充当微型插件，在数据被提取或发送时对其进行处理。
5. **管道**：管道是通过输入、过滤器和输出的定义数据流，管道可以创建复杂的工作流，实现多阶段数据处理。

这些组件协同工作，使Logstash成为集中日志、转换数据和与各种外部系统集成的强大工具。

### 3.2 Logstash示例

让我们举个例子来说明如何将输入文件处理为JSON格式的输出，让我们在/tmp目录中创建一个example.log输入文件：

```text
2024-10-12 10:01:15 INFO User login successful
2024-10-12 10:05:32 ERROR Database connection failed
2024-10-12 10:10:45 WARN Disk space running low
```

然后我们可以通过提供配置来运行logstash-e命令：

```shell
$ sudo logstash -e '
input { 
  file { 
    path => "/tmp/example.log" 
    start_position => "beginning" 
    sincedb_path => "/dev/null" 
  } 
} 
filter { 
  grok { 
    match => { "message" => "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:loglevel} %{GREEDYDATA:message}" }
  } 
  mutate {
    remove_field => ["log", "timestamp", "event", "@timestamp"]
  }
}
output { 
  file {
    path => "/tmp/processed-logs.json"
    codec => json_lines
  }
}'
```

让我们解释一下配置的不同部分：

- 整个命令链(input/filter/output)是一个管道。
- 使用grok过滤器从日志中提取时间戳、日志级别和消息字段。
- 使用mutate过滤器删除不必要的信息。
- 在output过滤器中应用带有编解码器的JSON格式。
- 输入的example.log文件处理后，输出将在processed-log.json文件中以JSON格式编码。

让我们看一个输出示例：

```text
{"message":["2024-10-12 10:05:32 ERROR Database connection failed","Database connection failed"],"host":{"name":"tuyucheng"},"@version":"1"}
{"message":["2024-10-12 10:10:45 WARN Disk space running low","Disk space running low"],"host":{"name":"tuyucheng"},"@version":"1"}
{"message":["2024-10-12 10:01:15 INFO User login successful","User login successful"],"host":{"name":"tuyucheng"},"@version":"1"}
```

我们可以看到，输出文件是JSON，其中包含附加信息，例如@version，我们可以使用它来记录更改并确保任何下游进程(如Elasticsearch中的查询)都知道它以保持数据一致性。

## 4. Kafka

让我们看一下Kakfa的主要组件以及发布和消费消息的命令行示例。

### 4.1 Kafka组件

**[Apache Kafka](https://www.baeldung.com/apache-kafka)是一个开源分布式事件流平台，用于构建实时数据管道和应用程序**。

我们来看看它的主要组成部分：

1. **[主题和分区](https://www.baeldung.com/kafka-topics-partitions)**：Kafka将消息组织成不同的类别，称为主题，每个主题又被划分为多个分区，从而允许在多个服务器上并行处理数据。例如，在电商应用程序中，订单数据、支付交易和用户活动日志可能被分别设置为不同的主题。
2. **生产者和消费者**：生产者将数据(消息)发布到Kafka主题，而消费者是读取和处理这些消息的应用程序或服务。生产者将数据推送到Kafka的分布式代理，以确保可扩展性；而消费者可以订阅主题并从特定分区读取消息，Kafka保证消费者按顺序读取每条消息。
3. **代理**：Kafka代理(Broker)是存储和管理主题分区的服务器，多个Broker组成Kafka集群，负责分发数据并确保容错能力。如果一个Broker发生故障，其他Broker将接管数据，从而提供高可用性。
4. **[Kafka Streams](https://www.baeldung.com/java-kafka-streams)和[Kafka Connect](https://www.baeldung.com/kafka-connectors-guide)**：Kafka Streams是一个强大的流处理库，允许直接从Kafka主题进行实时数据处理。因此，它使应用程序能够动态处理和转换数据，例如计算实时分析或检测金融交易中的模式。另一方面，Kafka Connect简化了Kafka与外部系统的集成，它提供了用于集成数据库、云服务和其他应用程序的连接器。
5. **[ZooKeeper和KRaft](https://www.baeldung.com/kafka-shift-from-zookeeper-to-kraft)**：传统上，Kafka使用ZooKeeper进行分布式配置管理，包括管理Broker元数据和分区复制的Leader选举。随着KRaft(Kafka Raft)的推出，Kafka现在支持无ZooKeeper架构，但ZooKeeper在许多设置中仍然很常用。

这些组件共同使Kafka能够提供一个可扩展、容错、分布式消息传递平台，可以处理大量流数据。

### 4.2 Kafka示例

让我们创建一个主题，发布一个简单的“Hello,World”消息，并消费它。

首先，让我们创建一个主题，它可以属于多个分区，通常代表我们域中的一个主题：

```shell
$ /bin/kafka-topics.sh \
  --create \
  --topic hello-world \
  --bootstrap-server localhost:9092 \
  --partitions 1 \
  --replication-factor 1
```

我们将收到主题创建的消息：

```shell
$ Created topic hello-world.
```

现在让我们尝试向主题发送一条消息：

```shell
$ /bin/kafka-console-producer.sh \
  --topic hello-world \
  --bootstrap-server localhost:9092 \
  <<< "Hello, World!"
```

现在，可以消费我们的消息：

```shell
$ /bin/kafka-console-consumer.sh \
  --topic hello-world \
  --from-beginning \
  --bootstrap-server localhost:9092
```

我们将通过使用Kafka日志存储来获取该特定主题的消息：

```text
Hello, World!
```

## 5. Logstash和Kafka之间的核心区别

Logstash和Kafka是现代数据处理架构不可或缺的组成部分，它们各自发挥着独特而又互补的作用。

### 5.1 Logstash

Logstash是一个开源数据处理管道，专门用于提取数据、转换数据并将结果发送到各种输出。**它的优势在于其解析和丰富数据的能力，使其成为处理日志和事件数据的理想选择**。

例如，一个典型的用例可能涉及一个Web应用程序，其中Logstash从多个服务器采集日志。然后，它应用过滤器来提取相关字段，例如时间戳和错误消息。最后，它将这些丰富的数据转发到Elasticsearch，以便在Kibana中进行索引和可视化，从而监控应用程序性能并诊断实时问题。

### 5.2 Kafka

相比之下，Kafka是一个分布式流媒体平台，擅长处理高吞吐量、容错和实时数据流。**它充当消息代理，促进记录流的发布和订阅**。

例如，在电商架构中，Kafka可以捕获来自各种服务的用户活动事件，例如网站点击、购买和库存更新。这些事件可以生成到Kafka主题中，从而允许多个下游服务(例如推荐引擎、分析平台和通知系统)实时使用这些数据。

### 5.3 差异

Logstash专注于数据转换、丰富原始日志并将其发送到各个目的地，而Kafka则强调可靠的消息传递和流处理，允许跨不同系统的实时数据流。

让我们看看主要的区别：

| 特征     | Logstash| Kafka                   |
|--------| ------------------------------------------------------------ |-------------------------|
| **主要目的**   | 日志和事件数据的数据收集、处理和转换管道| 用于实时数据流的分布式消息代理         |
| **架构**     | 基于插件的管道，具有输入、过滤器和输出来处理数据流| 基于集群，生产者和消费者通过代理和主题进行交互 |
| **消息保留**   | 实时处理数据，通常不会永久存储数据| 按可配置的保留期存储消息，从而实现消息重播   |
| **数据提取**   | 使用多个输入插件从多个来源(日志、文件、数据库等)提取数据| 以可扩展、分布式的方式从生产者那里获取大量数据 |
| **数据转换**   | 使用grok、mutate和 GeoIP等过滤器进行强大的数据转换| 有限的数据转换(通常在下游系统中完成)     |
| **消息传递保证** | 处理流程中的数据；没有内置的消息传递语义保证| 支持传递语义：至少一次、最多一次或恰好一次   |
| **整合焦点**   | 主要集成各种数据源并将其转发到Elasticsearch、数据库或文件等存储/监控系统| 主要集成分布式数据流系统和分析平台       |
| **典型用例**   | 集中日志记录、数据解析、转换和实时系统监控| 事件驱动架构、流分析、分布式日志记录和数据管道 |

它们共同帮助组织构建强大的数据管道，促进实时洞察和决策，展示其在不断发展的数据架构格局中的关键作用。

## 6. Logstash和Kafka可以一起使用吗？

Logstash和Kafka可以无缝协作以创建强大的数据处理管道，结合它们的优势来增强数据提取、处理和交付。

### 6.1 从Logstash

例如，Logstash可以充当数据收集器和处理器，提取各种数据源(例如日志、指标和事件)，然后将这些数据转换为特定的格式或模式。例如，在微服务架构中，Logstash可以从各种微服务收集日志，应用过滤器提取相关信息，然后将结构化数据转发到Kafka主题进行进一步处理。

### 6.2 至Kafka

数据一旦存储到Kafka中，就可以被需要实时处理和分析的多种应用程序和服务使用。例如，金融机构可以使用Kafka从其支付处理系统中流式传输交易数据，供各种应用程序(包括欺诈检测系统、分析平台和报告工具)使用。

### 6.3 LogStash与Kafka

Logstash简化了日志和事件的初始提取和转换；同时，Kafka是一个可扩展、容错的消息传递主干，可确保在整个架构中可靠地传递数据。

**通过集成Logstash和Kafka，企业可以构建强大而灵活的数据管道，高效处理大量数据，实现实时分析和洞察**。这种协作将数据提取与处理分离，从而提高数据架构的可扩展性和弹性。

## 7. 总结

在本教程中，我们通过提供架构和命令行示例了解了Logstash和Kafka的工作原理，我们了解了它们的主要用途，并通过描述其主要组件来说明它们各自的最佳实际用途。最后，我们了解了这两个系统之间的主要区别以及它们如何协同工作。