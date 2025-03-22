---
layout: post
title:  将数据发送到Kafka中的特定分区
category: spring-boot
copyright: spring-boot
excerpt: Spring Boot
---

## 1. 简介

[Apache Kafka](https://www.baeldung.com/apache-kafka)是一个分布式流平台，擅长处理海量实时数据流。Kafka将数据组织为[主题](https://www.baeldung.com/kafka-topics-partitions)，并进一步将主题划分为分区。**每个分区充当独立通道，实现并行处理和容错**。

在本教程中，我们深入研究将数据发送到Kafka中特定分区的技术。我们将探讨与此方法相关的好处、实施方法和潜在挑战。

## 2. 了解Kafka分区

现在，我们来探讨一下Kafka分区的基本概念。

### 2.1 什么是Kafka分区

当生产者向Kafka主题发送消息时，Kafka使用指定的分区策略将这些消息组织到分区中。分区是表示线性有序消息序列的基本单位，一旦生成消息，就会根据所选的分区策略将其分配给特定分区。**随后，该消息将附加到该分区内日志的末尾**。

### 2.2 并行性和消费者组

一个Kafka主题可以分为多个分区，并且可以为[消费者组](https://www.baeldung.com/kafka-manage-consumer-groups)分配这些分区的子集，组内的每个消费者都独立处理其分配的分区中的消息。这种并行处理机制增强了整体吞吐量和可扩展性，使Kafka能够高效处理大量数据。

### 2.3 顺序和处理保证

**在单个分区内，Kafka可确保按接收顺序处理消息**，这可确保依赖[消息顺序](https://www.baeldung.com/kafka-message-ordering#1-producer-and-consumer-timing)的应用程序(如金融交易或事件日志)按顺序处理。但请注意，由于网络延迟和其他操作考虑因素，消息的接收顺序可能与最初发送的顺序不同。

**在不同的分区上，Kafka不强制保证顺序**。来自不同分区的消息可能会同时处理，从而导致事件顺序可能发生变化。在设计依赖于严格消息顺序的应用程序时，必须考虑这一特性。

### 2.4 容错和高可用性

分区也为Kafka提供了出色的容错能力，每个分区都可以在多个代理之间复制。如果代理发生故障，复制的分区仍然可以访问，并确保对数据的持续访问。

Kafka集群可以无缝地将消费者重定向到健康的代理，从而保持数据可用性和高系统可靠性。

## 3. 为什么要发送数据到特定分区

在本节中，我们将探讨向特定分区发送数据的原因。

### 3.1 数据亲和力

**数据亲和性是指将相关数据有意地分组到同一个分区中**。通过将相关数据发送到特定的分区，我们可以确保这些数据一起处理，从而提高处理效率。

例如，考虑这样一种情况：我们可能希望确保客户的订单位于同一分区中，以便进行订单跟踪和分析，保证特定客户的所有订单最终位于同一分区可简化跟踪和分析流程。

### 3.2 负载均衡

此外，在分区之间均匀分布数据有助于**确保最佳资源利用率**。跨分区均匀分布数据有助于优化Kafka集群内的资源利用率，通过根据负载考虑将数据发送到分区，我们可以防止资源瓶颈并确保每个分区接收到可管理且均衡的工作负载。

### 3.3 优先顺序

在某些情况下，并非所有数据都具有相同的优先级或紧迫性。Kafka的分区功能可将关键数据定向到专用分区以加快处理，从而实现关键数据的优先级排序，这种优先处理可确保高优先级消息比不太重要的消息得到及时关注和更快的处理。

## 4. 发送到特定分区的方法

Kafka提供了各种将消息分配给分区的策略，提供数据分布和处理的灵活性。以下是一些可用于将消息发送到特定分区的常用方法。

### 4.1 粘性分区

**在Kafka 2.4及更高版本中，粘性分区器旨在将没有key的消息保留在同一分区中**。但是，此行为并不是绝对的，并且会与batch.size和linger.ms等批处理设置交互。

为了优化消息传递，Kafka在将消息发送到代理之前会将消息分组为批次。batch.size设置(默认16384字节)控制最大批次大小，影响消息在粘性分区器下停留在同一分区的时间。

linger.ms配置(默认值0毫秒)在发送批次之前引入延迟，可能会延长没有key的消息的粘性行为。

在以下测试用例中，假设默认批处理配置保持不变。我们将发送三条消息，但不明确分配key，我们应该期望它们最初被分配给同一个分区：

```java
kafkaProducer.send("default-topic", "message1");
kafkaProducer.send("default-topic", "message2");
kafkaProducer.send("default-topic", "message3");

await().atMost(2, SECONDS)
    .until(() -> kafkaMessageConsumer.getReceivedMessages()
        .size() >= 3);

List<ReceivedMessage> records = kafkaMessageConsumer.getReceivedMessages();

Set<Integer> uniquePartitions = records.stream()
    .map(ReceivedMessage::getPartition)
    .collect(Collectors.toSet());

Assert.assertEquals(1, uniquePartitions.size());
```

### 4.2 基于Key的方法

在基于Key的方法中，**Kafka将具有相同Key的消息定向到同一分区，从而优化相关数据的处理**。这是通过哈希函数实现的，确保消息key到分区的确定性映射。

在此测试用例中，具有相同key partitionA的消息应始终位于同一分区中，让我们用以下代码片段来说明基于key的分区：

```java
kafkaProducer.send("order-topic", "partitionA", "critical data");
kafkaProducer.send("order-topic", "partitionA", "more critical data");
kafkaProducer.send("order-topic", "partitionB", "another critical message");
kafkaProducer.send("order-topic", "partitionA", "another more critical data");

await().atMost(2, SECONDS)
    .until(() -> kafkaMessageConsumer.getReceivedMessages()
        .size() >= 4);

List<ReceivedMessage> records = kafkaMessageConsumer.getReceivedMessages();
Map<String, List<ReceivedMessage>> messagesByKey = groupMessagesByKey(records);

messagesByKey.forEach((key, messages) -> {
    int expectedPartition = messages.get(0)
        .getPartition();
    for (ReceivedMessage message : messages) {
        assertEquals("Messages with key '" + key + "' should be in the same partition", message.getPartition(), expectedPartition);
    }
});
```

此外，通过基于key的方法，共享相同key的消息将按照它们在特定分区中生成的顺序一致地接收，**这保证了分区内消息顺序的保存，尤其是相关消息**。

在此测试用例中，我们按照特定顺序生成带有key partitionA的消息，并且测试主动验证这些消息在分区内是否按照相同的顺序接收：

```java
kafkaProducer.send("order-topic", "partitionA", "message1");
kafkaProducer.send("order-topic", "partitionA", "message3");
kafkaProducer.send("order-topic", "partitionA", "message4");

await().atMost(2, SECONDS)
    .until(() -> kafkaMessageConsumer.getReceivedMessages()
        .size() >= 3);

List<ReceivedMessage> records = kafkaMessageConsumer.getReceivedMessages();

StringBuilder resultMessage = new StringBuilder();
records.forEach(record -> resultMessage.append(record.getMessage()));
String expectedMessage = "message1message3message4";

assertEquals("Messages with the same key should be received in the order they were produced within a partition", expectedMessage, resultMessage.toString());
```

### 4.3 自定义分区

为了进行细粒度控制，Kafka允许定义自定义分区器。这些类实现了Partitioner接口，使我们能够根据消息内容、元数据或其他因素编写逻辑来确定目标分区。

在本节中，我们将根据客户类型创建自定义分区逻辑，以便在将订单发送到Kafka主题时进行处理。具体来说，高级客户订单将被定向到一个分区，而普通客户订单将被定向到另一个分区。

首先，**我们创建一个名为CustomPartitioner的类，继承自Kafka Partitioner接口**。在此类中，我们使用自定义逻辑重写partition()方法来确定每条消息的目标分区：

```java
public class CustomPartitioner implements Partitioner {
    private static final int PREMIUM_PARTITION = 0;
    private static final int NORMAL_PARTITION = 1;

    @Override
    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
        String customerType = extractCustomerType(key.toString());
        return "premium".equalsIgnoreCase(customerType) ? PREMIUM_PARTITION : NORMAL_PARTITION;
    }

    private String extractCustomerType(String key) {
        String[] parts = key.split("_");
        return parts.length > 1 ? parts[1] : "normal";
    }

    // more methods
}
```

接下来，要在Kafka中应用此自定义分区器，**我们需要在生产者配置中设置PARTITIONER_CLASS_CONFIG属性**。Kafka将使用此分区器根据CustomPartitioner类中定义的逻辑确定每条消息的分区。

方法setProducerToUseCustomPartitioner()用于设置Kafka生产者使用CustomPartitioner：

```java
private KafkaTemplate<String, String> setProducerToUseCustomPartitioner() {
    Map<String, Object> producerProps = KafkaTestUtils.producerProps(embeddedKafkaBroker.getBrokersAsString());
    producerProps.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
    producerProps.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
    producerProps.put(ProducerConfig.PARTITIONER_CLASS_CONFIG, CustomPartitioner.class.getName());
    DefaultKafkaProducerFactory<String, String> producerFactory = new DefaultKafkaProducerFactory<>(producerProps);

    return new KafkaTemplate<>(producerFactory);
}
```

然后，我们构建一个测试用例，以确保自定义分区逻辑正确地将高级和普通客户订单路由到各自的分区：

```java
KafkaTemplate<String, String> kafkaTemplate = setProducerToUseCustomPartitioner();

kafkaTemplate.send("order-topic", "123_premium", "Order 123, Premium order message");
kafkaTemplate.send("order-topic", "456_normal", "Normal order message");

await().atMost(2, SECONDS)
    .until(() -> kafkaMessageConsumer.getReceivedMessages()
        .size() >= 2);

consumer.assign(Collections.singletonList(new TopicPartition("order-topic", 0)));
ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));

for (ConsumerRecord<String, String> record : records) {
    assertEquals("Premium order message should be in partition 0", 0, record.partition());
    assertEquals("123_premium", record.key());
}
```

### 4.4 直接分区分配

当在主题之间手动迁移数据或调整跨分区的数据分布时，直接分区分配可以帮助控制消息放置。Kafka还提供了使用接收分区号的ProductRecord构造函数将消息直接发送到特定分区的功能。通过指定分区号，**我们可以明确指定每条消息的目标分区**。

在此测试用例中，我们在send()方法中指定第二个参数来接收分区号：

```java
kafkaProducer.send("order-topic", 0, "123_premium", "Premium order message");
kafkaProducer.send("order-topic", 1, "456_normal", "Normal order message");

await().atMost(2, SECONDS)
    .until(() -> kafkaMessageConsumer.getReceivedMessages()
        .size() >= 2);

List<ReceivedMessage> records = kafkaMessageConsumer.getReceivedMessages();

for (ReceivedMessage record : records) {
    if ("123_premium".equals(record.getKey())) {
        assertEquals("Premium order message should be in partition 0", 0, record.getPartition());
    } else if ("456_normal".equals(record.getKey())) {
        assertEquals("Normal order message should be in partition 1", 1, record.getPartition());
    }
}
```

## 5. 从特定分区消费

要在消费者端消费Kafka中特定分区的数据，**我们可以使用KafkaConsumer.assign()方法指定要订阅的分区**。这可以对消费进行细粒度的控制，但需要手动管理分区偏移量。

以下是使用allocate()方法从特定分区消费消息的示例：

```java
KafkaTemplate<String, String> kafkaTemplate = setProducerToUseCustomPartitioner();

kafkaTemplate.send("order-topic", "123_premium", "Order 123, Premium order message");
kafkaTemplate.send("order-topic", "456_normal", "Normal order message");

await().atMost(2, SECONDS)
    .until(() -> kafkaMessageConsumer.getReceivedMessages()
        .size() >= 2);

consumer.assign(Collections.singletonList(new TopicPartition("order-topic", 0)));
ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));

for (ConsumerRecord<String, String> record : records) {
    assertEquals("Premium order message should be in partition 0", 0, record.partition());
    assertEquals("123_premium", record.key());
}
```

## 6. 潜在的挑战和考虑因素

**将消息发送到特定分区时，存在分区之间负载分布不均匀的风险**。如果用于分区的逻辑未在所有分区之间均匀分布消息，则可能会发生这种情况。此外，扩展Kafka集群(涉及添加或删除代理)可能会触发分区重新分配。在重新分配期间，代理可能会移动分区，这可能会扰乱消息的顺序或导致暂时不可用。

因此，**我们应该使用Kafka工具或指标定期监控每个分区上的负载**。例如，[Kafka管理客户端和Micrometer](https://www.baeldung.com/java-kafka-consumer-lag)可以帮助深入了解分区运行状况和性能。我们可以使用管理客户端来检索有关主题、分区及其当前状态的信息；并使用Micrometer进行指标监控。

此外，预计需要主动调整分区策略或水平扩展Kafka集群，以有效管理特定分区上增加的负载。我们还可以考虑增加分区数量或调整key范围以实现更均匀的分布。

## 7. 总结

总之，向Apache Kafka中的特定分区发送消息的能力为优化数据处理和提高整体系统效率提供了强大的可能性。

在本教程中，我们探索了将消息定向到特定分区的各种方法，包括基于key的方法、自定义分区和直接分区分配。每种方法都具有独特的优势，使我们能够根据应用的具体要求进行自定义。