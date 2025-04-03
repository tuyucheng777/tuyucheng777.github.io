---
layout: post
title:  使用Java处理Kafka Producer TimeOutException
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 概述

在本教程中，我们将学习如何处理[Kafka Producer](https://www.baeldung.com/apache-kafka#1-producers-amp-consumers)中的TimeOutException。

首先，让我们了解一下TimeOutException发生时的可能情况，然后看看如何解决该问题。

## 2. Kafka Producer中的TimeOutException

我们通过创建ProducerRecord开始向Kafka生成消息，它必须包含我们要发送记录的主题和一个值。我们还可以可选地指定键、分区、时间戳和/或标头集合。

然后分区器会为我们选择一个[分区](https://www.baeldung.com/kafka-topics-partitions)，通常基于[ProducerRecord](https://www.baeldung.com/java-kafka-message-key#2-publish-messages)键。**一旦分区器选择了分区，生产者就会确定该记录的主题和分区。然后，生产者会将该记录添加到一批记录中，并将其发送到同一主题和分区(我们将其视为缓冲区)。一个单独的线程负责将这些记录批次发送到适当的Kafka代理**。

Kafka在从生产者向代理发送消息时使用缓冲概念，一旦我们从KafkaProducer调用send()方法来发送ProducerRecord，系统就会将消息放在缓冲区中，并在单独的线程中将其发送到缓冲区。

请求超时或者批量过大会导致Kafka Producer抛出TimeOutException异常，即超出缓冲区限制或者遇到网络瓶颈。我们来一一了解一下。

## 3. 请求超时

一旦我们将记录添加到批次中，我们就需要在指定的时间内发送该批次，以确保按时发送。配置参数request.timeout.ms控制时间限制，默认为30秒：

```java
producerProperties.put(ProducerConfig.REQUEST_TIMEOUT_MS_CONFIG, 60000);
```

我们更改了请求超时时间，以便有更多时间发送每个批次。一旦批次排队时间超过60秒，我们就会收到TimeOutException。

## 4. 大批量

kafka生产者会等待将缓冲区中的数据发送到代理，直到满足批处理大小。如果生产者不满足批处理大小，则请求超时。因此，我们可以减小批处理大小并降低请求超时的可能性：

```java
producerProperties.put(ProducerConfig.BATCH_SIZE_CONFIG, 100000);
```

通过减少批处理大小，我们可以更频繁地向代理发送更少数量的批处理消息，这也许可以避免TimeOutException。

## 5. 网络瓶颈

如果我们以高于发送方线程处理能力的速率向代理发送消息，则可能导致网络瓶颈，从而引发TimeOutException。我们可以使用配置linger.ms来处理这个问题：

```java
producerProperties.put(ProducerConfig.LINGER_MS_CONFIG, 10);
```

linger.ms属性控制在发送当前批次之前等待其他消息的时间，当KafkaProducer填满当前批次或达到linger.ms限制时，它会发送一批消息。

默认情况下，只要有生产者线程可用，生产者就会立即发送消息，即使批次中只有一条消息。通过将linger.ms设置为大于0，我们指示生产者等待几毫秒，将其他消息添加到批次中，然后再将其发送到代理。

这会稍微增加延迟并显著增加吞吐量-每条消息的开销要低得多，而且如果启用压缩，效果会更好。

## 6. 复制因子

Kafka提供了复制策略的配置，Topic级别的配置和Broker级别的配置均参考min.insync.replicas。

**如果复制因子小于min.insync.replicas，则写入不会获得足够的确认，因此会超时**。重新创建复制因子 > min.insync.replicas的主题可以解决此问题。

在配置集群以实现数据持久性时，我们可以通过将min.insync.replicas设置为2来确保生产者至少有两个副本已赶上并“同步”。我们应该使用此设置以及配置生产者以确认“所有”请求，这可确保至少有两个副本(领导者和另一个副本)确认写入才能成功。

这可以防止在领导者请求写入，然后遭遇故障，并且领导权被转移到没有成功写入的副本的情况下发生数据丢失。如果没有这些持久设置，生产者会认为它已成功生成，并且消息将被丢弃并丢失。

但是，配置更高的持久性会导致效率降低，因为会产生额外的开销，因此kafka不建议具有高吞吐量且可以容忍偶尔的消息丢失的集群将此设置从默认值1更改。

## 7. 引导服务器地址

一些与网络相关的问题也可能导致TimeOutException。

防火墙可能会阻塞Kafka端口，无论是在生产者端、代理端还是中间的某个地方。从运行生产者的服务器尝试nc -z broker-ip <port_number\>：

```shell
$ nc -z  192.168.123.132 9092
```

我们可以发现是否防火墙阻塞了该端口。

如果DNS解析出现问题，即使端口是开放的，生产者也无法找到IP地址。因此，如果其他事情都没有问题，我们也可以检查这一点。

## 8. 总结

在本文中，我们了解到KafkaProducer类中的TimeOutException可能是由请求超时、批处理大小或网络瓶颈引起的。我们还讨论了其他可能性，例如错误的复制因子或服务器地址配置。