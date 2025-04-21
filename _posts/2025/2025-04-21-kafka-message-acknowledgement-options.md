---
layout: post
title:  Kafka生产者和消费者消息确认选项
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 简介

众所周知，[Apache Kafka](https://www.baeldung.com/apache-kafka)是一个消息传递和流式传输系统，它提供了确认选项以确保可靠性保证。在本教程中，我们将介绍Apache Kafka中[生产者和消费者](https://www.baeldung.com/apache-kafka#1-producers-amp-consumers)的确认选项。

## 2. 生产者确认选项

即使Kafka Broker的配置可靠，我们也必须将生产者配置得相对合理，我们可以使用三种确认模式之一来配置生产者，下文将详细介绍。

### 2.1 无确认

我们可以将属性acks设置为0：

```java
static KafkaProducer<String, String> producerack0;

static Properties getProducerProperties() {
    Properties producerProperties = new Properties();
    producerProperties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,
            KAFKA_CONTAINER.getBootstrapServers());
    return producerProperties;
}

static void setUp() throws IOException, InterruptedException {
    Properties producerProperties = getProducerProperties();
    producerProperties.put(ProducerConfig.ACKS_CONFIG, "0");
    producerack0 = new KafkaProducer<>(producerProperties);
}
```

在此配置中，**生产者不会等待代理的回复，它假定消息已成功发送**。如果出现问题，代理未收到消息，生产者将无法感知，消息将丢失。

但是，由于生产者不等待服务器的任何响应，它可以以网络支持的最快速度发送消息，从而实现高吞吐量。

如果生产者成功通过网络发送消息，则认为消息已成功写入Kafka。如果发送的对象无法序列化或网卡故障等错误，则客户端发送消息失败。但是，消息到达Broker后，即使[分区](https://www.baeldung.com/apache-kafka#4-clusters-and-partition-replicas)处于离线状态、正在进行Leader选举，甚至整个Kafka集群不可用，我们也不会收到任何错误。

使用acks = 0运行时，生产者延迟较低，尽管它不会改善端到端延迟，因为消费者只有在系统将消息复制到所有可用副本后才会看到消息。

### 2.2 确认领导者

我们可以将属性acks设置为1：

```java
static KafkaProducer<String, String> producerack1;

static Properties getProducerProperties() {
    Properties producerProperties = new Properties();
    producerProperties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,
            KAFKA_CONTAINER.getBootstrapServers());
    return producerProperties;
}

static void setUp() throws IOException, InterruptedException {
    Properties producerack1Prop = getProducerProperties();
    producerack1Prop.put(ProducerConfig.ACKS_CONFIG, "1");
    producerack1 = new KafkaProducer<>(producerack1Prop);
}
```

**当Leader副本收到消息时，生产者会立即收到来自Broker的成功响应**。如果由于任何错误导致消息无法写入Leader，生产者会收到错误响应并重试发送消息，从而避免潜在的数据丢失。

我们仍然可能因为其他原因而丢失消息，例如，在系统将最新消息复制到新Leader之前，Leader崩溃了。

Leader在收到消息并将其写入分区数据文件后，会立即发送确认或错误。如果Leader关闭或崩溃，我们可能会丢失数据，崩溃可能会导致一些已成功写入并确认的消息在崩溃前无法复制到跟随者。

使用acks= 1配置时，写入Leader的速度可能快于其复制消息的速度，在这种情况下，我们最终会得到副本不足的分区，因为Leader在复制消息之前会先确认来自生产者的消息。由于需要等待一个副本收到消息，因此延迟会高于acks = 0配置。

### 2.3 全部确认

我们还可以将属性acks设置为all：

```java
static KafkaProducer<String, String> producerackAll;

static Properties getProducerProperties() {
    Properties producerProperties = new Properties();
    producerProperties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,
            KAFKA_CONTAINER.getBootstrapServers());
    return producerProperties;
}

static void setUp() throws IOException, InterruptedException {
    Properties producerackAllProp = getProducerProperties();
    producerackAllProp.put(ProducerConfig.ACKS_CONFIG,
            "all");
    producerackAll = new KafkaProducer<>(producerackAllProp);
}
```

一**旦所有同步副本都收到该消息，生产者就会收到来自代理的成功响应**。这是最安全的模式，因为我们可以确保多个代理拥有该消息，并且即使在崩溃的情况下消息也能保留下来。

Leader会等待所有同步副本都收到消息后，才会发回确认或错误，Broker上的min.insync.replicas配置允许我们指定生产者确认消息之前必须接收的最小副本数量。

这是最安全的选项，生产者会持续尝试发送消息，直到消息完全提交。此配置下的生产者延迟最高，因为生产者需要等待所有同步副本都收到所有消息，才能将消息批次标记为“完成”并继续处理。

将acks属性设置为值-1相当于将其设置为值all。

**属性acks只能设置为以下三个可能值之一：0、1或all/-1，如果设置为这三个值之外的任何值，Kafka都会抛出ConfigException**。

对于acks配置1及all配置，我们可以使用生产者属性retries、retry.backoff.ms和delivery.timeout.ms来处理[生产者重试](https://www.baeldung.com/kafka-producer-retries)。

## 3. 消费者确认选项

只有在Kafka将数据标记为已提交后，消费者才能访问数据，这确保系统将数据写入所有同步的副本，这保证了消费者收到一致的数据。他们唯一的责任是跟踪已读取的消息和尚未读取的消息，这是在消费消息时不丢失消息的关键。

**当从分区读取数据时，消费者会获取一批消息，检查该批次中的最后一个[偏移量](https://www.baeldung.com/kafka-consumer-offset)，然后从收到的最后一个偏移量开始请求另一批消息**，这保证了Kafka消费者始终以正确的顺序获取新数据，而不会丢失任何消息。

我们有4个消费者配置属性，了解这些属性对于配置消费者以实现所需的可靠性行为非常重要。

### 3.1 组ID

每个Kafka消费者都属于一个组，由属性group.id标识。

假设一个组中有两个消费者，也就是说它们拥有相同的组ID，Kafka系统会将每个消费者分配到主题中分区的一个子集，每个消费者单独读取一部分消息，整个组会读取所有消息。

如果我们需要消费者自己查看其订阅主题中的每一条消息，它需要一个唯一的group.id。

### 3.2 自动偏移量重置

**属性auto.offset.reset确定消费者开始读取没有提交偏移量或无效提交偏移量的分区时的行为**：

```java
static Properties getConsumerProperties() {
    Properties consumerProperties = new Properties();
    consumerProperties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, KAFKA_CONTAINER.getBootstrapServers());
    return consumerProperties;
}

Properties consumerProperties = getConsumerProperties(); 
consumerProperties.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false); 
consumerProperties.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "latest");

try (KafkaConsumer<String, String> consumer = new KafkaConsumer<>(consumerProperties)) {
    // ...
}
```

或者

```java
static Properties getConsumerProperties() {
    Properties consumerProperties = new Properties();
    consumerProperties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, KAFKA_CONTAINER.getBootstrapServers());
    return consumerProperties;
}

Properties consumerProperties = getConsumerProperties(); 
consumerProperties.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false); 
consumerProperties.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");

try (KafkaConsumer<String, String> consumer = new KafkaConsumer<>(consumerProperties)) {
    // ...
}
```

默认值为latest，这意味着，如果缺少有效的偏移量，消费者将从最新的记录开始读取，消费者仅考虑其启动运行后写入的记录。

该属性的另一个值是earthest，这意味着，如果缺少有效的偏移量，消费者将从分区的最前面开始读取所有数据。如果将auto.offset.reset配置设置为none，则从无效偏移量消费时会导致异常。

### 3.3 启用自动提交

属性enable.auto.commit控制消费者是否自动提交偏移量，默认为true：

```java
static Properties getConsumerProperties() {
    Properties consumerProperties = new Properties();
    consumerProperties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, KAFKA_CONTAINER.getBootstrapServers());
    return consumerProperties;
}

Properties consumerProperties = getConsumerProperties(); 
consumerProperties.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, true);

try (KafkaConsumer<String, String> consumer = new KafkaConsumer<>(consumerProperties)) {
    // ...
}
```

如果设置为false，我们可以控制系统何时提交偏移量，这使我们能够最大限度地减少重复并避免数据丢失。

将属性enable.auto.commit设置为true允许我们使用auto.commit.interval.ms控制提交的频率。

### 3.4 自动提交间隔

在自动提交配置中，属性auto.commit.interval.ms配置Kafka系统提交偏移量的频率：

```java
static Properties getConsumerProperties() {
    Properties consumerProperties = new Properties();
    consumerProperties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, KAFKA_CONTAINER.getBootstrapServers());
    return consumerProperties;
}

Properties consumerProperties = getConsumerProperties(); 
consumerProperties.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, 7000); 
consumerProperties.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, true);

try (KafkaConsumer<String, String> consumer = new KafkaConsumer<>(consumerProperties)) {
    // ...
}
```

默认值为每5秒一次，通常，更频繁地提交会增加开销，但可以减少消费者停止时可能发生的重复次数。

## 4. 总结

在本文中，我们了解了Apache Kafka的生产者和消费者确认选项以及如何使用它们。Kafka中的确认选项允许开发人员在性能和可靠性之间进行微调，使其成为适用于各种用例的多功能系统。