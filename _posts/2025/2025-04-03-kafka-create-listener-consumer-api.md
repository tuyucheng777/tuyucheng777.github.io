---
layout: post
title:  使用Consumer API创建Kafka监听器
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 概述

在本教程中，我们将学习如何创建[Kafka](https://www.baeldung.com/apache-kafka)监听器并使用Kafka的Consumer API从主题中消费消息。之后，我们将使用Producer API和[Testcontainers](https://www.baeldung.com/docker-test-containers)测试我们的实现。

我们将专注于设置不依赖Spring Boot模块的KafkaConsumer。

## 2. 创建自定义Kafka监听器

**我们的自定义监听器将在内部使用[kafka-clients](https://mvnrepository.com/artifact/org.apache.kafka/kafka-clients)库中的生产者和消费者API**，让我们首先将此依赖添加到pom.xml文件中：

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>3.6.1</version>
</dependency>

```

对于本文中的代码示例，我们将创建一个CustomKafkaListener类，该类将监听名为“tuyucheng.articles.published”的主题。在内部，我们的类将围绕KafkaConsumer并利用它来订阅主题：

```java
class CustomKafkaListener {
    private final String topic;
    private final KafkaConsumer<String, String> consumer;

    // ...
}
```

### 2.1 创建KafkaConsumer

**要创建KafkaConsumer，我们需要通过Properties对象提供有效的配置**。让我们创建一个简单的消费者，可以在创建CustomKafkaListener实例时将其用作默认值：

```java
public CustomKafkaListener(String topic, String bootstrapServers) {
    this(topic, defaultKafkaConsumer(bootstrapServers));
}

static KafkaConsumer<String, String> defaultKafkaConsumer(String boostrapServers) {
    Properties props = new Properties();
    props.setProperty(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, boostrapServers);
    props.setProperty(ConsumerConfig.GROUP_ID_CONFIG, "test_group_id");
    props.setProperty(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
    props.setProperty(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
    props.setProperty(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
    return new KafkaConsumer<>(props);
}
```

对于此示例，我们对大多数这些属性进行了硬编码，但理想情况下，它们应该从配置文件中加载。让我们快速看看每个属性的含义：

- Boostrap Servers：用于建立与Kafka集群的初始连接的主机和端口对列表
- Group ID：允许一组消费者共同消费一组主题分区的ID
- Auto Offset Reset：当没有初始偏移时，Kafka日志中开始读取数据的位置
- Key/Value Deserializers：键和值的反序列化器类。在我们的示例中，我们将使用String键和值以及以下反序列化器：org.apache.kafka.common.serialization.StringDeserializer

通过这个最小配置，我们将能够订阅主题并轻松测试实现。有关可用属性的完整列表，请参阅[官方文档](https://docs.confluent.io/platform/current/installation/configuration/consumer-configs.html)。

### 2.2 订阅主题

**现在，我们需要提供一种方法来订阅主题并开始轮询消息。这可以使用KafkaConsumer的subscribe()方法来实现，然后无限循环调用poll()方法**。此外，由于此方法将阻塞线程，我们可以实现Runnable接口以提供与CompletableFuture的良好集成：

```java
class CustomKafkaListener implements Runnable {
    private final String topic;
    private final KafkaConsumer<String, String> consumer;

    // constructors

    @Override
    void run() {
        consumer.subscribe(Arrays.asList(topic));
        while (true) {
            consumer.poll(Duration.ofMillis(100))
                    .forEach(record -> log.info("received: " + record));
        }
    }
}
```

现在，我们的CustomKafkaListener可以像这样启动而不会阻塞主线程：

```java
String topic = "tuyucheng.articles.published";
String bootstrapServers = "localhost:9092";

var listener = new CustomKafkaListener(topic, bootstrapServers)
CompletableFuture.runAsync(listener);
```

### 2.3 消费记录

目前，我们的应用程序仅监听主题并记录所有传入消息，让我们进一步改进它以允许更复杂的场景并使其更易于测试。例如，**我们可以允许定义一个Consumer<String\>来接收来自主题的每个新事件**：

```java
class CustomKafkaListener implements Runnable {
    private final String topic;
    private final KafkaConsumer<String, String> consumer;
    private Consumer<String> recordConsumer;

    CustomKafkaListener(String topic, KafkaConsumer<String, String> consumer) {
        this.topic = topic;
        this.consumer = consumer;
        this.recordConsumer = record -> log.info("received: " + record);
    }

    // ...

    @Override
    public void run() {
        consumer.subscribe(Arrays.asList(topic));
        while (true) {
            consumer.poll(Duration.ofMillis(100))
                    .forEach(record -> recordConsumer.accept(record.value()));
        }
    }

    CustomKafkaListener onEach(Consumer newConsumer) {
        recordConsumer = recordConsumer.andThen(newConsumer);
        return this;
    }
}
```

将recordConsumer声明为Consumer<String\>允许我们使用默认方法andThen()链接多个函数，对于每个传入消息，将逐个调用这些函数。

## 3. 测试

为了测试我们的实现，我们将创建一个KafkaProducer并使用它将一些消息发布到“tuyucheng.articles.published”主题。然后，我们将启动CustomKafkaListener并验证它是否准确处理所有活动。

### 3.1 设置Kafka测试容器

**我们可以利用Testcontainers库在我们的测试环境中启动Kafka容器**。首先，我们需要为[JUnit5扩展](https://mvnrepository.com/artifact/org.testcontainers/junit-jupiter)和[Kafka模块](https://mvnrepository.com/artifact/org.testcontainers/kafka)添加Testcontainer依赖：

```xml
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>kafka</artifactId>
    <version>1.19.3</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>1.19.3</version>
    <scope>test</scope>
</dependency>
```

现在，我们可以创建一个具有特定Docker镜像名称的KafkaContainer，然后，我们将添加@Container和@Testcontainers注解，以允许Testcontainers JUnit5扩展管理容器的生命周期：

```java
@Testcontainers
class CustomKafkaListenerLiveTest {

    @Container
    private static final KafkaContainer KAFKA_CONTAINER = new KafkaContainer(DockerImageName.parse("confluentinc/cp-kafka:latest"));

    // ...
}
```

### 3.2 创建并启动监听器

首先，我们将主题名称定义为硬编码字符串，并从KAFKA_CONTAINER中提取bootstrapServers。此外，我们将创建一个用于收集消息的ArrayList<String\>：

```java
String topic = "tuyucheng.articles.published";
String bootstrapServers = KAFKA_CONTAINER.getBootstrapServers();
List<String> consumedMessages = new ArrayList<>();
```

**我们将使用这些属性创建CustomKafkaListener的实例，并指示它捕获新消息并将其添加到consumerMessages列表中**：

```java
CustomKafkaListener listener = new CustomKafkaListener(topic, bootstrapServers).onEach(consumedMessages::add);
listener.run();
```

但是，请务必注意，按原样运行它可能会阻塞线程并冻结测试。为了防止这种情况，我们将使用CompletableFuture异步执行它：

```java
CompletableFuture.runAsync(listener);
```

虽然对于测试来说并不重要，但我们首先也可以在try-with-resources块中实例化监听器：

```java
var listener = new CustomKafkaListener(topic, bootstrapServers).onEach(consumedMessages::add);
CompletableFuture.runAsync(listener);
```

### 3.3 发布消息

**为了将文章名称发送到“tuyucheng.articles.published”主题，我们将使用Properties对象设置一个KafkaProducer**，其方法与我们为消费者所做的类似。

```java
static KafkaProducer<String, String> testKafkaProducer() {
    Properties props = new Properties();
    props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, KAFKA_CONTAINER.getBootstrapServers());
    props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
    props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
    return new KafkaProducer<>(props);
}
```

此方法将允许我们发布消息来测试我们的实现，让我们创建另一个测试工具方法，它将为作为参数收到的每篇文章发送一条消息：

```java
private void publishArticles(String topic, String... articles) {
    try (KafkaProducer<String, String> producer = testKafkaProducer()) {
        Arrays.stream(articles)
                .map(article -> new ProducerRecord<String,String>(topic, article))
                .forEach(producer::send);
    }
}
```

### 3.4 验证

让我们把所有部分放在一起并运行测试，我们已经讨论了如何创建CustomKafkaListener并开始发布数据：

```java
@Test
void givenANewCustomKafkaListener_thenConsumesAllMessages() {
    // given
    String topic = "tuyucheng.articles.published";
    String bootstrapServers = KAFKA_CONTAINER.getBootstrapServers();
    List<String> consumedMessages = new ArrayList<>();

    // when
    CustomKafkaListener listener = new CustomKafkaListener(topic, bootstrapServers).onEach(consumedMessages::add);
    CompletableFuture.runAsync(listener);

    // and
    publishArticles(topic,
            "Introduction to Kafka",
            "Kotlin for Java Developers",
            "Reactive Spring Boot",
            "Deploying Spring Boot Applications",
            "Spring Security"
    );

    // then
    // ...
}
```

我们的最终任务是等待异步代码完成并确认consumedMessages列表包含预期内容。为此，我们将使用[Awaitility](https://www.baeldung.com/awaitility-testing)库，利用其await().untilAsserted()：

```java
// then
await().untilAsserted(() -> 
    assertThat(consumedMessages).containsExactlyInAnyOrder(
        "Introduction to Kafka",
        "Kotlin for Java Developers",
        "Reactive Spring Boot",
        "Deploying Spring Boot Applications",
        "Spring Security"
    ));
```

## 4. 总结

在本教程中，我们学习了如何在不依赖更高级别的Spring模块的情况下使用Kafka的消费者和生产者API。首先，我们使用封装了KafkaConsumer的CustomKafkaListener创建了一个消费者。为了进行测试，我们实现了KafkaProducer，并使用Testcontainers和Awaitility验证了我们的设置。