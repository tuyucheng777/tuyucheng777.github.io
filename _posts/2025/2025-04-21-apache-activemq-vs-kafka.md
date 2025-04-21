---
layout: post
title:  Apache ActiveMQ与Kafka
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1.概述

在分布式架构中，应用程序通常需要在彼此之间交换数据。一方面，这可以通过彼此直接通信来实现。另一方面，为了实现高可用性和分区容忍度，并保持应用程序之间的松耦合，消息传递是一个合适的解决方案。

因此，我们可以在多种产品中进行选择。Apache基金会提供了ActiveMQ和Kafka，我们将在本文中对它们进行比较。

## 2. 一般事实

### 2.1 Active MQ

**Active MQ是传统的消息代理之一，其目标是确保应用程序之间以安全可靠的方式交换数据**。它处理的数据量较小，因此专门用于定义明确的消息格式和事务性消息传递。

值得注意的是，除了这个“经典”版本之外，还有另一个版本Active MQ Artemis，这款下一代代理基于HornetQ，后者的代码已于2015年由RedHat提供给Apache基金会，[Active MQ官网](https://activemq.apache.org/)上写道：

> 一旦Artemis的功能达到与“Classic”代码库相当的水平，它将成为ActiveMQ的下一个主要版本。

因此，为了进行比较，我们需要考虑两个版本，我们将使用术语“Active MQ”和“Artemis”来区分它们。

### 2.2 Kafka

与Active MQ相比，**Kafka是一个用于处理海量数据的分布式系统**。我们不仅可以使用它进行传统的消息传递，还可以用来：

- 网站活动追踪
- 指标
- 日志聚合
- 流处理
- 事件溯源
- 提交日志

随着使用微服务构建的典型云架构的出现，这些要求变得非常重要。

### 2.3 JMS的作用和消息传递的演变

Java消息服务(JMS)是Java EE应用程序中用于发送和接收消息的通用API，它是消息系统早期发展的一部分，至今仍是标准。在Jakarta EE中，它被采用为Jakarta Messaging。因此，了解以下核心概念可能会有所帮助：

- Java原生但独立于供应商的API
- 需要JCA资源适配器来实现特定于供应商的通信协议
- 消息目的地模型：
  - 队列(P2P)确保消息排序，即使在有多个消费者的情况下也能一次性处理消息
  - 主题(PubSub)是发布-订阅模式的一种实现，这意味着多个消费者将在订阅主题期间接收消息
- 消息格式：
  - 标头作为代理处理的标准化元信息(如优先级或到期日期)
  - 属性是消费者可用于消息处理的非标准化元信息
  - 包含有效负载的Body-JMS声明了5种类型的消息，但这仅与使用API相关，与本次比较无关

**然而，演进的方向是开放和独立的-独立于消费者和生产者的平台，也独立于消息代理的供应商**，有一些协议定义了它们自己的目标模型：

- [AMQP](https://www.amqp.org/)：用于独立于供应商的消息传递的二进制协议-使用通用节点
- [MQTT](https://mqtt.org/)：用于嵌入式系统和物联网的轻量级二进制协议-使用主题
- [STOMP](https://stomp.github.io/)：一种简单的基于文本的协议，甚至允许通过浏览器发送消息-使用通用目的地

**另一项发展是，通过云架构的普及，将之前可靠的单条消息传输(“传统消息传递”)添加到根据“发射后不理睬”原则处理大量数据中**。可以说，Active MQ与Kafka之间的比较，实际上是对这两种方法的典型代表的比较。例如，[NATS](https://nats.io/)可以作为Kafka的替代方案。

## 3. 比较

在本节中，我们将比较Active MQ和Kafka之间架构和开发中最有趣的特性。

### 3.1 消息目标模型、协议和API

Active MQ完全实现了JMS消息目标模型(队列和主题)，并将AMQP、MQTT和STOMP消息映射到这些模型。例如，STOMP消息会映射到主题(Topic)中的JMS BytesMessage消息。此外，它还支持[OpenWire协议](https://activemq.apache.org/openwire)，允许跨语言访问Active MQ。

Artemis独立于标准API和协议定义了自己的消息目标模型，并且还需要将它们映射到该模型：

- 消息被发送到一个地址，该地址被赋予一个唯一的名称、一个路由类型以及0个或多个队列。

- 路由类型决定了消息如何从某个地址路由到绑定到该地址的队列，路由类型定义了两种类型：

  - ANYCAST：消息被路由到地址上的单个队列
  - 多播：消息被路由到地址上的每个队列

Kafka仅定义了主题(Topic)，主题由多个分区(Partition)(至少1个)和可放置在不同Broker上的副本(Replica)组成，找到对主题进行分区的最佳策略是一项挑战，我们必须注意：

- 一条消息分发到一个分区
- 仅确保一个分区内的消息的排序
- 默认情况下，后续消息将在主题的分区之间循环分发
- 如果我们使用消息键，那么具有相同键的消息将落入同一个分区

Kafka有自己的[API](https://kafka.apache.org/documentation/#api)，虽然也有[JMS资源适配器](https://docs.payara.fish/enterprise/docs/documentation/ecosystem/cloud-connectors/apache-kafka.html)，但我们应该意识到这些概念并不完全兼容。AMQP、MQTT和STOMP尚未得到官方支持，但有[AMQP](https://github.com/ppatierno/kafka-connect-amqp)和[MQTT](https://github.com/johanvandevenne/kafka-connect-mqtt)[连接器](https://www.baeldung.com/kafka-connectors-guide)。

### 3.2 消息格式和处理

Active MQ支持JMS标准消息格式，该格式由消息头、属性和消息体组成(如上所述)。代理必须维护每条消息的传递状态，从而导致吞吐量降低。由于Active MQ支持JMS，因此消费者可以同步从目标拉取消息，也可以异步接收代理推送的消息。

Kafka没有定义任何消息格式-这完全由生产者负责。每条消息没有任何投递状态，只有每个消费者和分区的偏移量(Offset)。偏移量是最后一条已投递消息的索引，这不仅速度更快，而且还允许通过重置偏移量来重新发送消息，而无需询问生产者。

### 3.3 Spring与CDI集成

JMS是Java/Jakarta EE标准，因此已完全集成到Java/Jakarta EE应用程序中。因此，应用服务器可以轻松管理与Active MQ和Artemis的连接。借助Artemis，我们甚至可以使用[嵌入式代理](https://activemq.apache.org/components/artemis/documentation/latest/cdi-integration.html)。对于Kafka，只有在使用[JMS资源适配器](https://docs.payara.fish/enterprise/docs/documentation/ecosystem/cloud-connectors/apache-kafka.html)或[Eclipse MicroProfile Reactive](https://dzone.com/articles/using-jakarta-eemicroprofile-to-connect-to-apache)时才可以使用托管连接。

Spring集成了[JMS](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms)以及[AMQP](https://spring.io/projects/spring-amqp)、[MQTT](https://docs.spring.io/spring-integration/reference/mqtt.html)和[STOMP](https://docs.spring.io/spring-integration/reference/stomp.html)。此外，它还支持[Kafka](https://spring.io/projects/spring-kafka)。借助Spring Boot，我们可以为[Active MQ](https://memorynotfound.com/spring-boot-embedded-activemq-configuration-example/)、[Artemis](https://activemq.apache.org/components/artemis/documentation/1.0.0/spring-integration.html)和[Kafka](https://www.baeldung.com/spring-boot-kafka-testing)使用嵌入式代理。

## 4. Active MQ/Artemis和Kafka的用例

以下几点为我们指明了何时使用哪种产品最好。

### 4.1 Active MQ/Artemis的用例

- 每天仅处理少量消息
- 高度的可靠性和交易性
- 动态数据转换、ETL作业

### 4.2 Kafka的用例

- 处理大量数据
  - 实时数据处理
  - 应用程序活动跟踪
  - 日志记录和监控
- 无需数据转换的消息传递
- 没有传输保证的消息传递