---
layout: post
title:  向Kafka消息添加自定义标头
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 简介

[Apache Kafka](https://www.baeldung.com/spring-kafka)是一个开源分布式事件存储和容错流处理系统，**Kafka基本上是一个事件流平台，客户端可以在其中发布和订阅事件流**。通常，生产者应用程序将事件发布到Kafka，而消费者订阅这些事件，从而实现发布者-订阅者模型。

在本教程中，我们将学习如何使用Kafka生产者在Kafka消息中添加自定义标头。

## 2. 设置

Kafka提供了一个易于使用的Java库，我们可以使用它来创建Kafka生产者客户端(Producers)和消费者客户端(Consumers)。

### 2.1 依赖

首先，让我们将Kafka Clients Java库的[Maven依赖](https://mvnrepository.com/artifact/org.apache.kafka/kafka-clients)添加到项目pom.xml文件中：

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>3.4.0</version>
</dependency>
```

### 2.2 连接初始化

本指南假设我们在本地系统上运行着一个Kafka集群；此外，我们需要创建一个主题并与Kafka集群建立连接。

首先，让我们在集群中创建一个Kafka主题。可以参考我们的[Kafka主题创建指南](https://www.baeldung.com/kafka-topic-creation)创建一个主题“tuyucheng”。

其次，让我们创建一个新的[Properties](https://www.baeldung.com/java-properties)实例，其中包含将生产者连接到本地代理所需的最低限度的配置：

```java
Properties producerProperties = new Properties();
producerProperties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
producerProperties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
producerProperties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
```

最后，让我们创建一个KafkaProducer实例，用于发布消息：

```java
KafkaProducer <String, String> producer = new KafkaProducer<>(producerProperties);
```

KafkaProducer类的构造函数接收具有[bootstrap.servers](https://kafka.apache.org/documentation/#adminclientconfigs_bootstrap.servers)属性的Properties对象(或Map)并返回KafkaProducer的实例。

以类似的方式，让我们创建一个KafkaConsumer的实例，用于消费消息：

```java
Properties consumerProperties = new Properties();
consumerProperties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, KAFKA_CONTAINER.getBootstrapServers());
consumerProperties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
consumerProperties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
consumerProperties.put(ConsumerConfig.GROUP_ID_CONFIG, "ConsumerGroup1");

KafkaConsumer<String, String> consumer = new KafkaConsumer<>(consumerProperties);
```

我们将使用这些生产者和消费者实例来演示我们所有的编码示例。

现在我们已经配置了所有必要的依赖和连接，我们可以编写一个简单的应用程序来在Kafka消息中添加自定义标头。

## 3. 使用自定义标头发布消息

**Kafka版本0.11.0.0中添加了对Kafka消息中自定义标头的支持**，要创建Kafka消息(Record)，我们创建ProducerRecord<K,V\>的实例。ProducerRecord基本上标识了要发布消息的消息值和主题以及其他元数据。

**ProducerRecord类提供了各种构造函数来向Kafka消息添加自定义标头**，让我们来看看可以使用的几个构造函数：

- ProducerRecord(String topic, Integer partition, K key, V value, Iterable<Header\> headers)
- ProducerRecord(String topic, Integer partition, Long timestamp, K key, V value, Iterable<Header\> headers)

**ProducerRecord类的构造函数都接收Iterable<Header\>类型形式的自定义标头**。

为了理解，让我们创建一个ProducerRecord，将消息连同一些自定义标头一起发布到“tuyucheng”主题：

```java
List <Header> headers = new ArrayList<>();
headers.add(new RecordHeader("website", "tuyucheng.com".getBytes()));
ProducerRecord <String, String> record = new ProducerRecord <>("tuyucheng", null, "message", "Hello World", headers);

producer.send(record);
```

在这里，我们创建一个Header类型的列表，将其作为标头传递给构造函数。每个标头代表一个RecordHeader(String key, byte[] value)实例，该实例接收标头键作为字符串，接收标头值作为字节数组。

以类似的方式，我们可以使用第二个构造函数，它另外接收正在发布的记录的时间戳：

```java
List <Header> headers = new ArrayList<>();
headers.add(new RecordHeader("website", "tuyucheng.com".getBytes()));
ProducerRecord <String, String> record = new ProducerRecord <>("tuyucheng", null, System.currentTimeMillis(), "message", "Hello World", headers);

producer.send(record);
```

到目前为止，我们已经创建了一条带有自定义标头的消息并将其发布到Kafka。

接下来，让我们实现消费者代码来消费消息并验证其自定义标头。

## 4. 消费带有自定义标头的消息

首先，我们在Kafka主题“tuyucheng”上订阅消费者实例，以消费来自以下来源的消息：

```java
consumer.subscribe(Arrays.asList("tuyucheng"));
```

其次，我们使用轮询机制来轮询来自Kafka的新消息：

```java
ConsumerRecords<String, String> records = consumer.poll(Duration.ofMinutes(1));
```

KafkaConsumer.poll(Duration duration)方法轮询Kafka主题中的新消息，直到Duration参数指定的时间。该方法返回包含已获取消息的ConsumerRecords实例，ConsumerRecords基本上是ConsumerRecord类型的Iterable实例。

最后，我们循环遍历获取的记录并获取每条消息的自定义标头：

```java
for (ConsumerRecord<String, String> record : records) {
    System.out.println(record.key());
    System.out.println(record.value());

    Headers consumedHeaders = record.headers();
    for (Header header : consumedHeaders) {
        System.out.println(header.key());
        System.out.println(new String(header.value()));
    }
}
```

在这里，我们使用ConsumerRecord类中的各种Getter方法来获取消息键、值和自定义标头。**ConsumerRecord.headers()方法返回包含自定义标头的Headers实例**，Headers基本上是Header类型的Iterable实例。然后，我们循环遍历每个Header实例，并分别使用Header.key()和Header.value()方法获取标头键和值。

## 5. 总结

在本文中，我们学习了如何向Kafka消息添加自定义标头，我们研究了接收自定义标头的不同构造函数及其相应的实现。

然后我们看到了如何消费带有自定义标头的消息并验证它们。