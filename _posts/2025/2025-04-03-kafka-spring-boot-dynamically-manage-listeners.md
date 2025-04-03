---
layout: post
title:  在Spring Boot中动态管理Kafka监听器
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 概述

在当今的事件驱动架构中，有效地管理数据流至关重要。[Apache Kafka](https://www.baeldung.com/apache-kafka)是一种流行的选择，但尽管有[Spring Kafka](https://www.baeldung.com/spring-kafka)等辅助框架，将其集成到我们的应用程序中仍存在挑战。一个主要挑战是实现适当的动态监听器管理，它提供的灵活性和控制对于适应我们应用程序不断变化的工作负载和维护至关重要。

在本教程中，我们将学习**如何在Spring Boot应用程序中动态启动和停止Kafka监听器**。

## 2. 先决条件

首先，让我们将[spring-kafka](https://mvnrepository.com/artifact/org.springframework.kafka/spring-kafka)依赖导入到项目中：

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
    <version>3.1.2</version>
</dependency>
```

## 3. 配置Kafka消费者

生产者是向Kafka主题发布(写入)事件的应用程序。

在本教程中，我们将使用单元测试来模拟生产者向Kafka主题发送事件。订阅主题并处理事件流的消费者由我们应用程序内的监听器表示，此监听器配置为处理来自Kafka的传入消息。

让我们通过KafkaConsumerConfig类配置我们的Kafka消费者，其中包括Kafka代理的地址、消费者组ID以及键和值的反序列化器：

```java
@Bean
public DefaultKafkaConsumerFactory<String, UserEvent> consumerFactory() {
    Map<String, Object> props = new HashMap<>();
    props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
    props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
    props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, JsonDeserializer.class);
    props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
    props.put(JsonDeserializer.TRUSTED_PACKAGES, "com.tuyucheng.spring.kafka.start.stop.consumer");
    return new DefaultKafkaConsumerFactory<>(props, new StringDeserializer(), new JsonDeserializer<>(UserEvent.class));
}
```

## 4. 配置Kafka监听器

**在Spring Kafka中，使用@KafkaListener标注方法会创建一个监听器，该监听器会消费来自指定主题的消息**。为了定义它，让我们声明一个UserEventListener类：

```java
@KafkaListener(id = Constants.LISTENER_ID, topics = Constants.MULTI_PARTITION_TOPIC, groupId = "test-group",
        containerFactory = "kafkaListenerContainerFactory", autoStartup = "false")
public void processUserEvent(UserEvent userEvent) {
    logger.info("Received UserEvent: " + userEvent.getUserEventId());
    userEventStore.addUserEvent(userEvent);
}
```

上述监听器等待来自主题multi_partition_topic的消息，并使用processUserEvent()方法处理这些消息。我们将groupId指定为test-group，确保消费者成为更广泛组的一部分，从而促进跨多个实例的分布式处理。

我们使用id属性为每个监听器分配一个唯一标识符，在此示例中，分配的监听器ID为listener-id-1。

**autoStartup属性使我们能够控制在应用程序初始化时是否启动监听器**，在我们的示例中，我们将其设置为false，这意味着监听器不会在应用程序启动时自动启动。此配置为我们提供了手动启动监听器的灵活性。

这种手动启动可以由各种事件触发，例如新用户注册、应用程序内的特定条件(例如达到某个数据量阈值)或管理操作(例如通过管理界面手动启动监听器)。例如，如果在线零售应用程序在限时抢购期间检测到流量激增，它可以自动启动额外的监听器来处理增加的负载，从而优化性能。

UserEventStore充当监听器收到的事件的临时存储：

```java
@Component
public class UserEventStore {

    private final List<UserEvent> userEvents = new ArrayList<>();

    public void addUserEvent(UserEvent userEvent) {
        userEvents.add(userEvent);
    }

    public List<UserEvent> getUserEvents() {
        return userEvents;
    }

    public void clearUserEvents() {
        this.userEvents.clear();
    }
}
```

## 5. 动态控制监听器

让我们创建一个KafkaListenerControlService，使用KafkaListenerEndpointRegistry动态启动和停止Kafka监听器：

```java
@Service
public class KafkaListenerControlService {

    @Autowired
    private KafkaListenerEndpointRegistry registry;

    public void startListener(String listenerId) {
        MessageListenerContainer listenerContainer = registry.getListenerContainer(listenerId);
        if (listenerContainer != null && !listenerContainer.isRunning()) {
            listenerContainer.start();
        }
    }

    public void stopListener(String listenerId) {
        MessageListenerContainer listenerContainer = registry.getListenerContainer(listenerId);
        if (listenerContainer != null && listenerContainer.isRunning()) {
            listenerContainer.stop();
        }
    }
}
```

**KafkaListenerControlService可以根据分配的ID精确管理各个监听器实例**，startListener()和stopListener()方法都使用listenerId作为参数，使我们能够根据需要启动和停止从主题消费消息。

**KafkaListenerEndpointRegistry充当Spring应用程序上下文中定义的所有Kafka监听器端点的中央仓库**，它监视这些监听器容器，从而允许以编程方式控制它们的状态，无论是启动、停止还是暂停。对于需要实时调整消息处理活动而无需重新启动整个应用程序的应用程序来说，此功能至关重要。

## 6. 验证动态监听器组件

接下来，让我们重点测试Spring Boot应用程序中Kafka监听器的动态启动和停止功能。首先，让我们启动监听器：

```java
kafkaListenerControlService.startListener(Constants.LISTENER_ID);
```

然后我们通过发送并处理测试事件来验证监听器是否被激活：

```java
UserEvent startUserEventTest = new UserEvent(UUID.randomUUID().toString()); 
producer.send(new ProducerRecord<>(Constants.MULTI_PARTITION_TOPIC, startUserEventTest)); 
await().untilAsserted(() -> assertEquals(1, this.userEventStore.getUserEvents().size())); 
this.userEventStore.clearUserEvents();
```

现在监听器已处于激活状态，我们将发送一批10条消息进行处理。发送4条消息后，我们将停止监听器，然后将剩余的消息发送到Kafka主题：

```java
for (long count = 1; count <= 10; count++) {
    UserEvent userEvent = new UserEvent(UUID.randomUUID().toString());
    Future<RecordMetadata> future = producer.send(new ProducerRecord<>(Constants.MULTI_PARTITION_TOPIC, userEvent));
    RecordMetadata metadata = future.get();
    if (count == 4) {
        await().untilAsserted(() -> assertEquals(4, this.userEventStore.getUserEvents().size()));
        this.kafkaListenerControlService.stopListener(Constants.LISTENER_ID);
        this.userEventStore.clearUserEvents();
    }
    logger.info("User Event ID: " + userEvent.getUserEventId() + ", Partition : " + metadata.partition());
}
```

在启动监听器之前，我们将验证事件存储中没有消息：

```java
assertEquals(0, this.userEventStore.getUserEvents().size());
kafkaListenerControlService.startListener(Constants.LISTENER_ID);
await().untilAsserted(() -> assertEquals(6, this.userEventStore.getUserEvents().size()));
kafkaListenerControlService.stopListener(Constants.LISTENER_ID);
```

监听器重新启动后，它会处理我们在监听器停止后发送到Kafka主题的剩余6条消息。此测试展示了Spring Boot应用程序动态管理Kafka监听器的能力。

## 7. 用例

动态监听器管理在需要高适应性的场景中表现出色。例如，在高峰负载期间，我们可以动态启动更多监听器以提高吞吐量并减少处理时间。相反，在维护或低流量期间，我们可以停止监听器以节省资源。这种灵活性还有利于在功能标志后面部署新功能，从而实现无缝的动态调整而不会影响整个系统。

让我们考虑这样一个场景：一个电子商务平台引入了一个新的推荐引擎，旨在通过根据浏览历史和购买模式推荐产品来增强用户体验。为了在全面推出之前验证此功能的有效性，我们决定将其部署在功能标志后面。

激活此功能标志将启动Kafka监听器，当最终用户与平台交互时，由Kafka监听器支持的推荐引擎会处理传入的用户活动数据流，以生成个性化的产品推荐。

当我们停用该功能标志时，我们会停止Kafka监听器，平台会默认使用其现有的推荐引擎。无论新引擎处于何种测试阶段，这都能确保无缝的用户体验。

在此功能处于激活状态时，我们会积极收集数据、监控性能指标并对推荐引擎进行调整。我们会多次重复测试此功能，直到达到预期结果。

通过这个迭代过程，动态监听器管理被证明是一个有价值的工具，它允许无缝地引入新功能。

## 8. 总结

在本文中，我们讨论了Kafka与Spring Boot的集成，重点介绍了动态管理Kafka监听器，此功能对于管理波动的工作负载和执行日常维护至关重要。此外，它还支持功能切换、根据流量模式扩展服务以及使用特定触发器管理事件驱动的工作流。