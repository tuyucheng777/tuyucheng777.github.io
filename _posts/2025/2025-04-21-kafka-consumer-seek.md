---
layout: post
title:  Kafka中的消费者搜索
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 概述

**Kafka中的查找类似于在读取之前先在磁盘上定位存储的数据，在从分区读取数据之前，我们必须首先查找到正确的位置**。 

Kafka消费者[偏移量](https://www.baeldung.com/kafka-commit-offsets#what-is-offset)是一个唯一、不断递增的数字，用于标记事件记录在分区中的位置。组中的每个消费者都会为每个分区保留自己的偏移量，以便跟踪进度。

消费者可能需要在分区中的不同位置处理消息，例如重播事件或跳到最新消息。

在本教程中，让我们探索Spring Kafka API方法来检索分区内各个位置的消息。

## 2. 使用Java API进行查找

大多数情况下，消费者会从分区的起始位置读取消息，并持续监听新的消息。然而，在某些情况下，我们可能需要从特定位置、时间或相对位置读取消息。

让我们探索一个API，它提供不同的端点，通过指定偏移量或从开头或结尾读取来从分区中检索记录。

### 2.1 通过偏移量查找

**Spring Kafka提供了[seek()](https://kafka.apache.org/0100/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html#seek(org.apache.kafka.common.TopicPartition,long))方法将读取器定位在分区内的给定偏移量处**。

让我们首先通过获取分区和偏移量值来探索在分区内按偏移量进行查找：

```java
@GetMapping("partition/{partition}/offset/{offset}")
public ResponseEntity<Response> getOneByPartitionAndOffset(@PathVariable("partition") int partition,
                                                           @PathVariable("offset") int offset) {
    try (KafkaConsumer<String, String> consumer = (KafkaConsumer<String, String>) consumerFactory.createConsumer()) {
        TopicPartition topicPartition = new TopicPartition(TOPIC_NAME, partition);
        consumer.assign(Collections.singletonList(topicPartition));
        consumer.seek(topicPartition, offset);
        ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
        Iterator<ConsumerRecord<String, String>> recordIterator = records.iterator();
        if (recordIterator.hasNext()) {
            ConsumerRecord<String, String> consumerRecord = recordIterator.next();
            Response response = new Response(consumerRecord.partition(), consumerRecord.offset(), consumerRecord.value());
            return new ResponseEntity<>(response, HttpStatus.OK);
        }
    }
    return new ResponseEntity<>(HttpStatus.NOT_FOUND);
}
```

这里，API暴露了一个端点：partition/{partition}/offset/{offset}，它将主题、分区和偏移量传递给seek()方法，从而定位消费者在指定位置检索消息。响应模型包含分区、偏移量和消息内容：

```java
public record Response(int partition, long offset, String value) { }
```

为简单起见，该API仅在指定位置检索一条记录。但是，我们可以对其进行修改，以恢复从该偏移量开始的所有消息；它不处理给定偏移量不可用的情况。

为了测试这一点，作为第一步，让我们添加一个在所有测试之前运行的方法，向指定主题生成5条简单消息：

```java
@BeforeAll
static void beforeAll() {
    // set producer config for the broker
    testKafkaProducer = new KafkaProducer<>(props);
    int partition = 0;
    IntStream.range(0, 5)
            .forEach(m -> {
                String key = String.valueOf(new Random().nextInt());
                String value = "Message no : %s".formatted(m);
                ProducerRecord<String, String> record = new ProducerRecord<>(TOPIC_NAME,
                        partition,
                        key,
                        value
                );
                try {
                    testKafkaProducer.send(record).get();
                } catch (InterruptedException | ExecutionException e) {
                    throw new RuntimeException(e);
                }
            });
}
```

这里设置了生产者配置，向分区0发送5条格式为“Message no: %s”.formatted(m)的消息，其中m代表0到4之间的整数。

接下来，让我们添加一个测试，通过传递分区0和偏移量2来调用上述端点：

```java
@Test
void givenKafkaBrokerExists_whenSeekByPartition_thenMessageShouldBeRetrieved() {
    this.webClient.get()
        .uri("/seek/api/v1/partition/0/offset/2")
        .exchange()
        .expectStatus()
        .isOk()
        .expectBody(String.class)
        .isEqualTo("{\"partition\":0,\"offset\":2,\"value\":\"Message no : 2\"}");
}
```

通过调用此API端点，我们可以看到位于偏移量2处的第三条消息已成功接收。

### 2.2 按开头查找

**[seekToBeginning()](https://kafka.apache.org/0100/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html#seekToBeginning(java.util.Collection))方法将消费者定位在分区的开始处，允许它从第一个消息开始检索消息**。

接下来，让我们添加一个端点，在分区的开头公开第一条消息：

```java
@GetMapping("partition/{partition}/beginning")
public ResponseEntity<Response> getOneByPartitionToBeginningOffset(@PathVariable("partition") int partition) {
    try (KafkaConsumer<String, String> consumer = (KafkaConsumer<String, String>) consumerFactory.createConsumer()) {
        TopicPartition topicPartition = new TopicPartition(TOPIC_NAME, partition);
        consumer.assign(Collections.singletonList(topicPartition));
        consumer.seekToBeginning(Collections.singleton(topicPartition));
        ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
        Iterator<ConsumerRecord<String, String>> recordIterator = records.iterator();
        if (recordIterator.hasNext()) {
            ConsumerRecord<String, String> consumerRecord = recordIterator.next();
            Response response = new Response(consumerRecord.partition(), consumerRecord.offset(), consumerRecord.value());
            return new ResponseEntity<>(response, HttpStatus.OK);
        }
    }
    return new ResponseEntity<>(HttpStatus.NOT_FOUND);
}
```

这里，API提供了端点partion/{partition}/beginning，并将主题和分区传递给seekToBeginning()方法，这将使消费者从分区的起始位置读取消息。响应包含分区、偏移量和消息内容。

接下来，让我们添加一个测试来检索分区0开头的消息。请注意，测试的@BeforeAll部分确保生产者向测试主题推送了5条消息：

```java
@Test
void givenKafkaBrokerExists_whenSeekByBeginning_thenFirstMessageShouldBeRetrieved() {
    this.webClient.get()
        .uri("/seek/api/v1/partition/0/beginning")
        .exchange()
        .expectStatus()
        .isOk()
        .expectBody(String.class)
        .isEqualTo("{\"partition\":0,\"offset\":0,\"value\":\"Message no : 0\"}");
}
```

我们可以通过调用此API端点来检索存储在偏移量0处的第一条消息。

### 2.3 按结尾查找

**[seekToEnd()](https://kafka.apache.org/0100/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html#seekToEnd(java.util.Collection))方法将消费者定位在分区的末尾，允许它检索附加的任何未来消息**。

接下来，让我们创建一个寻找分区末尾偏移位置的端点：

```java
@GetMapping("partition/{partition}/end")
public ResponseEntity<Long> getOneByPartitionToEndOffset(@PathVariable("partition") int partition) {
    try (KafkaConsumer<String, String> consumer = (KafkaConsumer<String, String>) consumerFactory.createConsumer()) {
        TopicPartition topicPartition = new TopicPartition(TOPIC_NAME, partition);
        consumer.assign(Collections.singletonList(topicPartition));
        consumer.seekToEnd(Collections.singleton(topicPartition));
        return new ResponseEntity<>(consumer.position(topicPartition), HttpStatus.OK);
    }
}
```

此API提供端点partition/{partition}/end，将主题和分区传递给seekToEnd()方法，这将使消费者从分区末尾读取消息。

由于查找末尾意味着没有可用的新消息，因此此API会显示分区内的当前偏移位置，让我们添加一个测试来验证这一点：

```java
@Test
void givenKafkaBrokerExists_whenSeekByEnd_thenLastMessageShouldBeRetrieved() {
    this.webClient.get()
        .uri("/seek/api/v1/partition/0/end")
        .exchange()
        .expectStatus()
        .isOk()
        .expectBody(Long.class)
        .isEqualTo(5L);
}
```

使用seekToEnd()会将消费者移动到下一个将写入下一条消息的偏移量，使其位于上一条可用消息之后一个位置。当我们调用此API端点时，响应将返回上一个偏移量加1。

### 2.4 通过实现ConsumerSeekAware类来实现查找

除了使用消费者API在特定位置读取消息之外，我们还可以扩展Spring Kafka中的[AbstractConsumerSeekAware](https://docs.spring.io/spring-kafka/api/org/springframework/kafka/listener/AbstractConsumerSeekAware.html)类。**该类允许消费者动态控制Kafka分区中的查找**，它提供了在分区分配期间查找特定偏移量或时间戳的方法，从而可以更精细地控制消息消费。

除了上述查找方法之外，AbstractConsumerSeekAware还提供从特定时间戳搜索或相对位置搜索。

让我们在本节中探讨相对位置搜索：

```java
void seekRelative(java.lang.String topic, int partition, long offset, boolean toCurrent)
```

**Spring Kafka中的[seekRelative()](https://javadoc.io/doc/org.springframework.kafka/spring-kafka/2.6.9/org/springframework/kafka/listener/ConsumerSeekAware.ConsumerSeekCallback.html#seekRelative(java.lang.String,int,long,boolean))方法允许消费者在分区内查找相对于当前偏移量或起始偏移量的位置**，每个参数都有特定的作用：

- **topic**：从中读取消息的Kafka主题的名称
- **partition**：主题中发生搜索的分区号
- **offset**：相对于当前偏移量或起始偏移量移动的位置数，可以为正数或负数
- **toCurrent**：boolean值，如果为true，则该方法相对于当前偏移量进行查找；如果为false，则该方法相对于分区的起始位置进行查找。

让我们添加一个自定义监听器，使用seekRelative() API在分区内查找最新消息：

```java
@Component
class ConsumerListener extends AbstractConsumerSeekAware {

    public static final Map<String, String> MESSAGES = new HashMap<>();

    @Override
    public void onPartitionsAssigned(Map<TopicPartition, Long> assignments, ConsumerSeekCallback callback) {
        assignments.keySet()
                .forEach(tp -> callback.seekRelative(tp.topic(), tp.partition(), -1, false));
    }

    @KafkaListener(id = "test-seek", topics = "test-seek-topic")
    public void listen(ConsumerRecord<String, String> in) {
        MESSAGES.put(in.key(), in.value());
    }
}
```

[onPartitionsAssigned](https://docs.spring.io/spring-kafka/api/org/springframework/kafka/listener/ConsumerSeekAware.html#onPartitionsAssigned(java.util.Map,org.springframework.kafka.listener.ConsumerSeekAware.ConsumerSeekCallback))方法中调用seekRelative()方法，在消费者收到分区分配时手动调整其位置。

偏移量为-1表示消费者从参考点向后移动一个位置，在本例中，由于toCurrent设置为false，它指示消费者相对于分区末尾进行寻址，这意味着消费者将从最后一条可用消息向后移动一个位置。

内存中的HashMap跟踪读取的消息以进行测试，并将接收到的消息存储为字符串。

最后，让我们添加一个测试，通过检查Map来验证系统是否成功检索到偏移量4处的消息：

```java
@Test
void givenKafkaBrokerExists_whenMessagesAreSent_ThenLastMessageShouldBeRetrieved() {
    Map<String, String> messages = consumerListener.MESSAGES;
    Assertions.assertEquals(1, messages.size());
    Assertions.assertEquals("Message no : 4", messages.get("4"));
}
```

测试的@BeforeAll部分确保生产者向测试主题推送5条消息，查找配置成功检索到分区中的最后一条消息。

## 3. 总结

在本教程中，我们探讨了Kafka消费者如何使用Spring Kafka在分区中寻找特定位置。

我们首先研究了使用消费者API进行定位，这在需要精确控制分区中的读取位置时非常有用。此方法最适合诸如重放事件、跳过某些消息或基于偏移量应用自定义逻辑等场景。

接下来，我们研究了使用监听器时执行定位操作，这种方法更适合持续消费消息。由于enable.auto.commit属性默认设置为true，因此该方法会在处理后定期自动提交偏移量。