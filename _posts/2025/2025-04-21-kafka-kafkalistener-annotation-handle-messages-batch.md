---
layout: post
title:  使用@KafkaListener在Kafka中批量消费消息
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 概述

在本教程中，我们将讨论如何使用Spring Kafka库的@KafkaListener注解批量处理Kafka消息。**Kafka Broker是一个中间件，用于持久化来自源系统的消息，目标系统配置为定期轮询Kafka主题/队列，然后从中读取消息**。

这可以防止目标系统或服务宕机时消息丢失，当目标服务恢复时，它们会继续接收未处理的消息。因此，这种架构有助于提高消息的持久性，从而提高系统的容错能力。

## 2. 为什么要批量处理消息？

多个源或事件生产者同时向同一个Kafka队列或主题发送消息的情况很常见，因此，大量消息可能会在其中堆积，如果目标服务或消费者在一次会话中收到这些海量消息，它们可能无法高效地处理它们。

这可能会产生连锁反应，导致瓶颈。最终，这会影响所有依赖消息的下游进程。因此，消息消费者或消息监听者应该限制其在同一时间点可以处理的消息数量。

**要以批处理模式运行，我们必须根据主题上发布的数据量和应用程序的容量来配置正确的批处理大小**。此外，消费者应用程序应设计为能够批量处理消息，以满足SLA的要求。

此外，如果没有批处理，消费者必须定期轮询Kafka主题才能逐条获取消息，这种方法会给计算资源带来压力。因此，批处理比每次轮询处理一条消息效率更高。

但是， **批处理可能不适用于以下特定情况**：

- 消息量较小
- 在时间敏感的应用中，立即处理至关重要
- 计算和内存资源受到限制
- 严格的消息排序至关重要

## 3. 使用@KafkaListener注解进行批处理

为了理解批处理，我们首先要定义一个用例。然后，我们将先使用基本消息处理来实现它，然后再使用批处理来实现它。这样，我们就能更好地理解批量处理消息的重要性。

### 3.1 用例描述

假设一家公司的数据中心运行着许多关键的IT基础设施设备，例如服务器和网络设备，多种监控工具会跟踪这些设备的KPI(关键绩效指标)，由于运营团队希望进行主动监控，他们期望获得实时且可操作的分析数据。因此，存在严格的SLA来将KPI传输到目标分析应用程序。

运营团队配置监控工具，定期将KPI发送到Kafka主题，消费者应用程序从主题读取消息，然后将其推送到数据湖，应用程序从数据湖读取数据并生成实时分析。

**让我们分别实现一个配置了批处理的消费者和未配置批处理的消费者，我们将分析这两种实现方式的差异和结果**。

### 3.2 先决条件

