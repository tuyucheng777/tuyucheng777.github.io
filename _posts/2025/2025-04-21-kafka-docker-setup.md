---
layout: post
title:  使用Docker设置Apache Kafka的指南
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 概述

**Docker是软件行业用于创建、打包和部署应用程序的最流行的容器引擎之一**。

在本教程中，我们将学习如何使用Docker执行[Apache Kafka](https://www.baeldung.com/spring-kafka#overview)设置。

至关重要的是，自2.8.0版本以来，Apache Kafka支持不依赖ZooKeeper的模式。**此外，从Confluent Platform 7.5开始，ZooKeeper已弃用。因此，我们特意将ZooKeeper和Kafka容器的版本都标记为7.4.4**。

## 2. 单节点设置

单节点Kafka代理设置可以满足大多数本地开发需求，因此让我们从学习这个简单的设置开始。

### 2.1 docker-compose.yml配置

要启动Apache Kafka服务器，我们首先需要启动[Zookeeper](https://www.baeldung.com/java-zookeeper#overview)服务器。

**我们可以在[docker-compose.yml](https://www.baeldung.com/docker-compose)文件中配置这种依赖关系**，以确保Zookeeper服务器始终在Kafka服务器之前启动并在其之后停止。

让我们创建一个简单的docker-compose.yml文件，其中包含两个服务，即zookeeper和kafka：

```shell
$ cat docker-compose.yml
version: '2'
services:
    zookeeper:
        image: confluentinc/cp-zookeeper:7.4.4
        environment:
            ZOOKEEPER_CLIENT_PORT: 2181
            ZOOKEEPER_TICK_TIME: 2000
        ports:
            - 22181:2181

    kafka:
        image: confluentinc/cp-kafka:7.4.4
        depends_on:
            - zookeeper
        ports:
            - 29092:29092
        environment:
            KAFKA_BROKER_ID: 1
            KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
            KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092,PLAINTEXT_HOST://localhost:29092
            KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
            KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
            KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
```

在此设置中，我们的Zookeeper服务器在2181端口监听kafka服务，该服务定义在同一容器设置中。但是，对于主机上运行的任何客户端，该服务都会暴露在22181端口上。

类似地，**kafka服务通过端口29092暴露给主机应用程序**，但实际上它是在**由KAFKA_ADVERTISED_LISTENERS属性配置**的容器环境内的端口9092上进行通知的。

### 2.2 启动Kafka服务器

让我们通过使用[docker-compose](https://www.baeldung.com/ops/docker-compose)命令启动容器来启动Kafka服务器：

```shell
$ docker-compose up -d
Creating network "kafka_default" with the default driver
Creating kafka_zookeeper_1 ... done
Creating kafka_kafka_1     ... done
```

现在，让我们**使用[nc](https://www.baeldung.com/linux/netcat-command#scanning-for-open-ports-using-netcat)命令来验证两个服务器是否都在监听各自的端口**：

```shell
$ nc -zv localhost 22181
Connection to localhost port 22181 [tcp/*] succeeded!
$ nc -zv localhost 29092
Connection to localhost port 29092 [tcp/*] succeeded!
```

此外，我们还可以在容器启动时检查详细日志并验证Kafka服务器是否启动：

```shell
$ docker-compose logs kafka | grep -i started
kafka_1      | [2024-02-26 16:06:27,352] DEBUG [ReplicaStateMachine controllerId=1] Started replica state machine with initial state -> HashMap() (kafka.controller.ZkReplicaStateMachine)
kafka_1      | [2024-02-26 16:06:27,354] DEBUG [PartitionStateMachine controllerId=1] Started partition state machine with initial state -> HashMap() (kafka.controller.ZkPartitionStateMachine)
kafka_1      | [2024-02-26 16:06:27,365] INFO [KafkaServer id=1] started (kafka.server.KafkaServer)
```

这样，我们的Kafka设置就可以使用了。

### 2.3 使用Kafka Tool连接

最后，让我们**使用[Kafka Tool](https://kafkatool.com/download.html) GUI实用程序与我们新创建的Kafka服务器建立连接**，稍后我们将可视化此设置：

![](/assets/images/2025/kafka/kafkadockersetup01.png)

值得注意的是，我们可能需要使用**Bootstrap服务器属性**从主机连接到监听端口29092的Kafka服务器。

最后，我们应该能够在左侧边栏上看到连接：

![](/assets/images/2025/kafka/kafkadockersetup02.png)

因此，由于这是一个新设置，因此“Topics”和“Consumers”条目为空。一旦我们有了一些主题，我们就应该能够跨分区可视化数据。此外，如果有活跃的消费者连接到我们的Kafka服务器，我们也可以查看它们的详细信息。

## 3. Kafka集群设置

为了获得更稳定的环境，我们需要一个弹性的设置，让我们**扩展docker-compose.yml文件来创建一个多节点的Kafka集群**。

### 3.1 docker-compose.yml配置

Apache Kafka的集群设置需要为Zookeeper服务器和Kafka服务器提供冗余。

因此，让我们为Zookeeper和Kafka服务各添加一个节点的配置：

```yaml
$ cat docker-compose.yml
---
version: '2'
services:
    zookeeper-1:
        image: confluentinc/cp-zookeeper:7.4.4
        environment:
            ZOOKEEPER_CLIENT_PORT: 2181
            ZOOKEEPER_TICK_TIME: 2000
        ports:
            - 22181:2181

    zookeeper-2:
        image: confluentinc/cp-zookeeper:7.4.4
        environment:
            ZOOKEEPER_CLIENT_PORT: 2181
            ZOOKEEPER_TICK_TIME: 2000
        ports:
            - 32181:2181

    kafka-1:
        image: confluentinc/cp-kafka:7.4.4
        depends_on:
            - zookeeper-1
            - zookeeper-2

        ports:
            - 29092:29092
        environment:
            KAFKA_BROKER_ID: 1
            KAFKA_ZOOKEEPER_CONNECT: zookeeper-1:2181,zookeeper-2:2181
            KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka-1:9092,PLAINTEXT_HOST://localhost:29092
            KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
            KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
            KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
    kafka-2:
        image: confluentinc/cp-kafka:7.4.4
        depends_on:
            - zookeeper-1
            - zookeeper-2
        ports:
            - 39092:39092
        environment:
            KAFKA_BROKER_ID: 2
            KAFKA_ZOOKEEPER_CONNECT: zookeeper-1:2181,zookeeper-2:2181
            KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka-2:9092,PLAINTEXT_HOST://localhost:39092
            KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
            KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
            KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
```

重要的是，**我们应该确保服务名称和KAFKA_BROKER_ID在各个服务中是唯一的**。

此外，**每个服务必须向主机公开一个唯一的端口**，虽然zookeeper-1和zookeeper-2监听的是端口2181，但它们分别通过端口22181和32181将其公开给主机。同样的逻辑也适用于kafka-1和kafka-2服务，它们分别监听端口29092和39092。

### 3.2 启动Kafka集群

让我们使用docker-compose命令来启动集群：

```shell
$ docker-compose up -d
Creating network "kafka_default" with the default driver
Creating kafka_zookeeper-1_1 ... done
Creating kafka_zookeeper-2_1 ... done
Creating kafka_kafka-2_1     ... done
Creating kafka_kafka-1_1     ... done
```

集群启动后，让我们使用Kafka Tool通过为Kafka服务器和相应端口指定逗号分隔的值来连接到集群：

![](/assets/images/2025/kafka/kafkadockersetup03.png)

最后我们来看一下集群中可用的多个代理节点：

![](/assets/images/2025/kafka/kafkadockersetup04.png)

## 4. 总结

在本文中，我们使用Docker技术创建Apache Kafka的单节点和多节点设置。

我们还使用Kafka Tool来连接和可视化配置的代理服务器详细信息。