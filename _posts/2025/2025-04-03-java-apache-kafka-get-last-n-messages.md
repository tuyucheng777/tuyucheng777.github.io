---
layout: post
title:  获取Apache Kafka主题中的最后N条消息
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 简介

在这个简短的教程中，我们将了解如何从Apache Kafka主题中检索最后N条消息。

在本文的第一部分中，我们将重点介绍执行此操作所需的先决条件。在第二部分中，我们将使用[Kafka Java API](https://kafka.apache.org/documentation/#api)库构建一个小型实用程序来使用Java读取消息。最后，我们将提供简短的指导，以便使用[KafkaCat](https://github.com/edenhill/kcat)从命令行实现相同的结果。

## 2. 先决条件

**要从Kafka主题中检索最后N条消息，只需从明确定义的偏移量开始消费消息即可，Kafka主题中的偏移量表示消费者的当前位置**。在上一篇文章中，我们已经了解了如何利用[consumer.seekToEnd()](https://kafka.apache.org/21/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html#seekToEnd-java.util.Collection-)方法[在Apache Kafka主题中获取特定数量的消息](https://www.baeldung.com/java-kafka-count-topic-messages)。

考虑相同的功能，我们可以通过执行一个简单的减法来计算正确的偏移量：offset = lastOffset - N，然后我们可以从这个位置开始轮询N条消息。

尽管如此，如果我们使用事务生产者生成记录，则此方法将不起作用。在这种情况下，偏移量将跳过一些数字以适应Kafka主题事务记录(提交/回滚等)。使用事务生产者的一个常见情况是我们需要[只处理一次Kafka消息](https://www.baeldung.com/kafka-exactly-once)。简而言之，如果我们从(lastOffset - N)开始读取消息，我们可能会消费少于N条消息，因为一些偏移量数字已[被事务记录消费](https://issues.apache.org/jira/browse/KAFKA-10009)。

## 3. 使用Java获取Kafka主题中的最后N条消息

首先，我们需要创建一个生产者和一个消费者：

```java
Properties producerProperties = new Properties();
producerProperties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, KAFKA_CONTAINER.getBootstrapServers());
producerProperties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
producerProperties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

Properties consumerProperties = new Properties();
consumerProperties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, KAFKA_CONTAINER.getBootstrapServers());
consumerProperties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
consumerProperties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
consumerProperties.put(ConsumerConfig.GROUP_ID_CONFIG, "ConsumerGroup1");

KafkaProducer<String, String> producer = new KafkaProducer<>(producerProperties);
KafkaConsumer<String, String> consumer = new KafkaConsumer<>(consumerProperties);
```

现在让我们生成一些消息：

```java
final String TOPIC1 = "tuyucheng-topic";
int messagesInTopic = 100;
for (int i = 0; i < messagesInTopic; i++) {
    producer.send(new ProducerRecord(TOPIC1, null, MESSAGE_KEY, String.valueOf(i))).get();
}
```

为了清晰和简单起见，我们假设我们只需要为消费者注册一个分区：

```java
TopicPartition partition = new TopicPartition(TOPIC1, 0);
List<TopicPartition> partitions = new ArrayList<>();
partitions.add(partition);
consumer.assign(partitions);
```

正如我们之前提到的，我们需要将偏移量定位在正确的位置，然后我们就可以开始轮询了：

```java
int messagesToRetrieve = 10;
consumer.seekToEnd(partitions);
long startIndex = consumer.position(partition) - messagesToRetrieve;
consumer.seek(partition, startIndex);
ConsumerRecords<String, String> records = consumer.poll(Duration.ofMinutes(1));
```

**如果网络特别慢，或者要检索的消息数量特别大，我们可能需要增加轮询持续时间**。在这种情况下，我们需要考虑内存中有大量记录可能会导致资源短缺问题。

现在让我们最终检查一下我们是否真正检索到了正确数量的消息：

```java
for (ConsumerRecord<String, String> record : records) {
    assertEquals(MESSAGE_KEY, record.key());
    assertTrue(Integer.parseInt(record.value()) >= (messagesInTopic - messagesToRetrieve));
    recordsReceived++;
}
assertEquals(messagesToRetrieve, recordsReceived);
```

## 4. 使用KafkaCat获取Kafka主题中的最后N条消息

[KafkaCat](https://github.com/edenhill/kcat)(kcat)是一个命令行工具，我们可以使用它来测试和调试Kafka主题。Kafka本身提供了大量脚本和shell工具来执行相同的操作，尽管如此，KafkaCat的简单性和易用性使其成为执行诸如检索Apache Kafka主题中的最后N条消息等操作的事实标准。[安装](https://github.com/edenhill/kcat#install)后，可以通过运行以下简单命令来检索Kafka主题中生成的最新N条消息：

```shell
$ kafkacat -C -b localhost:9092 -t topic-name -o -<N> -e
```

- -C表示我们需要消费消息
- -b表示Kafka Broker的位置
- -t表示主题名称
- -o表示需要从这个偏移量开始读取，负号表示需要从末尾读取N条消息
- -e选项在读完最后一条消息后退出

联系到我们上面讨论的案例，从名为“tuyucheng-topic”的主题中检索最后10条消息的命令是：

```shell
$ kafkacat -C -b localhost:9092 -t tuyucheng-topic -o -10 -e
```

## 5. 总结

在本简短教程中，我们了解了如何消费Kafka主题的最新N条消息。在第一部分中，我们使用了Java Kafka API库。在第二部分中，我们使用了名为KafkaCat的命令行实用程序。