在开始实现批处理之前，了解Spring Kafka库至关重要，之前，我们在[Spring Apache Kafka简介](https://www.baeldung.com/spring-kafka)一文中讨论了这个问题。

为了学习，我们需要一个Kafka实例。因此，**为了快速入门，我们将使用[嵌入式Kafka](https://docs.spring.io/spring-kafka/reference/testing.html)**。

最后，我们需要一个程序，在Kafka代理中创建一个事件队列，并定期向其发布示例消息。本质上，我们将使用[JUnit 5](https://www.baeldung.com/junit-5)来理解这个概念。

### 3.3 基本监听器

让我们从一个基本的监听器开始，它从Kafka Broker逐个读取消息，我们将在KafkaKpiConsumerWithNoBatchConfig配置类中定义[ConcurrentKafkaListenerContainerFactory](https://docs.spring.io/spring-kafka/reference/kafka/container-factory.html) Bean：

```java
public class KafkaKpiConsumerWithNoBatchConfig {

    @Bean(name = "kafkaKpiListenerContainerFactory")
    public ConcurrentKafkaListenerContainerFactory<String, String> kafkaKpiBasicListenerContainerFactory(ConsumerFactory<String, String> consumerFactory) {
        ConcurrentKafkaListenerContainerFactory<String, String> factory = new ConcurrentKafkaListenerContainerFactory();
        factory.setConsumerFactory(consumerFactory);
        factory.setConcurrency(1);
        factory.getContainerProperties().setPollTimeout(3000);
        return factory;
    }
}
```

**kafkaKpiBasicListenerContainerFactory()方法返回kafkaKpiListenerContainerFactory Bean，该Bean用于配置一个基本监听器，该监听器每次只能处理一条消息**：

```java
@Component
public class KpiConsumer {
    private CountDownLatch latch = new CountDownLatch(1);

    private ConsumerRecord<String, String> message;
    @Autowired
    private DataLakeService dataLakeService;

    @KafkaListener(
            id = "kpi-listener",
            topics = "kpi_topic",
            containerFactory = "kafkaKpiListenerContainerFactory")
    public void listen(ConsumerRecord<String, String> record) throws InterruptedException {
        this.message = record;

        latch.await();

        List<String> messages = new ArrayList<>();
        messages.add(record.value());
        dataLakeService.save(messages);
        // reset the latch
        latch = new CountDownLatch(1);
    }
    // General getter methods
}
```

我们在listen()方法上应用了@KafkaListener注解，该注解用于设置监听器主题和监听器容器工厂Bean。KpiConsumer类中的[java.util.concurrent.CountDownLatch](https://www.baeldung.com/java-countdown-latch)对象用于控制JUnit 5测试中的消息处理，我们将用它来理解整个概念。

CountDownLatch#await()方法会暂停监听线程，当测试方法调用CountDownLatch#countDown()方法时，监听线程会恢复。如果没有这个方法，理解和跟踪消息将会非常困难。最终，下游的DataLakeService#save()方法会收到一条消息进行处理。

现在让我们看一下帮助跟踪KpiListener类处理的消息的方法：

```java
@RepeatedTest(10)
void givenKafka_whenMessage1OnTopic_thenListenerConsumesMessages(RepetitionInfo repetitionInfo) {
    String testNo = String.valueOf(repetitionInfo.getCurrentRepetition());
    assertThat(kpiConsumer.getMessage().value()).isEqualTo("Test KPI Message-".concat(testNo));
    kpiConsumer.getLatch().countDown();
}
```

**当监控工具将KPI消息发布到kpi_topic Kafka主题时，监听器会按照消息到达的顺序接收它们**。

该方法每次执行时，都会跟踪到达KpiListener#listen()方法的消息，确认消息顺序后，释放Latch，监听器完成处理。

### 3.4 具有批处理能力的监听器

现在，让我们探索一下Kafka中的批处理支持。首先，在Spring配置类中定义ConcurrentKafkaListenerContainerFactory Bean：

```java
@Bean(name="kafkaKpiListenerContainerFactory")
public ConcurrentKafkaListenerContainerFactory<String, String> kafkaKpiBatchListenerContainerFactory(ConsumerFactory<String, String> consumerFactory) {
    ConcurrentKafkaListenerContainerFactory<String, String> factory = new ConcurrentKafkaListenerContainerFactory();

    Map<String, Object> configProps = new HashMap<>();
    configProps.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, "20");
    consumerFactory.updateConfigs(configProps);
    factory.setConcurrency(1);
    factory.setConsumerFactory(consumerFactory);
    factory.getContainerProperties().setPollTimeout(3000);
    factory.setBatchListener(true);

    return factory;
}
```

该方法与上一节中定义的kafkaKpiBasicListenerContainerFactory()方法类似，**我们通过调用ConsumerFactory#setBatchListener()方法启用了批处理**。

此外，我们借助ConsumerConfig.MAX_POLL_RECORDS_CONFIG属性设置了每次轮询的最大消息数，ConsumerFactory#setConcurrency()函数用于设置同时处理消息的并发消费者线程数，其他[配置](https://docs.spring.io/spring-kafka/reference/kafka/container-props.html)可以参考Spring Kafka官方网站。

此外，**还有像ConsumerConfig.DEFAULT_FETCH_MAX_BYTES和ConsumerConfig.DEFAULT_FETCH_MIN_BYTES这样的配置属性可以帮助限制消息大小**。

现在，让我们看看消费者：

```java
@Component
public class KpiBatchConsumer {
    private CountDownLatch latch = new CountDownLatch(1);
    @Autowired
    private DataLakeService dataLakeService;
    private List<String> receivedMessages = new ArrayList<>();

    @KafkaListener(
            id = "kpi-batch-listener",
            topics = "kpi_batch_topic",
            batch = "true",
            containerFactory = "kafkaKpiListenerContainerFactory")
    public void listen(ConsumerRecords<String, String> records) throws InterruptedException {
        records.forEach(record -> receivedMessages.add(record.value()));

        latch.await();

        dataLakeService.save(receivedMessages);
        latch = new CountDownLatch(1);
    }
    // Standard getter methods
}
```

KpiBatchConsumer与之前定义的KpiConsumer类类似，不同之处在于@KafkaListener注解多了一个batch属性。**listen()方法接收ConsumerRecords类型的参数，而不是ConsumerRecord类型，我们可以遍历ConsumerRecords对象来获取批次中的所有ConsumerRecord元素**。

监听器也可以按照消息到达的顺序处理批量接收的消息，然而，在Kafka中，[跨主题分区维护批量消息的顺序](https://www.baeldung.com/kafka-message-ordering)非常复杂。

这里，ConsumerRecord表示发布到Kafka主题的消息。最终，我们调用DataLakeService#save()方法并传入更多消息。最后，CountDownLatch类的作用与我们之前看到的相同。

假设有100条KPI消息被推送到Kafka主题kpi_batch_topic中，现在我们可以检查监听器的运行情况：

```java
@RepeatedTest(5)
void givenKafka_whenMessagesOnTopic_thenListenerConsumesMessages() {
    int messageSize = kpiBatchConsumer.getReceivedMessages().size();

    assertThat(messageSize % 20).isEqualTo(0);
    kpiBatchConsumer.getLatch().countDown();
}
```

与基本监听器逐一接收消息不同，这次监听器KpiBatchConsumer#listen()方法接收一批包含20条KPI消息。

## 4. 总结

在本文中，我们讨论了Kafka基本监听器和启用批处理的监听器之间的区别。批处理有助于同时处理多条消息，从而提升应用程序性能。然而，**适当限制批处理量和消息大小对于控制应用程序性能至关重要**。因此，必须经过仔细而严格的基准测试后进行优化。