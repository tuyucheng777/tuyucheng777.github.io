---
layout: post
title:  使用Apache Kafka读取多条消息
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 概述

在本教程中，我们将探索Kafka Consumer如何从代理检索消息。我们将了解可配置属性，这些属性可直接影响Kafka Consumer一次读取的消息数量。最后，我们将探索调整这些设置如何影响Consumer的行为。

## 2. 设置环境

**Kafka消费者以可配置大小的批次提取给定分区的记录，我们无法配置一次提取的确切记录数，但可以配置这些批次的大小(以字节为单位)**。

对于本文中的代码片段，我们需要一个简单的Spring应用程序，该应用程序使用[kafka-clients](https://mvnrepository.com/artifact/org.apache.kafka/kafka-clients)库与Kafka代理进行交互。我们将创建一个Java类，该类在内部使用KafkaConsumer订阅主题并记录传入的消息。如果你想深入了解，请随时阅读我们专门介绍[Kafka Consumer API](https://www.baeldung.com/kafka-create-listener-consumer-api)的文章并继续学习。

我们示例中的一个关键区别是日志记录：我们不是一次记录一条消息，而是收集它们并记录整个批次。这样，我们将能够准确地看到每个poll()获取了多少条消息。此外，让我们通过合并批次的初始和最终偏移量以及消费者的groupId等详细信息来丰富日志：

```java
class VariableFetchSizeKafkaListener implements Runnable {
    private final String topic;
    private final KafkaConsumer<String, String> consumer;

    // constructor

    @Override
    public void run() {
        consumer.subscribe(singletonList(topic));
        int pollCount = 1;

        while (true) {
            List<ConsumerRecord<String, String>> records = new ArrayList<>();
            for (var record : consumer.poll(ofMillis(500))) {
                records.add(record);
            }
            if (!records.isEmpty()) {
                String batchOffsets = String.format("%s -> %s", records.get(0).offset(), records.get(records.size() - 1).offset());
                String groupId = consumer.groupMetadata().groupId();
                log.info("groupId: {}, poll: #{}, fetched: #{} records, offsets: {}", groupId, pollCount++, records.size(), batchOffsets);
            }
        }
    }
}
```

**[Testcontainers](https://www.baeldung.com/docker-test-containers)库将通过启动一个运行Kafka代理的Docker容器来帮助我们设置测试环境**，如果你想了解有关设置Testcontainer的Kafka模块的更多信息，请查看我们如何[配置测试环境](https://www.baeldung.com/kafka-create-listener-consumer-api#testing)并继续操作。

在我们的特定情况下，我们可以定义一个附加方法，用于在给定主题上发布多条消息。例如，假设我们将温度传感器读取的值流式传输到名为“engine.sensor.temperature”的主题：

```java
void publishTestData(int recordsCount, String topic) {
    List<ProducerRecord<String, String>> records = IntStream.range(0, recordsCount)
        .mapToObj(__ -> new ProducerRecord<>(topic, "key1", "temperature=255F"))
        .collect(toList());
    // publish all to kafka
}
```

我们可以看到，我们对所有消息使用了相同的键。因此，所有记录都将发送到同一个分区。对于有效载荷，我们使用了一段简短的固定文本来描述温度测量。

## 3. 测试默认行为

**首先，使用默认的消费者配置创建一个Kafka监听器。然后，我们将发布一些消息，以查看我们的监听器消费了多少批次**。如我们所见，我们的自定义监听器在内部使用了Consumer API。因此，要实例化VariableFetchSizeKafkaListener，我们必须首先配置并创建一个KafkaConsumer：

```java
Properties props = new Properties();
props.setProperty(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, KAFKA_CONTAINER.getBootstrapServers());
props.setProperty(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
props.setProperty(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
props.setProperty(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
props.setProperty(ConsumerConfig.GROUP_ID_CONFIG, "default_config");
KafkaConsumer<String, String> kafkaConsumer = new KafkaConsumer<>(props);
```

目前，我们将使用KafkaConsumer的默认值作为最小和最大获取大小。基于此消费者，我们可以实例化监听器并异步运行它，以避免阻塞主线程：

```java
CompletableFuture.runAsync(
    new VariableFetchSizeKafkaListener(topic, kafkaConsumer)
);
```

最后，让我们阻塞测试线程几秒钟，让监听器有时间消费消息。本文的目标是启动监听器并观察它们的性能，我们将使用JUnit 5测试作为设置和探索其行为的便捷方式，但为了简单起见，我们不会包含任何特定断言。因此，这将是我们的起始@Test：

```java
@Test
void whenUsingDefaultConfiguration_thenProcessInBatchesOf() throws Exception {
    String topic = "engine.sensors.temperature";
    publishTestData(300, topic);

    Properties props = new Properties();
    props.setProperty(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, KAFKA_CONTAINER.getBootstrapServers());
    props.setProperty(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
    props.setProperty(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
    props.setProperty(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
    props.setProperty(ConsumerConfig.GROUP_ID_CONFIG, "default_config");
    KafkaConsumer<String, String> kafkaConsumer = new KafkaConsumer<>(props);

    CompletableFuture.runAsync(
      new VariableFetchSizeKafkaListener(topic, kafkaConsumer)
    );

    Thread.sleep(5_000L);
}
```

现在，让我们运行测试并检查日志以查看一批数据中将获取多少条记录：

```text
10:48:46.958 [ForkJoinPool.commonPool-worker-2] INFO  c.t.t.k.c.VariableFetchSizeKafkaListener - groupId: default_config, poll: #1, fetched: #300 records, offsets: 0 -> 299
```

我们可以注意到，由于消息较小，我们在一个批次中获取了所有300条记录。键和正文都是短字符串：键为4个字符，正文为16个字符长，总共20个字节，外加一些记录元数据的额外字节。另一方面，最大批次大小的默认值为1兆字节(1024 x 1024字节)，或简称为1048576字节。

## 4. 配置最大分区获取大小

**Kafka中的“max.partition.fetch.bytes”决定了消费者在单个请求中可以从单个分区获取的最大数据量**。因此，即使对于少量的短消息，我们也可以通过更改属性来强制我们的监听器分批获取记录。

为了观察这一点，让我们再创建两个VariableFetchSizeKafkaListener并将它们配置为仅500B或5KB。首先，让我们在专用方法中提取所有常见的消费者属性，以避免代码重复：

```java
Properties commonConsumerProperties() {
    Properties props = new Properties();
    props.setProperty(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, KAFKA_CONTAINER.getBootstrapServers());
    props.setProperty(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
    props.setProperty(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
    props.setProperty(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
    return props;
}
```

然后，让我们创建第一个监听器并异步运行它：

```java
Properties fetchSize_500B = commonConsumerProperties();
fetchSize_500B.setProperty(ConsumerConfig.GROUP_ID_CONFIG, "max_fetch_size_500B");
fetchSize_500B.setProperty(ConsumerConfig.MAX_PARTITION_FETCH_BYTES_CONFIG, "500");
CompletableFuture.runAsync(
    new VariableFetchSizeKafkaListener("engine.sensors.temperature", new KafkaConsumer<>(fetchSize_500B))
);
```

我们可以看到，我们为不同的监听器设置了不同的消费者组ID，这样它们就可以消费相同的测试数据。现在，让我们继续使用第二个监听器并完成测试：

```java
@Test
void whenChangingMaxPartitionFetchBytesProperty_thenAdjustBatchSizesWhilePolling() throws Exception {
    String topic = "engine.sensors.temperature";
    publishTestData(300, topic);

    Properties fetchSize_500B = commonConsumerProperties();
    fetchSize_500B.setProperty(ConsumerConfig.GROUP_ID_CONFIG, "max_fetch_size_500B");
    fetchSize_500B.setProperty(ConsumerConfig.MAX_PARTITION_FETCH_BYTES_CONFIG, "500");
    CompletableFuture.runAsync(
            new VariableFetchSizeKafkaListener(topic, new KafkaConsumer<>(fetchSize_500B))
    );

    Properties fetchSize_5KB = commonConsumerProperties();
    fetchSize_5KB.setProperty(ConsumerConfig.GROUP_ID_CONFIG, "max_fetch_size_5KB");
    fetchSize_5KB.setProperty(ConsumerConfig.MAX_PARTITION_FETCH_BYTES_CONFIG, "5000");
    CompletableFuture.runAsync(
            new VariableFetchSizeKafkaListener(topic, new KafkaConsumer<>(fetchSize_5KB))
    );

    Thread.sleep(10_000L);
}
```

如果我们运行这个测试，我们可以假设第一个消费者将获取比第二个消费者小大约10倍的批次。让我们分析一下日志：

```text
[worker-3] INFO - groupId: max_fetch_size_5KB, poll: #1, fetched: #56 records, offsets: 0 -> 55
[worker-2] INFO - groupId: max_fetch_size_500B, poll: #1, fetched: #5 records, offsets: 0 -> 4
[worker-2] INFO - groupId: max_fetch_size_500B, poll: #2, fetched: #5 records, offsets: 5 -> 9
[worker-3] INFO - groupId: max_fetch_size_5KB, poll: #2, fetched: #56 records, offsets: 56 -> 111
[worker-2] INFO - groupId: max_fetch_size_500B, poll: #3, fetched: #5 records, offsets: 10 -> 14
[worker-3] INFO - groupId: max_fetch_size_5KB, poll: #3, fetched: #56 records, offsets: 112 -> 167
[worker-2] INFO - groupId: max_fetch_size_500B, poll: #4, fetched: #5 records, offsets: 15 -> 19
[worker-3] INFO - groupId: max_fetch_size_5KB, poll: #4, fetched: #51 records, offsets: 168 -> 218
[worker-2] INFO - groupId: max_fetch_size_500B, poll: #5, fetched: #5 records, offsets: 20 -> 24
[...]
```

正如预期的那样，其中一个监听器确实获取了几乎比另一个大十倍的数据批次。**此外，重要的是要了解批次中的记录数量取决于这些记录及其元数据的大小**。为了强调这一点，我们可以观察到groupId为“max_fetch_size_5KB”的消费者在第4次轮询时获取的记录较少。

## 5. 配置最小获取大小

**Consumer API还允许通过“fetch.min.bytes”属性自定义最小提取大小，我们可以更改此属性以指定代理需要响应的最小数据量**。如果不满足此最小值，代理将等待更长时间，然后再向消费者的提取请求发送响应。为了强调这一点，我们可以在测试辅助方法中为测试发布者添加延迟。因此，生产者将在发送每条消息之间等待特定的毫秒数：

```java
@Test
void whenChangingMinFetchBytesProperty_thenAdjustWaitTimeWhilePolling() throws Exception {
    String topic = "engine.sensors.temperature";
    publishTestData(300, topic, 100L);
    // ...
}

void publishTestData(int measurementsCount, String topic, long delayInMillis) {
    // ...
}
```

让我们首先创建一个使用默认配置的VariableFetchSizeKafkaListener，其中“fetch.min.bytes”等于一个字节。与前面的示例类似，我们将在CompletableFuture中异步运行此消费者：

```java
// fetch.min.bytes = 1 byte (default)
Properties minFetchSize_1B = commonConsumerProperties();
minFetchSize_1B.setProperty(ConsumerConfig.GROUP_ID_CONFIG, "min_fetch_size_1B");
CompletableFuture.runAsync(
    new VariableFetchSizeKafkaListener(topic, new KafkaConsumer<>(minFetchSize_1B))
);
```

**使用此设置，并且由于我们引入了延迟，我们可以预期每个记录都会被单独检索，一个接一个**。换句话说，我们可以预期单个记录会有很多批次。此外，我们预计这些批次的消费速度与我们的KafkaProducer发布数据的速度相似，在我们的例子中是每100毫秒一次。让我们运行测试并分析日志：

```text
14:23:22.368 [worker-2] INFO - groupId: min_fetch_size_1B, poll: #1, fetched: #1 records, offsets: 0 -> 0
14:23:22.472 [worker-2] INFO - groupId: min_fetch_size_1B, poll: #2, fetched: #1 records, offsets: 1 -> 1
14:23:22.582 [worker-2] INFO - groupId: min_fetch_size_1B, poll: #3, fetched: #1 records, offsets: 2 -> 2
14:23:22.689 [worker-2] INFO - groupId: min_fetch_size_1B, poll: #4, fetched: #1 records, offsets: 3 -> 3
[...]
```

此外，我们可以通过将“fetch.min.bytes”值调整为更大的大小来强制消费者等待更多数据的积累：

```java
// fetch.min.bytes = 500 bytes
Properties minFetchSize_500B = commonConsumerProperties();
minFetchSize_500B.setProperty(ConsumerConfig.GROUP_ID_CONFIG, "mim_fetch_size_500B");
minFetchSize_500B.setProperty(ConsumerConfig.FETCH_MIN_BYTES_CONFIG, "500");
CompletableFuture.runAsync(
    new VariableFetchSizeKafkaListener(topic, new KafkaConsumer<>(minFetchSize_500B))
);
```

将该属性设置为500字节后，我们可以预期消费者将等待更长时间并获取更多数据。让我们也运行此示例并观察结果：

```text
14:24:49.303 [worker-3] INFO - groupId: mim_fetch_size_500B, poll: #1, fetched: #6 records, offsets: 0 -> 5
14:24:49.812 [worker-3] INFO - groupId: mim_fetch_size_500B, poll: #2, fetched: #4 records, offsets: 6 -> 9
14:24:50.315 [worker-3] INFO - groupId: mim_fetch_size_500B, poll: #3, fetched: #5 records, offsets: 10 -> 14
14:24:50.819 [worker-3] INFO - groupId: mim_fetch_size_500B, poll: #4, fetched: #5 records, offsets: 15 -> 19
[...]
```

## 6. 总结

在本文中，我们讨论了KafkaConsumer从代理获取数据的方式。我们了解到，默认情况下，如果有至少一条新记录，消费者将获取数据。另一方面，如果分区中的新数据超过1048576字节，它将被拆分为多个最大大小的批次。我们发现，自定义“fetch.min.bytes”和“max.partition.fetch.bytes”属性使我们能够定制Kafka的行为以满足我们的特定要求。