---
layout: post
title:  使用Reactor Kafka创建Kafka消费者
category: springreactive
copyright: springreactive
excerpt: Spring Reactive
---

## 1. 简介

Apache Kafka是一个流行的分布式事件流平台，与[Project Reactor](https://www.baeldung.com/reactor-core)结合使用，可以构建具有弹性和响应式的应用程序。[Reactor Kafka](https://www.baeldung.com/java-spring-webflux-reactive-kafka)是一个基于Reactor和[Kafka](https://www.baeldung.com/apache-kafka)生产者/消费者API构建的响应式API。

Reactor Kafka API使我们能够使用支持背压的函数式非阻塞API向Kafka发布消息并从Kafka消费消息，这意味着系统可以根据需求和资源可用性动态调整消息处理速率，确保高效且容错的运行。

在本教程中，我们将探索如何使用[Reactor Kafka](https://projectreactor.io/docs/kafka/release/reference/)创建Kafka消费者，并确保容错性和可靠性。我们将深入探讨背压、重试和错误处理等关键概念，以及如何以非阻塞方式异步处理消息。

## 2. 设置项目

首先，**我们应该在项目中包含[Spring Kafka](https://mvnrepository.com/artifact/org.springframework.kafka/spring-kafka)和[Reactor Kafka](https://mvnrepository.com/artifact/io.projectreactor.kafka/reactor-kafka) Maven依赖**：

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
<dependency>
    <groupId>io.projectreactor.kafka</groupId>
    <artifactId>reactor-kafka</artifactId>
</dependency>
```

## 3. 响应式Kafka消费者设置

接下来，我们将使用Reactor Kafka设置Kafka消费者。首先，我们将配置必要的消费者属性，确保其已正确设置以连接到Kafka。然后，我们将初始化消费者，最后，我们将了解如何以响应式方式消费消息。

### 3.1 配置Kafka消费者属性

现在，让我们配置Reactive Kafka消费者属性，**KafkaConfig配置类定义了消费者要使用的属性**：

```java
public class KafkaConfig {

    @Value("${spring.kafka.bootstrap-servers}")
    private String bootstrapServers;

    public static Map<String, Object> consumerConfig() {
        Map<String, Object> config = new HashMap<>();
        config.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        config.put(ConsumerConfig.GROUP_ID_CONFIG, "reactive-consumer-group");
        config.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
        config.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        config.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        return config;
    }
}
```

ConsumerConfig.GROUP_ID_CONFIG定义了消费者组，它能够在各个消费者之间实现消息负载均衡，同一组中的所有消费者负责处理来自同一主题的消息。

接下来我们在实例化ReactiveKafkaConsumerTemplate时使用配置类来消费事件：

```java
public ReactiveKafkaConsumerTemplate<String, String> reactiveKafkaConsumerTemplate() {
    return new ReactiveKafkaConsumerTemplate<>(receiverOptions());
}

private ReceiverOptions<String, String> receiverOptions() {
    Map<String, Object> consumerConfig = consumerConfig();
    ReceiverOptions<String, String> receiverOptions = ReceiverOptions.create(consumerConfig);
        return receiverOptions.subscription(Collections.singletonList("test-topic"));
}
```

receiverOptions()方法使用consumerConfig()中的设置来配置Kafka消费者，并订阅test-topic，以确保其监听消息。reactiveKafkaConsumerTemplate()方法初始化一个ReactiveKafkaConsumerTemplate，为我们的响应式应用程序启用非阻塞、背压感知的消息消费功能。

### 3.2 使用Reactive Kafka创建Kafka消费者

**在Reactor Kafka中，Kafka Consumer的抽象选择是一个入站Flux，所有从Kafka接收的事件都由框架发布**。此Flux是通过调用ReactiveKafkaConsumerTemplate上的receive()、receiveAtmostOnce()、receiveAutoAck()或receiveExactlyOnce()方法之一创建的。

在这个例子中，我们使用receive()运算符来消费入站Flux：

```java
public class ConsumerService {

    private final ReactiveKafkaConsumerTemplate<String, String> reactiveKafkaConsumerTemplate;

    public Flux<String> consumeRecord() {
        return reactiveKafkaConsumerTemplate.receive()
            .map(ReceiverRecord::value)
            .doOnNext(msg -> log.info("Received: {}", msg))
            .doOnError(error -> log.error("Consumer error: {}", error.getMessage()));
    }
}
```

这种方法允许系统在消息到达时以响应式方式处理，而不会阻塞或丢失消息。**通过使用响应式流，消费者可以按照自己的节奏扩展和处理消息，并在必要时施加背压**。在这里，我们通过doOnNext()记录收到的每条消息，并使用doOnError()记录错误。

## 4. 处理背压

使用Reactor Kafka消费者的主要优势之一是它支持背压，**这确保系统不会因高吞吐量而崩溃**。我们可以使用limitRate()限制处理速率，或使用buffer()进行批处理，而不是直接消费消息：

```java
public Flux<String> consumeWithLimit() {
    return reactiveKafkaConsumerTemplate.receive()
        .limitRate(2)
        .map(ReceiverRecord::value);
}
```

这里我们一次最多请求2条消息，以控制流量，这种方法确保了高效且可感知背压的消息处理。最后，它仅提取并返回消息值。

我们不需要单独处理它们，而是可以通过缓冲固定数量的记录，然后将它们作为一个组发出，从而批量消费它们：

```java
public Flux<String> consumeAsABatch() {
    return reactiveKafkaConsumerTemplate.receive()
        .buffer(2)
        .flatMap(messages -> Flux.fromStream(messages.stream()
            .map(ReceiverRecord::value)));
}
```

这里，我们最多缓冲两条记录，然后将它们作为批次发送。通过使用buffer(2)，它将消息分组并一起处理，从而减少了单独处理的开销。

## 5. 错误处理策略

**在响应式Kafka消费者中，管道中的错误充当终止信号，这会导致消费者关闭，从而使服务实例继续运行而不消费事件**。Reactor Kafka提供了多种策略来解决这个问题，例如使用retryWhen运算符的重试机制，该机制会捕获故障，重新订阅上游发布者，并重新创建Kafka消费者。

Kafka消费者的另一个常见问题是反序列化错误，当消费者由于格式异常而无法反序列化消息时，就会发生这种情况。为了处理所谓的错误，**我们可以使用Spring Kafka提供的ErrorHandlingDeserializer**。

### 5.1 重试策略

当我们想要重试失败的操作时，重试策略至关重要，**该策略确保以固定的延迟(例如5秒)持续重试，直到消费者成功重新连接或满足预定义的退出条件**。

让我们为消费者实现一个重试策略，以便当发生错误时它可以自动重试消息处理：

```java
public Flux<String> consumeWithRetryWithBackOff(AtomicInteger attempts) {
    return reactiveKafkaConsumerTemplate.receive()
        .flatMap(msg -> attempts.incrementAndGet() < 3 ? Flux.error(new RuntimeException("Failure")) : Flux.just(msg))
        .retryWhen(Retry.fixedDelay(3, Duration.ofSeconds(1)))
        .map(ReceiverRecord::value);
}
```

在此示例中，Retry.backoff(3, Duration.ofSeconds(1))指定系统尝试重试最多3次，退避时间为1秒。

### 5.2 使用ErrorHandlingDeserializer处理序列化错误

从Kafka消费消息时，如果消息格式与预期的模式不匹配，就会遇到反序列化错误。为了解决这个问题，**我们可以使用Spring Kafka的ErrorHandlingDeserializer**，它通过捕获反序列化错误来防止消费者失败。然后，它会将错误详细信息作为header添加到ReceiverRecord，而不是丢弃消息或抛出异常：

```java
private Map<String, Object> errorHandlingConsumerConfig(){
    Map<String, Object> config = new HashMap<>();
    config.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, ErrorHandlingDeserializer.class);
    config.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, ErrorHandlingDeserializer.class);
    config.put(ErrorHandlingDeserializer.KEY_DESERIALIZER_CLASS, StringDeserializer.class);
    config.put(ErrorHandlingDeserializer.VALUE_DESERIALIZER_CLASS, StringDeserializer.class);
    return config;
}
```

## 6. 总结

在本文中，我们探讨了如何使用Reactor Kafka创建Kafka消费者，重点介绍了错误处理、重试和背压管理，这些技术使我们的Kafka消费者即使在故障情况下也能保持容错和高效。