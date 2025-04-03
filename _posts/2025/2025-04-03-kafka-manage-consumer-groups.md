---
layout: post
title:  管理Kafka消费者组
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 简介

消费者组允许多个消费者读取同一主题，从而帮助创建更具可扩展性的[Kafka](https://www.baeldung.com/apache-kafka)应用程序。

在本教程中，我们将了解消费者组以及他们如何在消费者之间重新平衡分区。

## 2. 什么是消费者组？

消费者组是与一个或多个主题相关联的一组唯一消费者，每个消费者可以从零个、一个或多个分区读取数据。此外，每个分区在给定时间只能分配给一个消费者。分区分配会随着组成员的变化而变化，这称为组重新平衡。

消费者组是Kafka应用程序的重要组成部分，它允许对相似的消费者进行分组，并使他们能够从[分区主题](https://www.baeldung.com/kafka-topics-partitions)中并行读取数据。因此，它提高了Kafka应用程序的性能和可扩展性。

### 2.1 组协调器和组组长

当我们实例化一个消费者组时，Kafka还会创建组协调器。组协调器定期接收来自消费者的请求，称为心跳。如果消费者停止发送心跳，协调器会认为该消费者已离开组或崩溃。这是分区重新平衡的一个可能触发因素。

第一个请求组协调器加入组的消费者将成为组长，当由于任何原因发生重新平衡时，组长会从组协调器收到组成员列表。然后，**组长使用在[partition.assignment.strategy](https://docs.confluent.io/platform/current/installation/configuration/consumer-configs.html#partition-assignment-strategy)配置中设置的可自定义策略在该列表中的消费者之间重新分配分区**。

### 2.2 提交偏移量

Kafka使用已提交偏移量来跟踪从主题读取的最后一个位置，**已提交偏移量是消费者确认已成功处理的主题中的位置**。换句话说，它是其自身和其他消费者在后续轮次中读取事件的起点。

Kafka将所有分区的已提交偏移量存储在名为__consumer_offsets的内部主题中，我们可以放心地信任其信息，因为主题对于代理来说是持久且容错的。

### 2.3 分区重新平衡

分区重新平衡会将分区所有权从一个消费者更改为另一个消费者，当新消费者加入组或组中的消费者成员崩溃或取消订阅时，Kafka会自动执行重新平衡。

为了提高可扩展性，当新消费者加入组时，Kafka会与新添加的消费者公平共享其他消费者的分区。此外，**当消费者崩溃时，必须将其分区分配给组中的其余消费者**，以避免丢失任何未处理的消息。

分区重新平衡使用__consumer_offsets主题让消费者从正确的位置开始读取重新分配的分区。

在重新平衡期间，消费者无法消费消息。换句话说，代理在重新平衡完成之前将不可用。此外，消费者会丢失其状态并需要重新计算其缓存值。分区重新平衡期间的不可用和缓存重新计算会使事件消费速度变慢。

## 3. 设置应用程序

在本节中，我们将配置基础知识以启动并运行[Spring Kafka](https://www.baeldung.com/spring-kafka)应用程序。

### 3.1 创建基本配置

首先，让我们配置主题及其分区：

```java
@Configuration
public class KafkaTopicConfiguration {

    @Value(value = "${spring.kafka.bootstrap-servers}")
    private String bootstrapAddress;

    public KafkaAdmin kafkaAdmin() {
        Map<String, Object> configs = new HashMap<>();
        configs.put(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapAddress);
        return new KafkaAdmin(configs);
    }

    public NewTopic celciusTopic() {
        return TopicBuilder.name("topic-1")
                .partitions(2)
                .build();
    }
}
```

上面的配置很简单，我们只是配置了一个名为topic-1的新主题，其中包含两个分区。

现在，让我们配置生产者：

```java
@Configuration
public class KafkaProducerConfiguration {

    @Value(value = "${spring.kafka.bootstrap-servers}")
    private String bootstrapAddress;

    @Bean
    public ProducerFactory<String, Double> kafkaProducer() {
        Map<String, Object> configProps = new HashMap<>();
        configProps.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapAddress);
        configProps.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        configProps.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, DoubleSerializer.class);
        return new DefaultKafkaProducerFactory<>(configProps);
    }

    @Bean
    public KafkaTemplate<String, Double> kafkaProducerTemplate() {
        return new KafkaTemplate<>(kafkaProducer());
    }
}
```

在上面的Kafka生产者配置中，我们设置了代理地址和他们用于写入消息的[序列化器](https://www.baeldung.com/kafka-custom-serializer)。

最后，让我们配置消费者：

```java
@Configuration
public class KafkaConsumerConfiguration {

    @Value(value = "${spring.kafka.bootstrap-servers}")
    private String bootstrapAddress;

    @Bean
    public ConsumerFactory<String, Double> kafkaConsumer() {
        Map<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapAddress);
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, DoubleDeserializer.class);
        return new DefaultKafkaConsumerFactory<>(props);
    }

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, Double> kafkaConsumerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, Double> factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(kafkaConsumer());
        return factory;
    }
}
```

### 3.2 设置消费者

在我们的演示应用程序中，我们将从属于同一组(名为group-1)和topic-1的两个消费者开始：

```java
@Service
public class MessageConsumerService {
    @KafkaListener(topics = "topic-1", groupId = "group-1")
    public void consumer0(ConsumerRecord<?, ?> consumerRecord) {
        trackConsumedPartitions("consumer-0", consumerRecord);
    }

    @KafkaListener(topics = "topic-1", groupId = "group-1")
    public void consumer1(ConsumerRecord<?, ?> consumerRecord) {
        trackConsumedPartitions("consumer-1", consumerRecord);
    }
}
```

MessageConsumerService类使用@KafkaListener注解注册了两个消费者来监听group-1中的topic-1。

现在，我们还在MessageConsumerService类中定义一个字段和一个方法来跟踪已消费的分区：

```java
Map<String, Set<Integer>> consumedPartitions = new ConcurrentHashMap<>();

private void trackConsumedPartitions(String key, ConsumerRecord<?, ?> record) {
    consumedPartitions.computeIfAbsent(key, k -> new HashSet<>());
    consumedPartitions.computeIfPresent(key, (k, v) -> {
        v.add(record.partition());
        return v;
    });
}
```

在上面的代码中，我们使用[ConcurrentHashMap](https://www.baeldung.com/concurrenthashmap-reading-and-writing)将每个消费者名称映射到该消费者消费的所有分区的[HashSet](https://www.baeldung.com/java-hashset)。

## 4. 当消费者离开时可视化分区重新平衡

现在我们已经设置了所有配置并注册了消费者，我们可以直观地看到当其中一个消费者离开group-1时Kafka会做什么。为此，让我们定义使用嵌入式代理的[Kafka集成测试](https://www.baeldung.com/spring-boot-kafka-testing)的骨架：

```java
@SpringBootTest(classes = ManagingConsumerGroupsApplicationKafkaApp.class)
@EmbeddedKafka(partitions = 2, brokerProperties = {"listeners=PLAINTEXT://localhost:9092", "port=9092"})
public class ManagingConsumerGroupsIntegrationTest {

    private static final String CONSUMER_1_IDENTIFIER = "org.springframework.kafka.KafkaListenerEndpointContainer#1";
    private static final int TOTAL_PRODUCED_MESSAGES = 50000;
    private static final int MESSAGE_WHERE_CONSUMER_1_LEAVES_GROUP = 10000;

    @Autowired
    KafkaTemplate<String, Double> kafkaTemplate;

    @Autowired
    KafkaListenerEndpointRegistry kafkaListenerEndpointRegistry;

    @Autowired
    MessageConsumerService consumerService;
}
```

在上面的代码中，我们注入了生产和消费消息所需的Bean：kafkaTemplate和consumerService。我们还注入了kafkaListenerEndpointRegistry来操作已注册的消费者。

最后，我们定义了3个将在测试用例中使用的常量。

现在，我们来定义测试用例方法：

```java
@Test
public void givenContinuousMessageFlow_whenAConsumerLeavesTheGroup_thenKafkaTriggersPartitionRebalance() throws InterruptedException {
    int currentMessage = 0;

    do {
        kafkaTemplate.send("topic-1", RandomUtils.nextDouble(10.0, 20.0));
        Thread.sleep(0,100);
        currentMessage++;

        if (currentMessage == MESSAGE_WHERE_CONSUMER_1_LEAVES_GROUP) {
            String containerId = kafkaListenerEndpointRegistry.getListenerContainerIds()
                    .stream()
                    .filter(a -> a.equals(CONSUMER_1_IDENTIFIER))
                    .findFirst()
                    .orElse("");
            MessageListenerContainer container = kafkaListenerEndpointRegistry.getListenerContainer(containerId);
            Objects.requireNonNull(container).stop();
            kafkaListenerEndpointRegistry.unregisterListenerContainer(containerId);
            if(currentMessage % 1000 == 0){
                log.info("Processed {} of {}", currentMessage, TOTAL_PRODUCED_MESSAGES);
            }
        }
    } while (currentMessage != TOTAL_PRODUCED_MESSAGES);

    assertEquals(1, consumerService.consumedPartitions.get("consumer-1").size());
    assertEquals(2, consumerService.consumedPartitions.get("consumer-0").size());
}
```

在上面的测试中，我们创建了一个消息流，在某个时候，我们删除了其中一个消费者，因此Kafka会将其分区重新分配给剩余的消费者。让我们分解一下逻辑，使其更加透明：

1. 主循环使用kafkaTemplate和Apache Commons的[RandomUtils](https://commons.apache.org/proper/commons-lang/javadocs/api-3.9/org/apache/commons/lang3/RandomUtils.html)生成50000个随机数事件，当生成任意数量的消息时(本例中为10000条)，我们会停止并从代理中取消注册一个消费者。
2. 要取消注册消费者，我们首先使用流在容器中搜索匹配的消费者，并使用getListenerContainer()方法检索它。然后，我们调用stop()来停止容器Spring组件的执行。最后，**我们调用unregisterListenerContainer()以编程方式从KafkaBroker取消注册与容器变量关联的监听器**。

在讨论测试断言之前，让我们先看一下Kafka在测试执行期间生成的几行日志。

首先要看到的是consumer-1向组协调器发出的LeaveGroup请求：

```text
INFO o.a.k.c.c.i.ConsumerCoordinator - [Consumer clientId=consumer-group-1-1, groupId=group-1] Member consumer-group-1-1-4eb63bbc-336d-44d6-9d41-0a862029ba95 sending LeaveGroup request to coordinator localhost:9092
```

然后，组协调器会自动触发重新平衡并显示背后的原因：

```text
INFO  k.coordinator.group.GroupCoordinator - [GroupCoordinator 0]: Preparing to rebalance group group-1 in state PreparingRebalance with old generation 2 (__consumer_offsets-4) (reason: Removing member consumer-group-1-1-4eb63bbc-336d-44d6-9d41-0a862029ba95 on LeaveGroup)
```

回到我们的测试，我们将断言分区重新平衡正确发生。**由于我们取消注册了以1结尾的消费者，因此其分区应重新分配给剩余的消费者**，即consumer-0。因此，我们使用跟踪的消费记录图来检查consumer-1仅从一个分区消费，而consumer-0从两个分区消费。

## 5. 有用的消费者配置

现在，让我们讨论一些影响分区重新平衡的消费者配置以及为它们设置特定值的权衡。

### 5.1 会话超时和心跳频率

session.timeout.ms参数表示组协调器在触发分区重新平衡之前可以等待消费者发送心跳的最长时间(以毫秒为单位)。除session.timeout.ms外，heartbeat.interval.ms还表示消费者向组协调器发送心跳的频率(以毫秒为单位)。

**我们应该同时修改消费者超时和心跳频率，以便[heartbeat.interval.ms](https://docs.confluent.io/platform/current/installation/configuration/consumer-configs.html#heartbeat-interval-ms)始终低于[session.timeout.ms](https://docs.confluent.io/platform/current/installation/configuration/consumer-configs.html#heartbeat-interval-ms)**。这是因为我们不想让消费者在发送心跳之前因超时而死亡。通常，我们将心跳间隔设置为会话超时的33%，以保证在消费者死亡之前发送多个心跳。

默认的消费者会话超时设置为45秒，我们可以修改该值，只要我们了解修改它的利弊即可。

**当我们将会话超时设置为低于默认值时，我们可以提高消费者组从故障中恢复的速度，从而提高组可用性**。但是，在0.10.1.0之前的Kafka版本中，如果消费者的主线程在消费时间超过会话超时的消息时被阻塞，则消费者无法发送心跳。因此，消费者被视为死亡，组协调器将触发不必要的分区重新平衡。此问题已在[KIP-62](https://cwiki.apache.org/confluence/display/KAFKA/KIP-62%3A+Allow+consumer+to+send+heartbeats+from+a+background+thread)中修复，引入了仅发送心跳的后台线程。

如果我们为会话超时设置更高的值，我们将无法更快地检测故障。但是，这可能会解决上述Kafka版本早于0.10.1.0时出现的不必要的分区重新平衡问题。

### 5.2 最大轮询间隔时间

**另一个配置是max.poll.interval.ms，表示代理等待空闲消费者的最大时间**。超过该时间后，消费者将停止发送心跳，直到达到配置的会话超时并离开组。max.poll.interval.ms的[默认等待时间](https://docs.confluent.io/platform/current/installation/configuration/consumer-configs.html#max-poll-interval-ms)为5分钟。

如果我们为max.poll.interval.ms设置更高的值，我们就会为消费者提供更多空闲空间，这可能有助于避免重新平衡。但是，如果没有消息可供消费，增加该时间也可能会增加空闲消费者的数量。这在低吞吐量环境中可能是一个问题，因为消费者可能保持空闲更长时间，从而增加基础设施成本。

## 6. 总结

在本文中，我们了解了组长和组协调器角色的基础知识，我们还研究了Kafka如何管理消费者组和分区。

我们在实践中看到，当一个消费者离开组时，Kafka会自动重新平衡组内的分区。

了解Kafka何时触发分区重新平衡并相应地调整消费者配置至关重要。