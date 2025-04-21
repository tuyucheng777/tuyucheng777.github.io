---
layout: post
title:  连接到Docker中运行的Apache Kafka
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 概述

[Apache Kafka](https://kafka.apache.org/)是一个非常流行的事件流平台，经常与[Docker](https://www.docker.com/)配合使用，人们经常会遇到Kafka的连接建立问题，尤其是当客户端不在同一个Docker网络或同一主机上运行时，这主要是由于Kafka的监听器配置错误造成的。

在本教程中，我们将学习如何配置监听器，以便客户端可以连接到Docker中运行的Kafka代理。

## 2. 设置Kafka

在尝试建立连接之前，我们需要[使用Docker运行Kafka代理](https://www.baeldung.com/ops/kafka-docker-setup)。以下是[docker-compose.yaml](https://www.baeldung.com/ops/docker-compose)文件的片段：

```yaml
version: '2'
services:
    zookeeper:
        container_name: zookeeper
        networks:
            - kafka_network
        # ...

    kafka:
        container_name: kafka
        networks:
            - kafka_network
        ports:
            - 29092:29092
        environment:
            KAFKA_LISTENERS: EXTERNAL_SAME_HOST://:29092,INTERNAL://:9092
            KAFKA_ADVERTISED_LISTENERS: INTERNAL://kafka:9092,EXTERNAL_SAME_HOST://localhost:29092
            KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INTERNAL:PLAINTEXT,EXTERNAL_SAME_HOST:PLAINTEXT
            KAFKA_INTER_BROKER_LISTENER_NAME: INTERNAL
            # ... 

networks:
    kafka_network:
        name: kafka_docker_example_net
```

这里，我们定义了两个必备服务Kafka和Zookeeper，我们还定义了一个自定义网络kafka_docker_example_net，我们的服务将使用它。

稍后我们将更详细地了解KAFKA_LISTENERS、KAFKA_ADVERTISED_LISTENERS和KAFKA_LISTENER_SECURITY_PROTOCOL_MAP属性。

通过上面的docker-compose.yaml文件，我们启动服务：

```shell
docker-compose up -d
Creating network "kafka_docker_example_net" with the default driver
Creating zookeeper ... done
Creating kafka ... done
```

此外，我们将使用Kafka[控制台生产者](https://kafka-tutorials.confluent.io/kafka-console-consumer-producer-basics/kafka.html)实用程序作为示例客户端来测试与Kafka代理的连接，要在不使用Docker的情况下使用Kafka-console-producer脚本，我们需要下载[Kafka](https://kafka.apache.org/downloads)。

## 3. 监听器

在与Kafka代理连接时，监听器、通告监听器和监听器协议发挥着重要作用。

我们使用KAFKA_LISTENERS属性来管理监听器，其中我们声明一个以逗号分隔的URI列表，该列表指定代理应该监听传入TCP连接的套接字。

每个URI包含一个协议名称，后跟一个接口地址和一个端口：

```text
EXTERNAL_SAME_HOST://0.0.0.0:29092,INTERNAL://0.0.0.0:9092
```

这里，我们指定了一个0.0.0.0元地址，用于将套接字绑定到所有接口。此外，EXTERNAL_SAME_HOST和INTERNAL是自定义监听器的名称，在以URI格式定义监听器时需要指定它们。

### 3.1 引导

对于初始连接，Kafka客户端需要一个引导服务器列表，其中包含Broker的地址，该列表应至少包含一个指向集群中某个随机Broker的有效地址。

客户端将使用该地址连接到代理，如果连接成功，代理将返回有关集群的元数据，包括集群中所有代理的已通告监听器列表。对于后续连接，客户端将使用该列表访问代理。

### 3.2 通告监听器

仅仅声明监听器是不够的，因为它只是Broker的一个套接字配置，我们需要一种方法来告诉客户端(消费者和生产者)如何连接到Kafka。

此时，借助KAFKA_ADVERTISED_LISTENERS属性，通告监听器就派上用场了，它的格式与监听器的属性类似：

<侦听器协议\>://<通告的主机名\>:<通告的端口\>

初始引导过程之后，客户端使用指定为通告监听器的地址。

### 3.3 监听器安全协议映射

除了监听器和通告监听器之外，我们还需要告知客户端连接Kafka时要使用的安全协议。在KAFKA_LISTENER_SECURITY_PROTOCOL_MAP中，我们将自定义协议名称映射到有效的安全协议。

在上一节的配置中，我们声明了两个自定义协议名称INTERNAL和EXTERNAL_SAME_HOST，我们可以随意命名它们，但需要将它们映射到有效的安全协议。

我们指定的安全协议之一是PLAINTEXT，这意味着客户端无需向Kafka代理进行身份验证。此外，交换的数据未加密。

## 4. 从同一Docker网络连接的客户端

让我们从另一个容器启动Kafka控制台生产者并尝试向代理生成消息：

```shell
docker run -it --rm --network kafka_docker_example_net confluentinc/cp-kafka /bin/kafka-console-producer --bootstrap-server kafka:9092 --topic test_topic
>hello
>world
```

在这里，我们将此容器连接到现有的kafka_docker_example_net网络，以便与我们的代理自由通信。我们还指定了代理的地址kafka:9092以及主题的名称，该名称将自动创建。

我们能够向主题生成消息，这意味着与代理的连接已成功。

## 5. 从同一主机连接的客户端

当客户端未容器化时，让我们从主机连接到代理。对于外部连接，我们公布了EXTERNAL_SAME_HOST监听器，我们可以使用它从主机建立连接。从通告监听器属性中，我们知道必须使用localhost:29092地址才能访问Kafka代理。

为了测试来自同一主机的连接，我们将使用非容器化Kafka控制台生产者：

```shell
kafka-console-producer --bootstrap-server localhost:29092 --topic test_topic_2
>hi
>there
```

由于我们成功地生成了主题，这意味着初始引导和后续与代理的连接(客户端使用通告监听器)都成功了。

我们之前在docker-compose.yaml中配置的端口号29092使得Kafka代理可以在Docker外部访问。

## 6. 客户端从不同的主机连接

如果Kafka代理运行在不同的主机上，我们该如何连接它呢？很遗憾，我们无法复用现有的监听器，因为它们只适用于相同的Docker网络或主机连接。因此，我们需要定义一个新的监听器并进行通告：

```shell
KAFKA_LISTENERS: EXTERNAL_SAME_HOST://:29092,EXTERNAL_DIFFERENT_HOST://:29093,INTERNAL://:9092
KAFKA_ADVERTISED_LISTENERS: INTERNAL://kafka:9092,EXTERNAL_SAME_HOST://localhost:29092,EXTERNAL_DIFFERENT_HOST://157.245.80.232:29093
KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INTERNAL:PLAINTEXT,EXTERNAL_SAME_HOST:PLAINTEXT,EXTERNAL_DIFFERENT_HOST:PLAINTEXT
```

我们创建了一个名为EXTERNAL_DIFFERENT_HOST的新监听器，其安全协议为PLAINTEXT，关联端口为29093。在KAFKA_ADVERTISED_LISTENERS中，我们还添加了运行Kafka的云服务器的IP地址。

需要注意的是，我们不能使用localhost，因为我们是从另一台机器(本例中是本地机器)连接的。此外，端口29093已发布在ports部分下，因此Docker外部也可以访问它。

让我们尝试生成一些消息：

```shell
kafka-console-producer --bootstrap-server 157.245.80.232:29093 --topic test_topic_3
>hello
>REMOTE SERVER
```

我们可以看到我们能够连接到Kafka代理并成功生成消息。

## 7. 总结

在本文中，我们学习了如何配置监听器，以便客户端能够连接到Docker中运行的Kafka代理。我们研究了客户端运行在同一个Docker网络、同一个主机、不同主机等不同场景。我们发现，监听器、通告监听器和安全协议映射的配置决定了连接性。