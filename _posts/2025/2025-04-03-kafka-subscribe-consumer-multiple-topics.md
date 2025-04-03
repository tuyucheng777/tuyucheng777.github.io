---
layout: post
title:  如何为Kafka消费者订阅多个主题
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 概述

**在本教程中，我们将学习如何让[Kafka](https://www.baeldung.com/spring-kafka)消费者订阅多个主题**。当多个主题使用相同的业务逻辑时，这是一个常见的需求。

## 2. 创建模型类

我们将考虑一个包含两个Kafka主题的简单支付系统，一个用于卡支付，另一个用于银行转账。让我们创建模型类：

```java
public class PaymentData {
    private String paymentReference;
    private String type;
    private BigDecimal amount;
    private Currency currency;

    // standard getters and setters
}
```

## 3. 使用Kafka Consumer API订阅多个主题

我们将讨论的第一种方法是使用[Kafka Consumer API](https://www.baeldung.com/java-kafka-consumer-api-read)，让我们添加所需的[Maven依赖](https://mvnrepository.com/artifact/org.apache.kafka/kafka-clients)：

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>3.5.1</version>
</dependency>
```

并配置Kafka消费者：

```java
Properties properties = new Properties();
properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, KAFKA_CONTAINER.getBootstrapServers());
properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
properties.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
properties.put(ConsumerConfig.GROUP_ID_CONFIG, "payments");
kafkaConsumer = new KafkaConsumer<>(properties);
```

**在消费消息之前，我们需要使用subscribe()方法将kafkaConsumer订阅到两个主题**：

```java
kafkaConsumer.subscribe(Arrays.asList("card-payments", "bank-transfers"));
```

现在我们已准备好测试配置，让我们在每个主题上发布一条消息：

```java
void publishMessages() throws Exception {
    ProducerRecord<String, String> cardPayment = new ProducerRecord<>("card-payments",
            "{\"paymentReference\":\"A184028KM0013790\", \"type\":\"card\", \"amount\":\"275\", \"currency\":\"GBP\"}");
    kafkaProducer.send(cardPayment).get();

    ProducerRecord<String, String> bankTransfer = new ProducerRecord<>("bank-transfers",
            "{\"paymentReference\":\"19ae2-18mk73-009\", \"type\":\"bank\", \"amount\":\"150\", \"currency\":\"EUR\"}");
    kafkaProducer.send(bankTransfer).get();
}
```

最后，我们可以编写集成测试：

```java
@Test
void whenSendingMessagesOnTwoTopics_thenConsumerReceivesMessages() throws Exception {
    publishMessages();
    kafkaConsumer.subscribe(Arrays.asList("card-payments", "bank-transfers"));

    int eventsProcessed = 0;
    for (ConsumerRecord<String, String> record : kafkaConsumer.poll(Duration.ofSeconds(10))) {
        log.info("Event on topic={}, payload={}", record.topic(), record.value());
        eventsProcessed++;
    }
    assertThat(eventsProcessed).isEqualTo(2);
}
```

## 4. 使用Spring Kafka订阅多个主题

我们将讨论的第二种方法是使用[Spring Kafka](https://www.baeldung.com/spring-kafka)。

让我们将[spring-kafka](https://mvnrepository.com/artifact/org.springframework.kafka/spring-kafka)和[jackson-databind](https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind)依赖添加到pom.xml中：

```xml
<dependency> 
    <groupId>org.springframework.kafka</groupId> 
    <artifactId>spring-kafka</artifactId>
    <version>3.1.2</version>
</dependency>
<dependency> 
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.15.2</version>
</dependency>
```

我们还定义ConsumerFactory和ConcurrentKafkaListenerContainerFactory Bean：

```java
@Bean
public ConsumerFactory<String, PaymentData> consumerFactory() {
    List<String, String> config = new HashMap<>();
    config.put(BOOTSTRAP_SERVERS_CONFIG, bootstrapAddress);
    config.put(VALUE_DESERIALIZER_CLASS_CONFIG, JsonDeserializer.class);

    return new DefaultKafkaConsumerFactory<>(
            config, new StringDeserializer(), new JsonDeserializer<>(PaymentData.class));
}

@Bean
public ConcurrentKafkaListenerContainerFactory<String, PaymentData> containerFactory() {
    ConcurrentKafkaListenerContainerFactory<String, PaymentData> factory =
            new ConcurrentKafkaListenerContainerFactory<>();
    factory.setConsumerFactory(consumerFactory());
    return factory;
}
```

**我们需要使用[@KafkaListener](https://docs.spring.io/spring-kafka/api/org/springframework/kafka/annotation/KafkaListener.html)注解的topics属性订阅这两个主题**：

```java
@KafkaListener(topics = { "card-payments", "bank-transfers" }, groupId = "payments")
```

最后，我们可以创建消费者。此外，我们还包括Kafka标头以标识接收消息的主题：

```java
@KafkaListener(topics = { "card-payments", "bank-transfers" }, groupId = "payments")
public void handlePaymentEvents(
        PaymentData paymentData, @Header(KafkaHeaders.RECEIVED_TOPIC) String topic) {
    log.info("Event on topic={}, payload={}", topic, paymentData);
}
```

让我们[验证我们的配置](https://www.baeldung.com/spring-boot-kafka-testing)：

```java
@Test
public void whenSendingMessagesOnTwoTopics_thenConsumerReceivesMessages() throws Exception {
    CountDownLatch countDownLatch = new CountDownLatch(2);
    doAnswer(invocation -> {
        countDownLatch.countDown();
        return null;
    }).when(paymentsConsumer)
            .handlePaymentEvents(any(), any());

    kafkaTemplate.send("card-payments", createCardPayment());
    kafkaTemplate.send("bank-transfers", createBankTransfer());

    assertThat(countDownLatch.await(5, TimeUnit.SECONDS)).isTrue();
}
```

## 5. 使用Kafka CLI订阅多个主题

Kafka CLI是我们将讨论的最后一种方法。

首先，让我们针对每个主题发送一条消息：

```shell
$ bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic card-payments
>{"paymentReference":"A184028KM0013790", "type":"card", "amount":"275", "currency":"GBP"}

$ bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic bank-transfers
>{"paymentReference":"19ae2-18mk73-009", "type":"bank", "amount":"150", "currency":"EUR"}
```

现在，我们可以启动Kafka CLI消费者了，**include选项允许我们指定要包含用于消息消费的主题列表**：

```shell
$ bin/kafka-console-consumer.sh --from-beginning --bootstrap-server localhost:9092 --include "card-payments|bank-transfers"

```

以下是运行上一个命令时的输出：

```text
{"paymentReference":"A184028KM0013790", "type":"card", "amount":"275", "currency":"GBP"}
{"paymentReference":"19ae2-18mk73-009", "type":"bank", "amount":"150", "currency":"EUR"}
```

## 6. 总结

在本文中，我们学习了3种不同的方法让Kafka消费者订阅多个主题，这在为多个主题实现相同功能时非常有用。

前两种方式基于Kafka Consumer API和Spring Kafka，可以集成到现有应用中。最后一种方式使用Kafka CLI，可以快速验证多个主题。