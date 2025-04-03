---
layout: post
title:  Kafka生产者重试
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 概述

在这篇短文中，我们将探讨Kafka Producer的重试机制以及如何定制其设置以适应特定用例。

我们将讨论关键属性及其默认值，然后根据我们的示例对其进行自定义。

## 2. 默认配置

**Kafka Producer的默认行为是当消息未被代理确认时重试发布**，为了演示这一点，我们可以通过故意错误配置主题设置来导致生产者失败。

首先，让我们将[kafka-clients](https://mvnrepository.com/artifact/org.apache.kafka/kafka-clients)依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>3.8.0</version>
</dependency>
```

现在，我们需要模拟[Kafka](https://www.baeldung.com/apache-kafka)代理拒绝生产者发送的消息的用例。例如，我们可以使用“min.insync.replicas”主题配置，该配置在写入被视为成功之前验证最小数量的副本是否同步。

让我们创建一个主题并将此属性设置为2，即使我们的测试环境仅包含一个Kafka代理。因此，新消息始终被拒绝，这使我们能够测试生产者的重试机制：

```java
@Test
void givenDefaultConfig_whenMessageCannotBeSent_thenKafkaProducerRetries() throws Exception {
    NewTopic newTopic = new NewTopic("test-topic-1", 1, (short) 1)
            .configs(Map.of("min.insync.replicas", "2"));
    adminClient.createTopics(singleton(newTopic)).all().get();

    // publish message and verify exception
}
```

然后，我们创建一个KafkaProducer，向该主题发送一条消息，并验证它多次重试并最终在2分钟后超时：

```java
@Test
void givenDefaultConfig_whenMessageCannotBeSent_thenKafkaProducerRetries() throws Exception {
    // set topic config

    Properties props = new Properties();
    props.put(BOOTSTRAP_SERVERS_CONFIG, KAFKA_CONTAINER.getBootstrapServers());
    props.put(KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
    props.put(VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
    KafkaProducer<String, String> producer = new KafkaProducer<>(props);

    ProducerRecord<String, String> record = new ProducerRecord<>("test-topic-1", "test-value");
    assertThatThrownBy(() -> producer.send(record).get())
            .isInstanceOf(ExecutionException.class)
            .hasCauseInstanceOf(org.apache.kafka.common.errors.TimeoutException.class)
            .hasMessageContaining("Expiring 1 record(s) for test-topic-1-0");
}
```

**从异常和日志中我们可以看出，生产者尝试发送消息多次，最终在2分钟后超时**，此行为与KafkaProducer的[默认设置](https://docs.confluent.io/platform/current/installation/configuration/producer-configs.html)一致：

- retries(默认为Integer.MAX_VALUE)：发布消息的最大尝试次数
- delivery.timeout.ms(默认为120000)：在认为消息失败之前等待消息被确认的最长时间
- retry.backoff.ms(默认为100)：重试前等待的时间
- retry.backoff.max.ms(默认为1000)：连续重试之间的最大延迟

## 3. 自定义重试配置

**不用说，我们可以调整KafkaProducer的重试配置以更好地满足我们的需求**。

例如，我们可以将最大传送时间设置为5秒，重试之间使用500毫秒的延迟，并将最大重试次数降低到20次：

```java
@Test
void givenCustomConfig_whenMessageCannotBeSent_thenKafkaProducerRetries() throws Exception {
    // set topic config

    Properties props = new Properties();
    // other properties
    props.put(RETRIES_CONFIG, 20);
    props.put(RETRY_BACKOFF_MS_CONFIG, "500");
    props.put(DELIVERY_TIMEOUT_MS_CONFIG, "5000");
    KafkaProducer<String, String> producer = new KafkaProducer<>(props);

    ProducerRecord<String, String> record = new ProducerRecord<>("test-topic-2", "test-value");
    assertThatThrownBy(() -> producer.send(record).get())
            .isInstanceOf(ExecutionException.class)
            .hasCauseInstanceOf(org.apache.kafka.common.errors.TimeoutException.class)
            .hasMessageContaining("Expiring 1 record(s) for test-topic-2-0");
}
```

正如预期的那样，生产者在自定义的5秒超时后停止重试。日志显示重试间隔500毫秒，并确认重试计数从20开始，每次尝试后都会减少：

```text
12:57:19.599 [kafka-producer-network-thread | producer-1] WARN  o.a.k.c.producer.internals.Sender - [Producer clientId=producer-1] Got error produce response with correlation id 5 on topic-partition test-topic-2-0, retrying (19 attempts left). Error: NOT_ENOUGH_REPLICAS

12:57:20.107 [kafka-producer-network-thread | producer-1] WARN  o.a.k.c.producer.internals.Sender - [Producer clientId=producer-1] Got error produce response with correlation id 6 on topic-partition test-topic-2-0, retrying (18 attempts left). Error: NOT_ENOUGH_REPLICAS

12:57:20.612 [kafka-producer-network-thread | producer-1] WARN  o.a.k.c.producer.internals.Sender - [Producer clientId=producer-1] Got error produce response with correlation id 7 on topic-partition test-topic-2-0, retrying (17 attempts left). Error: NOT_ENOUGH_REPLICAS

[...]
```

## 4. 总结

在本简短教程中，我们探索了KafkaProducer的重试配置。我们学习了如何设置最大传送时间、指定重试次数以及配置失败尝试之间的延迟。