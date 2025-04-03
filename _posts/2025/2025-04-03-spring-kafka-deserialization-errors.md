---
layout: post
title:  如何捕获Spring Kafka中的反序列化错误
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 概述

在本文中，我们将了解[Spring Kafka](https://www.baeldung.com/spring-kafka)的RecordDeserializationException。之后，我们将创建一个自定义错误处理程序来捕获此异常并跳过无效消息，从而允许消费者继续处理下一个事件。

本文依赖于Spring Boot的Kafka模块，这些模块提供了与代理交互的便捷工具。

## 2. 创建Kafka监听器

**对于本文中的代码示例，我们将使用一个小型应用程序来监听主题“tuyucheng.articles.published”并处理传入的消息。为了展示自定义错误处理，我们的应用程序在遇到反序列化异常后应继续消费消息**。

Spring Kafka的版本将由[父Spring Boot](https://www.baeldung.com/spring-boot-dependency-management-custom-parent) pom自动解析，因此，我们只需添加模块依赖：

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

该模块使我们能够使用@KafkaListener注解，这是[Kafka Consumer API](https://www.baeldung.com/kafka-create-listener-consumer-api)的抽象，让我们利用此注解来创建ArticlesPublishedListener组件。此外，我们将引入另一个组件EmailService，它将对每条传入消息执行一项操作：

```java
@Component
class ArticlesPublishedListener {
    private final EmailService emailService;

    // constructor

    @KafkaListener(topics = "tuyucheng.articles.published")
    public void onArticlePublished(ArticlePublishedEvent event) {
        emailService.sendNewsletter(event.article());
    }
}

record ArticlePublishedEvent(String article) {
}
```

对于[消费者配置](https://www.baeldung.com/spring-kafka#1-consumer-configuration)，我们将重点定义对我们的示例至关重要的属性。当我们开发生产应用程序时，我们可以调整这些属性以满足我们的特定需求，或将它们外部化到单独的配置文件中：

```java
@Bean
ConsumerFactory<String, ArticlePublishedEvent> consumerFactory(
                @Value("${spring.kafka.bootstrap-servers}") String bootstrapServers
        ) {
    Map<String, Object> props = new HashMap<>();
    props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
    props.put(ConsumerConfig.GROUP_ID_CONFIG, "tuyucheng-app-1");
    props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
    props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
    props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, JsonDeserializer.class);
    return new DefaultKafkaConsumerFactory<>(
            props,
            new StringDeserializer(),
            new JsonDeserializer<>(ArticlePublishedEvent.class)
    );
}

@Bean
KafkaListenerContainerFactory<String, ArticlePublishedEvent> kafkaListenerContainerFactory(
        ConsumerFactory<String, ArticlePublishedEvent> consumerFactory
) {
    var factory = new ConcurrentKafkaListenerContainerFactory<String, ArticlePublishedEvent>();
    factory.setConsumerFactory(consumerFactory);
    return factory;
}
```

## 3. 设置测试环境

**为了设置我们的测试环境，我们可以利用Kafka [Testcontainer](https://www.baeldung.com/docker-test-containers)无缝启动Kafka Docker容器进行测试**：

```java
@Testcontainers
@SpringBootTest(classes = Application.class)
class DeserializationExceptionLiveTest {

    @Container
    private static KafkaContainer KAFKA = new KafkaContainer(DockerImageName.parse("confluentinc/cp-kafka:latest"));

    @DynamicPropertySource
    static void setProps(DynamicPropertyRegistry registry) {
        registry.add("spring.kafka.bootstrap-servers", KAFKA::getBootstrapServers);
    }

    // ...
}
```

**除此之外，我们还需要一个KafkaProducer和一个EmailService来验证监听器的功能，这些组件将向我们的监听器发送消息并验证其是否准确处理**。为了简化测试并避免Mock，我们将所有传入的文章保存在内存中的列表中，然后使用Getter访问它们：

```java
@Service
class EmailService {
    private final List<String> articles = new ArrayList<>();

    // logger, getter

    public void sendNewsletter(String article) {
        log.info("Sending newsletter for article: " + article);
        articles.add(article);
    }
}
```

因此，我们只需将EmailService注入到我们的测试类中，让我们继续创建testKafkaProducer：

```java
@Autowired
EmailService emailService;

static KafkaProducer<String, String> testKafkaProducer;

@BeforeAll
static void beforeAll() {
    Properties props = new Properties();
    props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, KAFKA.getBootstrapServers());
    props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
    props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
    testKafkaProducer = new KafkaProducer<>(props);
}
```

通过此设置，我们已经可以测试顺利流程了。让我们使用有效的JSON发布2篇文章，并验证我们的应用程序是否成功为每篇文章调用了emailService：

```java
@Test
void whenPublishingValidArticleEvent_thenProcessWithoutErrors() {
    publishArticle("{ \"article\": \"Kotlin for Java Developers\" }");
    publishArticle("{ \"article\": \"The S.O.L.I.D. Principles\" }");

    await().untilAsserted(() ->
            assertThat(emailService.getArticles())
                    .containsExactlyInAnyOrder(
                            "Kotlin for Java Developers",
                            "The S.O.L.I.D. Principles"
                    ));
}
```

## 4. 引发RecordDeserializationException

**如果配置的反序列化器无法正确解析消息的键或值，Kafka将抛出RecordDeserializationException。要重现此错误，我们只需发布一条包含无效JSON主体的消息**：

```java
@Test
void whenPublishingInvalidArticleEvent_thenCatchExceptionAndContinueProcessing() {
    publishArticle("{ \"article\": \"Introduction to Kafka\" }");
    publishArticle(" !! Invalid JSON !! ");
    publishArticle("{ \"article\": \"Kafka Streams Tutorial\" }");

    await().untilAsserted(() ->
            assertThat(emailService.getArticles())
                    .containsExactlyInAnyOrder(
                            "Kotlin for Java Developers",
                            "The S.O.L.I.D. Principles"
                    ));
}
```

如果我们运行此测试并检查控制台，我们会发现记录了一个重复的错误：

```text
ERROR 7716 --- [ntainer#0-0-C-1] o.s.k.l.KafkaMessageListenerContainer    : Consumer exception

**java.lang.IllegalStateException: This error handler cannot process 'SerializationException's directly; please consider configuring an 'ErrorHandlingDeserializer' in the value and/or key deserializer**
   at org.springframework.kafka.listener.DefaultErrorHandler.handleOtherException(DefaultErrorHandler.java:151) ~[spring-kafka-2.8.11.jar:2.8.11]
   at org.springframework.kafka.listener.KafkaMessageListenerContainer$ListenerConsumer.handleConsumerException(KafkaMessageListenerContainer.java:1815) ~[spring-kafka-2.8.11.jar:2.8.11]
   ...
**Caused by: org.apache.kafka.common.errors.RecordDeserializationException: Error deserializing key/value for partition tuyucheng.articles.published-0 at offset 1. If needed, please seek past the record to continue consumption.**
   at org.apache.kafka.clients.consumer.internals.Fetcher.parseRecord(Fetcher.java:1448) ~[kafka-clients-3.1.2.jar:na]
   ...
**Caused by: org.apache.kafka.common.errors.SerializationException: Can't deserialize data** [[32, 33, 33, 32, 73, 110, 118, 97, 108, 105, 100, 32, 74, 83, 79, 78, 32, 33, 33, 32]] from topic [tuyucheng.articles.published]
   at org.springframework.kafka.support.serializer.JsonDeserializer.deserialize(JsonDeserializer.java:588) ~[spring-kafka-2.8.11.jar:2.8.11]
   ...
**Caused by: com.fasterxml.jackson.core.JsonParseException: Unexpected character ('!' (code 33))**: expected a valid value (JSON String, Number, Array, Object or token 'null', 'true' or 'false')
   **at [Source: (byte[])" !! Invalid JSON !! "; line: 1, column: 3]**
   at com.fasterxml.jackson.core.JsonParser._constructError(JsonParser.java:2391) ~[jackson-core-2.13.5.jar:2.13.5]
   ...
```

然后，测试最终会超时并失败。如果我们检查断言错误，我们会注意到只有第一条消息被成功处理：

```text
org.awaitility.core.ConditionTimeoutException: Assertion condition 
Expecting actual:
  ["Introduction to Kafka"]
to contain exactly in any order:
  ["Introduction to Kafka", "Kafka Streams Tutorial"]
but could not find the following elements:
  ["Kafka Streams Tutorial"]
 within 5 seconds.
```

正如预期的那样，第二条消息的反序列化失败。因此，监听器继续尝试消费同一条消息，导致错误重复发生。

## 5. 创建错误处理程序

如果我们仔细分析故障日志，我们会注意到两个建议：

- 考虑配置一个“ErrorHandlingDeserializer”
- 如有需要，请查找过去的记录以继续消费

**换句话说，我们可以创建一个自定义错误处理程序来处理反序列化异常并增加消费者偏移量，这将使我们能够跳过无效消息并继续消费**。

### 5.1 实现CommonErrorHandler

为了实现CommonErrorHandler接口，我们必须重写2个没有默认实现的公共方法：

- handleOne()：调用来处理单个失败的记录；
- handleOtherException()：当抛出异常时调用，但不针对特定记录；

我们可以使用类似的方法来处理这两种情况，让我们首先捕获异常并记录错误消息：

```java
class KafkaErrorHandler implements CommonErrorHandler {

    @Override
    public void handleOne(Exception exception, ConsumerRecord<?, ?> record, Consumer<?, ?> consumer, MessageListenerContainer container) {
        handle(exception, consumer);
    }

    @Override
    public void handleOtherException(Exception exception, Consumer<?, ?> consumer, MessageListenerContainer container, boolean batchListener) {
        handle(exception, consumer);
    }

    private void handle(Exception exception, Consumer<?, ?> consumer) {
        log.error("Exception thrown", exception);
        // ...
    }
}
```

### 5.2 Kafka Consumer的seek()和commitSync()

**我们可以使用Consumer接口中的seek()方法手动更改主题中特定分区的当前偏移位置**，简而言之，我们可以使用它根据消息的偏移量根据需要重新处理或跳过消息。

在我们的例子中，如果异常是RecordDeserializationException的实例，我们将使用主题分区和下一个偏移量调用seek()方法：

```java
void handle(Exception exception, Consumer<?, ?> consumer) {
    log.error("Exception thrown", exception);
    if (exception instanceof RecordDeserializationException ex) {
        consumer.seek(ex.topicPartition(), ex.offset() + 1L);
        consumer.commitSync();
    } else {
        log.error("Exception not handled", exception);
    }
}
```

**我们可以注意到，我们需要从Consumer接口调用commitSync()，这将提交偏移量并确保Kafka代理确认并持久保存新位置**，此步骤至关重要，因为它会更新消费者组提交的偏移量，表明已成功处理调整位置之前的消息。

### 5.3 更新配置

**最后，我们需要将自定义错误处理程序添加到我们的消费者配置中**，让我们首先将其声明为@Bean：

```java
@Bean
CommonErrorHandler commonErrorHandler() {
    return new KafkaErrorHandler();
}
```

之后，我们将使用其专用的Setter将新的Bean添加到ConcurrentKafkaListenerContainerFactory：

```java
@Bean
ConcurrentKafkaListenerContainerFactory<String, ArticlePublishedEvent> kafkaListenerContainerFactory(
                ConsumerFactory<String, ArticlePublishedEvent> consumerFactory,
                CommonErrorHandler commonErrorHandler
        ) {
    var factory = new ConcurrentKafkaListenerContainerFactory<String, ArticlePublishedEvent>();
    factory.setConsumerFactory(consumerFactory);
    factory.setCommonErrorHandler(commonErrorHandler);
    return factory;
}
```

就这样，现在我们可以重新运行测试，并期望监听器跳过无效消息并继续消费消息。

## 6. 总结

在本文中，我们讨论了Spring Kafka的RecordDeserializationException，我们发现，如果处理不正确，它可能会阻塞给定分区的消费者组。

接下来，我们深入研究了Kafka的CommonErrorHandler接口并实现了它，以使我们的监听器能够在继续处理消息的同时处理反序列化失败。我们利用Consumer的API方法(即seek()和commitSync())通过相应地调整消费者偏移量来绕过无效消息。