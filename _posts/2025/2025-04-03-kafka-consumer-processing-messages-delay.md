---
layout: post
title:  消费者延迟处理Kafka消息
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 概述

[Apache Kafka](https://www.baeldung.com/apache-kafka)是一个事件流平台，可大规模收集、处理、存储和集成数据。有时，我们可能希望延迟处理来自Kafka的消息。例如，客户订单处理系统旨在延迟X秒后处理订单，并在此时间范围内处理取消订单。

在本文中，我们将探讨使用[Spring Kafka](https://www.baeldung.com/spring-kafka)消费者延迟处理Kafka消息。尽管Kafka不提供对延迟消息消费的开箱即用支持，但我们将研究另一种实现方案。

## 2. 应用背景

**Kafka提供了多种错误重试方法，我们将使用此重试机制来延迟消费者对消息的处理。因此，有必要了解[Kafka重试](https://www.baeldung.com/spring-retry-kafka-consumer)的工作原理**。

让我们考虑一个订单处理应用程序，客户可以在UI上下订单，用户可以在10秒内取消错误下达的订单。这些订单将发送到Kafka主题web.orders，我们的应用程序会在那里处理它们。

外部服务公开最新的订单状态(CREATED、ORDER_CONFIRMED、ORDER_PROCESSED、DELETED)。我们的应用程序需要接收消息，等待10秒，并与外部服务核对订单是否处于CONFIRMED状态，即用户在10秒内未取消订单，以处理订单。

为了测试，从web.orders.internal收到的内部订单不应延迟。

让我们添加一个简单的Order模型，其中orderGeneratedDateTime由生产者填充，orderProcessedTime由消费者在延迟一段时间后填充：

```java
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class Order {

    private UUID orderId;

    private LocalDateTime orderGeneratedDateTime;

    private LocalDateTime orderProcessedTime;

    private List<String> address;

    private double price;
}
```

## 3. Kafka监听器和外部服务

接下来，**我们将添加一个用于主题消费的监听器和一个公开订单状态的服务**。

让我们添加一个[KafkaListener](https://docs.spring.io/spring-kafka/reference/kafka/receiving-messages/listener-annotation.html)，它读取并处理来自主题web.orders和web.internal.orders的消息：

```java
@RetryableTopic(attempts = "1", include = KafkaBackoffException.class, dltStrategy = DltStrategy.NO_DLT)
@KafkaListener(topics = { "web.orders", "web.internal.orders" }, groupId = "orders")
public void handleOrders(String order) throws JsonProcessingException {
    Order orderDetails = objectMapper.readValue(order, Order.class);
    OrderService.Status orderStatus = orderService.findStatusById(orderDetails.getOrderId());
    if (orderStatus.equals(OrderService.Status.ORDER_CONFIRMED)) {
        orderService.processOrder(orderDetails);
    }
}
```

包含KafkaBackoffException很重要，这样监听器才允许重试。为简单起见，我们假设外部OrderService始终将订单状态返回为CONFIRMED。此外，processOrder()方法将订单处理时间设置为当前时间，并将订单保存到HashMap中：

```java
@Service
public class OrderService {

    HashMap<UUID, Order> orders = new HashMap<>();

    public Status findStatusById(UUID orderId) {
        return Status.ORDER_CONFIRMED;
    }

    public void processOrder(Order order) {
        order.setOrderProcessedTime(LocalDateTime.now());
        orders.put(order.getOrderId(), order);
    }
}
```

## 4. 自定义延迟消息监听器

Spring Kafka推出了[KafkaBackoffAwareMessageListenerAdapter](https://docs.spring.io/spring-kafka/docs/current/api/org/springframework/kafka/listener/adapter/KafkaBackoffAwareMessageListenerAdapter.html)，它扩展了AbstractAdaptableMessageListener并实现了AcknowledgingConsumerAwareMessageListener。**此适配器检查backoff dueTimestamp标头，并通过调用KafkaConsumerBackoffManager来取消消息或重试处理**。

现在让我们实现类似于KafkaBackoffAwareMessageListenerAdapter的DelayedMessageListenerAdapter，此适配器应提供灵活性来配置每个主题的延迟以及默认延迟0秒：

```java
public class DelayedMessageListenerAdapter<K, V> extends AbstractDelegatingMessageListenerAdapter<MessageListener<K, V>>
        implements AcknowledgingConsumerAwareMessageListener<K, V> {

    // Field declaration and constructor

    public void setDelayForTopic(String topic, Duration delay) {
        Objects.requireNonNull(topic, "Topic cannot be null");
        Objects.requireNonNull(delay, "Delay cannot be null");
        this.logger.debug(() -> String.format("Setting delay %s for listener id %s", delay, this.listenerId));
        this.delaysPerTopic.put(topic, delay);
    }

    public void setDefaultDelay(Duration delay) {
        Objects.requireNonNull(delay, "Delay cannot be null");
        this.logger.debug(() -> String.format("Setting delay %s for listener id %s", delay, this.listenerId));
        this.defaultDelay = delay;
    }

    @Override
    public void onMessage(ConsumerRecord<K, V> consumerRecord, Acknowledgment acknowledgment, Consumer<?, ?> consumer) throws KafkaBackoffException {
        this.kafkaConsumerBackoffManager.backOffIfNecessary(createContext(consumerRecord,
                consumerRecord.timestamp() + delaysPerTopic.getOrDefault(consumerRecord.topic(), this.defaultDelay)
                        .toMillis(), consumer));
        invokeDelegateOnMessage(consumerRecord, acknowledgment, consumer);
    }

    private KafkaConsumerBackoffManager.Context createContext(ConsumerRecord<K, V> data, long nextExecutionTimestamp, Consumer<?, ?> consumer) {
        return this.kafkaConsumerBackoffManager.createContext(nextExecutionTimestamp,
                this.listenerId,
                new TopicPartition(data.topic(), data.partition()), consumer);
    }
}
```

对于每条传入的消息，此适配器首先接收记录并检查主题的延迟设置。这将在配置中设置，如果未设置，则使用默认延迟。

KafkaConsumerBackoffManager#backOffIfNecessary方法的现有实现会检查上下文记录时间戳与当前时间戳之间的差异，如果差异为正，则表示无需消费，分区将暂停并引发[KafkaBackoffException](https://docs.spring.io/spring-kafka/api/org/springframework/kafka/listener/KafkaBackoffException.html)。否则，它会将记录发送到KafkaListener方法进行消费。

## 5. 监听器配置

**[ConcurrentKafkaListenerContainerFactory](https://docs.spring.io/spring-kafka/api/org/springframework/kafka/config/ConcurrentKafkaListenerContainerFactory.html)是Spring Kafka的默认实现，负责为KafkaListener构建容器**。它允许我们配置并发KafkaListener实例的数量，每个容器都可以看作是一个逻辑线程池，其中每个线程负责监听来自一个或多个Kafka主题的消息。

DelayedMessageListenerAdapter需要通过声明自定义ConcurrentKafkaListenerContainerFactory来配置监听器，我们可以为特定主题(如web.orders)设置延迟，也可以为任何其他主题设置默认延迟0：

```java
@Bean
public ConcurrentKafkaListenerContainerFactory<Object, Object> kafkaListenerContainerFactory(ConsumerFactory<Object, Object> consumerFactory,
                                                                                             ListenerContainerRegistry registry, TaskScheduler scheduler) {
    ConcurrentKafkaListenerContainerFactory<Object, Object> factory = new ConcurrentKafkaListenerContainerFactory<>();
    factory.setConsumerFactory(consumerFactory);
    KafkaConsumerBackoffManager backOffManager = createBackOffManager(registry, scheduler);
    factory.getContainerProperties()
            .setAckMode(ContainerProperties.AckMode.RECORD);
    factory.setContainerCustomizer(container -> {
        DelayedMessageListenerAdapter<Object, Object> delayedAdapter = wrapWithDelayedMessageListenerAdapter(backOffManager, container);
        delayedAdapter.setDelayForTopic("web.orders", Duration.ofSeconds(10));
        delayedAdapter.setDefaultDelay(Duration.ZERO);
        container.setupMessageListener(delayedAdapter);
    });
    return factory;
}

@SuppressWarnings("unchecked")
private DelayedMessageListenerAdapter<Object, Object> wrapWithDelayedMessageListenerAdapter(KafkaConsumerBackoffManager backOffManager,
                                                                                            ConcurrentMessageListenerContainer<Object, Object> container) {
    return new DelayedMessageListenerAdapter<>((MessageListener<Object, Object>) container.getContainerProperties()
            .getMessageListener(), backOffManager, container.getListenerId());
}

private ContainerPartitionPausingBackOffManager createBackOffManager(ListenerContainerRegistry registry, TaskScheduler scheduler) {
    return new ContainerPartitionPausingBackOffManager(registry,
            new ContainerPausingBackOffHandler(new ListenerContainerPauseService(registry, scheduler)));
}
```

值得注意的是，在RECORD级别设置确认模式对于确保消费者在处理过程中发生错误时重新传递消息至关重要。

最后，我们需要定义一个TaskScheduler Bean来在延迟时间之后恢复暂停的分区，并且这个调度程序需要注入到BackOffManager中，它将被DelayedMessageListenerAdapter使用：

```java
@Bean
public TaskScheduler taskScheduler() {
    return new ThreadPoolTaskScheduler();
}
```

## 6. 测试

让我们确保web.orders主题上的订单在经过测试处理之前经历10秒的延迟：

```java
@Test
void givenKafkaBrokerExists_whenCreateOrderIsReceived_thenMessageShouldBeDelayed() throws Exception {
    // Given
    var orderId = UUID.randomUUID();
    Order order = Order.builder()
            .orderId(orderId)
            .price(1.0)
            .orderGeneratedDateTime(LocalDateTime.now())
            .address(List.of("41 Felix Avenue, Luton"))
            .build();

    String orderString = objectMapper.writeValueAsString(order);
    ProducerRecord<String, String> record = new ProducerRecord<>("web.orders", orderString);

    // When
    testKafkaProducer.send(record)
            .get();
    await().atMost(Duration.ofSeconds(1800))
            .until(() -> {
                // then
                Map<UUID, Order> orders = orderService.getOrders();
                return orders != null && orders.get(orderId) != null && Duration.between(orders.get(orderId)
                                .getOrderGeneratedDateTime(), orders.get(orderId)
                                .getOrderProcessedTime())
                        .getSeconds() >= 10;
            });
}
```

接下来，我们将测试任何发送到web.internal.orders的订单是否遵循默认的0秒延迟：

```java
@Test
void givenKafkaBrokerExists_whenCreateOrderIsReceivedForOtherTopics_thenMessageShouldNotBeDelayed() throws Exception {
    // Given
    var orderId = UUID.randomUUID();
    Order order = Order.builder()
            .orderId(orderId)
            .price(1.0)
            .orderGeneratedDateTime(LocalDateTime.now())
            .address(List.of("41 Felix Avenue, Luton"))
            .build();

    String orderString = objectMapper.writeValueAsString(order);
    ProducerRecord<String, String> record = new ProducerRecord<>("web.internal.orders", orderString);

    // When
    testKafkaProducer.send(record)
            .get();
    await().atMost(Duration.ofSeconds(1800))
            .until(() -> {
                // Then
                Map<UUID, Order> orders = orderService.getOrders();
                System.out.println("Time...." + Duration.between(orders.get(orderId)
                                .getOrderGeneratedDateTime(), orders.get(orderId)
                                .getOrderProcessedTime())
                        .getSeconds());
                return orders != null && orders.get(orderId) != null && Duration.between(orders.get(orderId)
                                .getOrderGeneratedDateTime(), orders.get(orderId)
                                .getOrderProcessedTime())
                        .getSeconds() <= 1;
            });
}
```

## 7. 总结

在本教程中，我们探讨了Kafka消费者如何按固定间隔延迟处理消息。

我们可以通过利用嵌入的消息持续时间作为消息的一部分来修改实现以动态设置处理延迟。