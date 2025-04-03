---
layout: post
title:  确保Kafka中的消息顺序：策略和配置
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 概述

在本文中，我们将探讨[Apache Kafka](https://www.baeldung.com/apache-kafka)中有关消息顺序的挑战和解决方案。以正确的顺序处理消息对于维护分布式系统中的数据完整性和一致性至关重要，虽然Kafka提供了维护消息顺序的机制，但在分布式环境中实现这一点本身就很复杂。

## 2. 分区内顺序及其挑战

Kafka通过为每条消息分配唯一的偏移量来维护单个[分区](https://www.baeldung.com/kafka-topics-partitions)内的顺序，这保证了该分区内消息的顺序追加。但是，当我们扩展并使用多个分区时，维护全局顺序变得很复杂。不同的分区以不同的速率接收消息，这使得它们之间的严格顺序变得复杂。

### 2.1 生产者和消费者的时机

让我们来谈谈Kafka如何处理消息顺序，生产者发送消息的顺序和消费者接收消息的顺序略有不同。通过坚持使用一个分区，我们按照消息到达代理的顺序处理消息。但是，此顺序可能与我们最初发送消息的顺序不匹配。这种混淆可能是由于网络延迟或我们重新发送消息等原因造成的。为了保持一致，我们可以实现带有确认和重试的生产者。这样，我们就可以确保消息不仅能够到达Kafka，而且能够以正确的顺序到达。

### 2.2 多个分区带来的挑战

这种跨分区分布虽然有利于提高可扩展性和容错能力，但也带来了实现全局消息顺序的复杂性。例如，我们按顺序发送两条消息M1和M2，Kafka会像我们发送它们一样接收它们，但会将它们放在不同的分区中。问题在于，M1先发送并不意味着它会在M2之前得到处理。在处理顺序至关重要的场景中(例如金融交易)，这可能具有挑战性。

### 2.3 单分区消息顺序

我们创建名为'single_partition_topic'的[主题](https://www.baeldung.com/kafka-topics-partitions)，该主题有一个分区，以及名为'multi_partition_topic'的主题，该主题有5个分区。下面是一个具有单个分区的主题示例，其中生产者正在向该主题发送消息：

```java
Properties producerProperties = new Properties();
producerProperties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, KAFKA_CONTAINER.getBootstrapServers());
producerProperties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, LongSerializer.class.getName());
producerProperties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JacksonSerializer.class.getName());
producer = new KafkaProducer<>(producerProperties);
for (long sequenceNumber = 1; sequenceNumber <= 10; sequenceNumber++) {
    UserEvent userEvent = new UserEvent(UUID.randomUUID().toString());
    userEvent.setGlobalSequenceNumber(sequenceNumber);
    userEvent.setEventNanoTime(System.nanoTime());
    ProducerRecord<Long, UserEvent> producerRecord = new ProducerRecord<>(Config.SINGLE_PARTITION_TOPIC, userEvent);
    Future<RecordMetadata> future = producer.send(producerRecord);
    sentUserEventList.add(userEvent);
    RecordMetadata metadata = future.get();
    logger.info("User Event ID: " + userEvent.getUserEventId() + ", Partition : " + metadata.partition());
}
```

UserEvent是一个实现了Comparable接口的POJO类，有助于按globalSequenceNumber(外部序列号)对消息类进行排序。由于生产者发送的是POJO消息对象，因此我们实现了自定义的Jackson [Serializer](https://www.baeldung.com/jackson-custom-serialization)和[Deserializer](https://www.baeldung.com/jackson-deserialization)。

分区0接收所有用户事件，事件ID按以下顺序出现：

```text
841e593a-bca0-4dd7-9f32-35792ffc522e
9ef7b0c0-6272-4f9a-940d-37ef93c59646
0b09faef-2939-46f9-9c0a-637192e242c5
4158457a-73cc-4e65-957a-bf3f647d490a
fcf531b7-c427-4e80-90fd-b0a10bc096ca
23ed595c-2477-4475-a4f4-62f6cbb05c41
3a36fb33-0850-424c-81b1-dafe4dc3bb18
10bca2be-3f0e-40ef-bafc-eaf055b4ee26
d48dcd66-799c-4645-a977-fa68414ca7c9
7a70bfde-f659-4c34-ba75-9b43a5045d39
```

在Kafka中，每个消费者组都是一个独立的实体。如果两个消费者属于不同的消费者组，他们都会收到该主题上的所有消息。这是因为**Kafka将每个消费者组视为单独的订阅者**。

如果两个消费者属于同一个消费者组，并订阅了具有多个分区的主题，**则Kafka将确保每个消费者从唯一的一组分区中读取**，这是为了允许并发处理消息。

Kafka确保在一个消费者组内，没有两个消费者读取相同的消息，因此每个组的每个消息只处理一次。

以下代码适用于消费者消费来自同一主题的消息：

```java
Properties consumerProperties = new Properties();
consumerProperties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, KAFKA_CONTAINER.getBootstrapServers());
consumerProperties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, LongDeserializer.class.getName());
consumerProperties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, JacksonDeserializer.class.getName());
consumerProperties.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
consumerProperties.put(Config.CONSUMER_VALUE_DESERIALIZER_SERIALIZED_CLASS, UserEvent.class);
consumerProperties.put(ConsumerConfig.GROUP_ID_CONFIG, "test-group");
consumer = new KafkaConsumer<>(consumerProperties);
consumer.subscribe(Collections.singletonList(Config.SINGLE_PARTITION_TOPIC));
ConsumerRecords<Long, UserEvent> records = consumer.poll(TIMEOUT_WAIT_FOR_MESSAGES);
records.forEach(record -> {
    UserEvent userEvent = record.value();
    receivedUserEventList.add(userEvent);
    logger.info("User Event ID: " + userEvent.getUserEventId());
});
```

在这种情况下，我们得到的输出显示消费者以相同的顺序消费消息，以下是输出中的顺序事件ID：

```text
841e593a-bca0-4dd7-9f32-35792ffc522e
9ef7b0c0-6272-4f9a-940d-37ef93c59646
0b09faef-2939-46f9-9c0a-637192e242c5
4158457a-73cc-4e65-957a-bf3f647d490a
fcf531b7-c427-4e80-90fd-b0a10bc096ca
23ed595c-2477-4475-a4f4-62f6cbb05c41
3a36fb33-0850-424c-81b1-dafe4dc3bb18
10bca2be-3f0e-40ef-bafc-eaf055b4ee26
d48dcd66-799c-4645-a977-fa68414ca7c9
7a70bfde-f659-4c34-ba75-9b43a5045d39
```

### 2.4 多分区消息顺序

对于具有多个分区的主题，消费者和生产者的配置相同。唯一的区别是消息发送到的主题和分区，生产者将消息发送到主题“multi_partition_topic”：

```java
Future<RecordMetadata> future = producer.send(new ProducerRecord<>(Config.MULTI_PARTITION_TOPIC, sequenceNumber, userEvent));
sentUserEventList.add(userEvent);
RecordMetadata metadata = future.get();
logger.info("User Event ID: " + userEvent.getUserEventId() + ", Partition : " + metadata.partition());
```

消费者消费来自同一主题的消息：

```java
consumer.subscribe(Collections.singletonList(Config.MULTI_PARTITION_TOPIC));
ConsumerRecords<Long, UserEvent> records = consumer.poll(TIMEOUT_WAIT_FOR_MESSAGES);
records.forEach(record -> {
    UserEvent userEvent = record.value();
    receivedUserEventList.add(userEvent);
    logger.info("User Event ID: " + userEvent.getUserEventId());
});
```

生产者输出列出了事件ID及其各自的分区，如下所示：

```text
939c1760-140e-4d0c-baa6-3b1dd9833a7d, 0
47fdbe4b-e8c9-4b30-8efd-b9e97294bb74, 4
4566a4ec-cae9-4991-a8a2-d7c5a1b3864f, 4
4b061609-aae8-415f-94d7-ae20d4ef1ca9, 3
eb830eb9-e5e9-498f-8618-fb5d9fc519e4, 2
9f2a048f-eec1-4c68-bc33-c307eec7cace, 1
c300f25f-c85f-413c-836e-b9dcfbb444c1, 0
c82efad1-6287-42c6-8309-ae1d60e13f5e, 4
461380eb-4dd6-455c-9c92-ae58b0913954, 4
43bbe38a-5c9e-452b-be43-ebb26d58e782, 3
```

对于消费者，输出将显示**消费者未按相同顺序消费消息**，输出中的事件ID如下：

```text
939c1760-140e-4d0c-baa6-3b1dd9833a7d
47fdbe4b-e8c9-4b30-8efd-b9e97294bb74
4566a4ec-cae9-4991-a8a2-d7c5a1b3864f
c82efad1-6287-42c6-8309-ae1d60e13f5e
461380eb-4dd6-455c-9c92-ae58b0913954
eb830eb9-e5e9-498f-8618-fb5d9fc519e4
4b061609-aae8-415f-94d7-ae20d4ef1ca9
43bbe38a-5c9e-452b-be43-ebb26d58e782
c300f25f-c85f-413c-836e-b9dcfbb444c1
9f2a048f-eec1-4c68-bc33-c307eec7cace
```

## 3. 消息排序策略

### 3.1 使用单个分区

我们可以在Kafka中使用单个分区，如我们前面的'single_partition_topic'示例所示，这可以确保消息的顺序。但是，这种方法有其缺点：

- 吞吐量约束：想象一下我们在一家繁忙的披萨店，如果我们只有一名厨师(生产者)和一名服务员(消费者)在一张桌子(分区)上工作，那么他们只能提供一定数量的披萨，否则就会开始积压。在Kafka的世界里，当我们处理大量消息时，坚持使用单个分区就像那种单表场景。单个分区在高容量场景中会成为瓶颈，并且消息处理速度会受到限制，因为一次只有一个生产者和一个消费者可以在单个分区上操作。
- 并行性降低：在上述示例中，如果多个厨师(生产者)和服务员(消费者)在多个表(分区)上工作，则完成的订单数量会增加。Kafka的优势在于跨多个分区进行并行处理。如果只有一个分区，则此优势就会消失，导致顺序处理并进一步限制消息流。

本质上，**单个分区虽然保证了顺序，但却是以降低吞吐量为代价的**。

### 3.2 带时间窗口缓冲的外部顺序

在这种方法中，生产者会为每条消息标记一个全局序列号。多个消费者实例会同时从不同的分区消费消息，并使用这些序列号对消息进行重新顺序，从而确保全局顺序。

在具有多个生产者的实际场景中，**我们将通过可在所有生产者进程中访问的共享资源(例如数据库序列或分布式计数器)来管理全局序列**，这可确保序列号在所有消息中都是唯一的且有序的，无论哪个生产者发送它们：

```java
for (long sequenceNumber = 1; sequenceNumber <= 10 ; sequenceNumber++) {
    UserEvent userEvent = new UserEvent(UUID.randomUUID().toString());
    userEvent.setEventNanoTime(System.nanoTime());
    userEvent.setGlobalSequenceNumber(sequenceNumber);
    Future<RecordMetadata> future = producer.send(new ProducerRecord<>(Config.MULTI_PARTITION_TOPIC, sequenceNumber, userEvent));
    sentUserEventList.add(userEvent);
    RecordMetadata metadata = future.get();
    logger.info("User Event ID: " + userEvent.getUserEventId() + ", Partition : " + metadata.partition());
}
```

在消费者方面，我们将消息分组到时间窗口中，然后按顺序处理它们。在特定时间范围内到达的消息，我们会将它们分批处理，一旦窗口过去，我们就会处理该批消息。这可确保该时间范围内的消息按顺序处理，即使它们在窗口内的不同时间到达。消费者会缓冲消息，并在处理之前根据序列号对它们重新顺序。我们需要确保以正确的顺序处理消息，为此，消费者应该有一个缓冲期，在处理缓冲消息之前，它会多次轮询消息，并且这个缓冲期足够长，可以应对潜在的消息顺序问题：

```java
consumer.subscribe(Collections.singletonList(Config.MULTI_PARTITION_TOPIC));
List<UserEvent> buffer = new ArrayList<>();
long lastProcessedTime = System.nanoTime();
ConsumerRecords<Long, UserEvent> records = consumer.poll(TIMEOUT_WAIT_FOR_MESSAGES);
records.forEach(record -> {
    buffer.add(record.value());
});
while (!buffer.isEmpty()) {
    if (System.nanoTime() - lastProcessedTime > BUFFER_PERIOD_NS) {
        processBuffer(buffer, receivedUserEventList);
        lastProcessedTime = System.nanoTime();
    }
    records = consumer.poll(TIMEOUT_WAIT_FOR_MESSAGES);
    records.forEach(record -> {
        buffer.add(record.value());
    });
}

void processBuffer(List buffer, List receivedUserEventList) {
    Collections.sort(buffer);
    buffer.forEach(userEvent -> {
        receivedUserEventList.add(userEvent);
        logger.info("Processing message with Global Sequence number: " + userEvent.getGlobalSequenceNumber() + ", User Event Id: "  + userEvent.getUserEventId());
    });
    buffer.clear();
}
```

每个事件ID都会与其对应的分区一起出现在输出中，如下所示：

```text
d6ef910f-2e65-410d-8b86-fa0fc69f2333, 0
4d6bfe60-7aad-4d1b-a536-cc735f649e1a, 4
9b68dcfe-a6c8-4cca-874d-cfdda6a93a8f, 4
84bd88f5-9609-4342-a7e5-d124679fa55a, 3
55c00440-84e0-4234-b8df-d474536e9357, 2
8fee6cac-7b8f-4da0-a317-ad38cc531a68, 1
d04c1268-25c1-41c8-9690-fec56397225d, 0
11ba8121-5809-4abf-9d9c-aa180330ac27, 4
8e00173c-b8e1-4cf7-ae8c-8a9e28cfa6b2, 4
e1acd392-db07-4325-8966-0f7c7a48e3d3, 3
```

具有全局序列号和事件ID的消费者输出：

```text
1, d6ef910f-2e65-410d-8b86-fa0fc69f2333
2, 4d6bfe60-7aad-4d1b-a536-cc735f649e1a
3, 9b68dcfe-a6c8-4cca-874d-cfdda6a93a8f
4, 84bd88f5-9609-4342-a7e5-d124679fa55a
5, 55c00440-84e0-4234-b8df-d474536e9357
6, 8fee6cac-7b8f-4da0-a317-ad38cc531a68
7, d04c1268-25c1-41c8-9690-fec56397225d
8, 11ba8121-5809-4abf-9d9c-aa180330ac27
9, 8e00173c-b8e1-4cf7-ae8c-8a9e28cfa6b2
10, e1acd392-db07-4325-8966-0f7c7a48e3d3

```

### 3.3 带缓冲的外部顺序注意事项

在此方法中，每个消费者实例都会缓冲消息并根据其序列号按顺序处理它们。但是，有几个注意事项：

- 缓冲区大小：缓冲区的大小可能会根据传入消息的数量而增加。在按序列号严格排序的实现中，我们可能会看到缓冲区显著增长，尤其是在消息传递出现延迟的情况下。例如，如果我们每分钟处理100条消息，但由于延迟而突然收到200条消息，则缓冲区将意外增长。因此，我们必须有效管理缓冲区大小，并准备好策略以防超出预期限制。
- 延迟：当我们缓冲消息时，我们实际上是让它们在处理之前等待一段时间(引入延迟)。一方面，它帮助我们保持有序；另一方面，它减慢了整个过程。这一切都是为了在保持秩序和最小化延迟之间找到适当的平衡。
- 失败：如果消费者失败，我们可能会丢失缓冲的消息，为了防止这种情况，我们可能需要定期保存缓冲区的状态。
- 延迟消息：在窗口处理后到达的消息将乱序，根据用例，我们可能需要策略来处理或丢弃此类消息。
- 状态管理：如果处理涉及有状态的操作，我们将需要机制来管理和跨窗口保持状态。
- 资源利用率：在缓冲区中保存大量消息需要内存，我们需要确保有足够的资源来处理这个问题，尤其是当消息在缓冲区中停留的时间较长时。

### 3.4 幂等生产者

Kafka的幂等生产者功能旨在精确地传递一次消息，从而防止任何重复，这在生产者可能由于网络错误或其他瞬时故障而重试发送消息的情况下至关重要。虽然幂等的主要目标是防止消息重复，但它间接影响消息顺序。Kafka使用两个东西来实现幂等：生产者ID(PID)和序列号，后者充当幂等键，并且在特定分区的上下文中是唯一的。

- 序列号：Kafka为生产者发送的每条消息分配序列号，这些序列号在每个分区中都是唯一的，确保生产者按特定顺序发送的消息在被Kafka接收时，会以相同的顺序写入特定分区中。序列号保证单个分区内的顺序，但是，当向多个分区生成消息时，无法保证分区之间的全局顺序。例如，如果生产者分别向分区P1、P2和P3发送消息M1、M2和M3，则每条消息在其分区内都会收到一个唯一的序列号。但是，它不能保证这些分区之间的相对消费顺序。
- 生产者ID(PID)：启用幂等性时，代理会为每个生产者分配一个唯一的生产者ID(PID)，此PID与序列号相结合，使Kafka能够识别并丢弃生产者重试产生的任何重复消息。

**Kafka按照消息生成的顺序将消息写入分区，从而保证消息的顺序，这得益于序列号，并使用PID和幂等特性防止重复**。要启用幂等生产者，我们需要在生产者的配置中将“enable.idempotence”属性设置为true：

```java
props.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, "true");
```

## 4. 生产者和消费者的关键配置

Kafka生产者和消费者有一些关键配置，它们可以影响消息顺序和吞吐量。

### 4.1 生产者配置

- MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION：如果我们要发送一堆消息，那么Kafka中的此设置有助于确定我们可以发送多少条消息而无需等待“已读”回执。如果我们将其设置为高于1而不启用幂等性，那么如果我们必须重新发送消息，我们最终可能会打乱消息的顺序。但是，如果我们启用幂等性，即使我们一次发送一堆消息，Kafka也会保持消息的顺序。对于非常严格的顺序，例如确保在发送下一条消息之前先读取每条消息，我们应该将此值设置为1。如果我们想要优先考虑速度而不是完美的顺序，那么我们可以将其设置为5，但这可能会引发顺序问题。
- BATCH_SIZE_CONFIG和LINGER_MS_CONFIG：Kafka控制默认批处理大小(以字节为单位)，旨在将同一分区的记录分组为更少的请求，以获得更好的性能。如果我们将此限制设置得太低，我们将发送许多小组，这会减慢我们的速度。但是，如果我们将其设置得太高，这可能不是内存的最佳利用方式。如果组尚未填满，Kafka可以等待一段时间再发送，此等待时间由LINGER_MS_CONFIG控制。如果更多消息足够快地进入以填满我们设置的限制，它们会立即发送，但如果没有，Kafka不会继续等待-它会在时间到了时发送我们拥有的任何消息。这就像平衡速度和效率，确保我们一次发送足够的消息而不会出现不必要的延迟。

```java
props.put(ProducerConfig.MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION, "1");
props.put(ProducerConfig.BATCH_SIZE_CONFIG, "16384");
props.put(ProducerConfig.LINGER_MS_CONFIG, "5");
```

### 4.2 消费者配置

- MAX_POLL_RECORDS_CONFIG：这是Kafka消费者每次请求数据时抓取的记录数的限制，如果我们将此数字设置得很高，我们可以一次处理大量数据，从而提高吞吐量。但有一个问题-我们获取的数据越多，保持一切井然有序就越困难。因此，我们需要找到一个既高效又不至于不堪重负的最佳点。
- FETCH_MIN_BYTES_CONFIG：如果我们将此数字设置为较高，Kafka会等到它有足够的数据来满足我们的最小字节数后再发送。这意味着更少的行程(或获取)，这对效率很有好处。但如果我们很着急，想要快速获取数据，我们可以将此数字设置得较低，这样Kafka就可以更快地向我们发送它拥有的任何数据。例如，如果我们的消费者应用程序是资源密集型或需要保持严格的消息顺序，尤其是在多线程的情况下，较小的批次可能会有益。
- FETCH_MAX_WAIT_MS_CONFIG：这将决定我们的消费者等待Kafka收集足够的数据以满足我们的FETCH_MIN_BYTES_CONFIG的时间。如果我们将此时间设置得较高，我们的消费者愿意等待更长时间，可能一次获得更多数据。但如果我们很着急，我们会将其设置得较低，这样我们的消费者就可以更快地获得数据，即使数据量没有那么多。这是等待更大运输量和快速移动之间的平衡行为。

```java
props.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, "500");
props.put(ConsumerConfig.FETCH_MIN_BYTES_CONFIG, "1");
props.put(ConsumerConfig.FETCH_MAX_WAIT_MS_CONFIG, "500");
```

## 5. 总结

在本文中，我们深入探讨了Kafka中消息顺序的复杂性。我们探讨了挑战并提出了应对策略，无论是通过单个分区、使用时间窗口缓冲的外部顺序还是幂等生产者，Kafka都提供了定制解决方案来满足消息顺序的需求。