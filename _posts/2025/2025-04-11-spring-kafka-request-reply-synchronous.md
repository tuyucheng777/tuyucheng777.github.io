---
layout: post
title:  使用ReplyingKafkaTemplate与Apache Kafka进行同步通信
category: kafka
copyright: kafka
excerpt: Apache Kafka
---

## 1. 概述

[Apache Kafka](https://www.baeldung.com/apache-kafka)已成为构建[事件驱动架构](https://www.baeldung.com/cs/eda-software-design)的最流行和广泛使用的消息传递系统之一，其中一个[微服务](https://www.baeldung.com/cs/microservices)向某个主题发布消息，另一个微服务则异步消费和处理该消息。

但是，有些情况下，发布者微服务需要立即响应才能继续进行进一步处理，虽然Kafka本质上是为异步通信而设计的，但它可以配置为通过单独的主题支持同步[请求-回复](https://www.enterpriseintegrationpatterns.com/patterns/messaging/RequestReply.html)通信。

**在本教程中，我们将探讨如何使用Apache Kafka在Spring Boot应用程序中实现同步请求-回复通信**。

## 2. 设置项目

为了演示，我们将模拟一个通知调度系统，我们将创建一个Spring Boot应用程序，该应用程序将同时充当生产者和消费者。

### 2.1 依赖

让我们首先将[Spring Kafka依赖](https://mvnrepository.com/artifact/org.springframework.kafka/spring-kafka)添加到项目的pom.xml文件中：

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
    <version>3.3.4</version>
</dependency>
```

该依赖为我们提供了建立连接和与[配置的Kafka实例](https://www.baeldung.com/ops/kafka-docker-setup)交互所需的类。

### 2.2 定义请求-回复消息

接下来，**让我们定义两个[记录](https://www.baeldung.com/java-record-keyword)来表示我们的请求和回复消息**：

```java
record NotificationDispatchRequest(String emailId, String content) {
}

public record NotificationDispatchResponse(UUID notificationId) {
}
```

这里，NotificationDispatchRequest记录保存通知的emailId和内容，而NotificationDispatchResponse记录包含处理请求后生成的唯一notificationId。

### 2.3 定义Kafka主题和配置属性

现在，**让我们定义请求和回复Kafka主题，此外，我们将配置从消费者组件接收回复的超时时间**。

我们将这些属性存储在项目的application.yaml文件中，并使用[@ConfigurationProperties](https://www.baeldung.com/configuration-properties-in-spring-boot)将值映射到Java记录，以便我们的配置和服务层可以引用：

```java
@Validated
@ConfigurationProperties(prefix = "cn.tuyucheng.taketoday.kafka.synchronous")
record SynchronousKafkaProperties(
                @NotBlank
                String requestTopic,

                @NotBlank
                String replyTopic,

                @NotNull @DurationMin(seconds = 10) @DurationMax(minutes = 2)
                Duration replyTimeout
        ) {
}
```

我们还添加了[校验注解](https://www.baeldung.com/java-validation)，以确保所有必需的属性都已正确配置，如果任何定义的校验失败，Spring [ApplicationContext](https://www.baeldung.com/spring-application-context)将无法启动，**这使我们能够遵循快速失败原则**。

下面是我们的application.yaml文件的片段，它定义了将自动映射到我们的SynchronousKafkaProperties记录的必需属性：

```yaml
cn:
    tuyucheng:
        taketoday:
            kafka:
                synchronous:
                    request-topic: notification-dispatch-request
                    reply-topic: notification-dispatch-response
                    reply-timeout: 30s
```

在这里，我们配置请求和回复Kafka主题名称以及30秒的回复超时。

除了自定义属性之外，我们还在application.yaml文件中添加一些核心Kafka配置属性：

```yaml
spring:
    kafka:
        bootstrap-servers: ${KAFKA_BOOTSTRAP_SERVERS}
        producer:
            key-serializer: org.apache.kafka.common.serialization.StringSerializer
            value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
        consumer:
            group-id: synchronous-kafka-group
            key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
            value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
            properties:
                spring:
                    json:
                        trusted:
                            packages: cn.tuyucheng.taketoday.kafka.synchronous
        properties:
            allow:
                auto:
                    create:
                        topics: true
```

首先，为了允许我们的应用程序连接到配置的Kafka实例，我们使用[环境变量](https://www.baeldung.com/spring-boot-properties-env-variables#use-environment-variable-in-applicationyml-file)配置其引导服务器URL。

接下来，我们为消费者和生产者配置键和值的序列化和反序列化属性。此外，对于我们的消费者，我们配置一个group-id并信任包含我们的请求-回复记录的包以进行JSON反序列化。

**配置上述属性后，Spring Kafka会自动为我们创建ConsumerFactory和ProducerFactory类型的Bean**，我们将在下一节中使用它们来定义其他Kafka配置Bean。

最后，我们启用主题自动创建功能，这样Kafka便会在主题不存在时自动创建主题。需要注意的是，我们仅为了演示而启用此属性— 在生产应用程序中不应执行此操作。

### 2.4 定义Kafka配置Bean

有了配置属性，让我们定义必要的Kafka配置Bean：

```java
@Bean
KafkaMessageListenerContainer<String, NotificationDispatchResponse> kafkaMessageListenerContainer(
                ConsumerFactory<String, NotificationDispatchResponse> consumerFactory
        ) {
    String replyTopic = synchronousKafkaProperties.replyTopic();
    ContainerProperties containerProperties = new ContainerProperties(replyTopic);
    return new KafkaMessageListenerContainer<>(consumerFactory, containerProperties);
}
```

首先，我们注入ConsumerFactory实例，并将其与配置的replyTopic一起使用以创建KafkaMessageListenerContainer Bean，**此Bean负责创建一个容器，用于轮询来自回复主题的消息**。

接下来，我们将定义在服务层中用于执行同步通信的核心Bean：

```java
@Bean
ReplyingKafkaTemplate<String, NotificationDispatchRequest, NotificationDispatchResponse> replyingKafkaTemplate(
    ProducerFactory<String, NotificationDispatchRequest> producerFactory,
    KafkaMessageListenerContainer<String, NotificationDispatchResponse> kafkaMessageListenerContainer
) {
    Duration replyTimeout = synchronousKafkaProperties.replyTimeout();
    var replyingKafkaTemplate = new ReplyingKafkaTemplate<>(producerFactory, kafkaMessageListenerContainer);
    replyingKafkaTemplate.setDefaultReplyTimeout(replyTimeout);
    return replyingKafkaTemplate;
}
```

使用ProducerFactory和先前定义的KafkaMessageListenerContainer Bean，我们创建了一个ReplyingKafkaTemplate Bean。此外，使用自动注入的synchronousKafkaProperties，我们配置了在application.yaml文件中定义的reply-timeout，这将决定我们的服务在超时之前等待响应的时间。

**这个ReplyingKafkaTemplate Bean管理请求和回复主题之间的交互，使得通过Kafka进行同步通信成为可能**。

最后，让我们定义Bean以使我们的监听器组件能够将响应发送回回复主题：

```java
@Bean
KafkaTemplate<String, NotificationDispatchResponse> kafkaTemplate(ProducerFactory<String, NotificationDispatchResponse> producerFactory) {
    return new KafkaTemplate<>(producerFactory);
}

@Bean
KafkaListenerContainerFactory<ConcurrentMessageListenerContainer<String, NotificationDispatchRequest>> kafkaListenerContainerFactory(
        ConsumerFactory<String, NotificationDispatchRequest> consumerFactory,
        KafkaTemplate<String, NotificationDispatchResponse> kafkaTemplate
) {
    var factory = new ConcurrentKafkaListenerContainerFactory<String, NotificationDispatchRequest>();
    factory.setConsumerFactory(consumerFactory);
    factory.setReplyTemplate(kafkaTemplate);
    return factory;
}
```

首先，我们使用ProducerFactory Bean创建一个标准的KafkaTemplate Bean。

然后，我们将其与ConsumerFactory Bean一起使用来定义KafkaListenerContainerFactory Bean，**此Bean使我们的监听器组件能够在完成所需处理后从请求主题消费消息，并将消息发送回回复主题**。

## 3. 使用Kafka实现同步通信

配置完成后，让我们在两个配置的Kafka主题之间实现同步请求-回复通信。

### 3.1 使用ReplyingKafkaTemplate发送和接收消息

首先，让我们创建一个NotificationDispatchService类，该类使用我们之前定义的ReplyingKafkaTemplate Bean将消息发送到配置的请求主题：

```java
@Service
@EnableConfigurationProperties(SynchronousKafkaProperties.class)
class NotificationDispatchService {

    private final SynchronousKafkaProperties synchronousKafkaProperties;
    private final ReplyingKafkaTemplate<String, NotificationDispatchRequest, NotificationDispatchResponse> replyingKafkaTemplate;

    // standard constructor

    NotificationDispatchResponse dispatch(NotificationDispatchRequest notificationDispatchRequest) {
        String requestTopic = synchronousKafkaProperties.requestTopic();
        ProducerRecord<String, NotificationDispatchRequest> producerRecord = new ProducerRecord<>(requestTopic, notificationDispatchRequest);

        var requestReplyFuture = replyingKafkaTemplate.sendAndReceive(producerRecord);
        return requestReplyFuture.get().value();
    }
}
```

在dispatch()方法中，我们使用自动装配的synchronousKafkaProperties实例来提取在application.yaml文件中配置的requestTopic。然后，我们将它与方法参数中传递的notificationDispatchRequest一起使用来创建ProducerRecord实例。

接下来，**我们将创建的ProducerRecord实例传递给sendAndReceive()方法，以将消息发布到请求主题**。该方法返回一个RequestReplyFuture对象，我们使用它等待响应，然后返回其值。

在底层，当我们调用sendAndReceive()方法时，**ReplyingKafkaTemplate类会生成一个唯一的关联ID(随机UUID)，并将其附加到传出消息的header中**。此外，它还会添加一个header，其中包含它期望响应返回的回复主题名称。请记住，我们已经在KafkaMessageListenerContainer Bean中配置了回复主题。

ReplyingKafkaTemplate Bean使用生成的关联ID作为键，将RequestReplyFuture对象存储在线程安全的[ConcurrentHashMap](https://www.baeldung.com/java-concurrent-map#concurrenthashmap)中，**这使得它即使在多线程环境中也能工作并支持并发请求**。

### 3.2 定义Kafka消息监听器

接下来，为了完成我们的实现，让我们创建一个监听器组件，用于监听配置的请求主题中的消息并将响应发送回回复主题：

```java
@Component
class NotificationDispatchListener {

    @SendTo
    @KafkaListener(topics = "${cn.tuyucheng.taketoday.kafka.synchronous.request-topic}")
    NotificationDispatchResponse listen(NotificationDispatchRequest notificationDispatchRequest) {
        // ... processing logic
        UUID notificationId = UUID.randomUUID();
        return new NotificationDispatchResponse(notificationId);
    }
}
```

我们使用@KafkaListener注解来监听我们在application.yaml文件中配置的请求主题。

在我们的listen()方法中，我们只需返回一个包含唯一notificationId的NotificationDispatchResponse记录。

重要的是，我们用@SendTo注解标注我们的方法，它指示Spring Kafka从消息头中提取关联ID和回复主题名称。**它使用它们自动将方法的返回值发送到提取的回复主题，并将相同的关联ID添加到消息头**。

这允许我们的NotificationDispatchService类中的ReplyingKafkaTemplate Bean使用它最初生成的关联ID获取正确的RequestReplyFuture对象。

## 4. 总结

在本文中，我们探讨了如何使用Apache Kafka实现Spring Boot应用程序中两个组件之间的同步通信。

我们完成了必要的配置并模拟了通知调度系统。

通过使用ReplyingKafkaTemplate，我们可以将Apache Kafka的异步特性转换为同步请求-回复模式；这种方法有点不合常规，因此在生产中实施之前，仔细评估它是否与项目的架构一致非常重要。