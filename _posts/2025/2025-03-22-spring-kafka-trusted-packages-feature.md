---
layout: post
title:  Spring Kafka可信包功能
category: spring-boot
copyright: spring-boot
excerpt: Spring Boot
---

## 1. 简介

在本教程中，我们将回顾[Spring Kafka](https://www.baeldung.com/spring-kafka)可信包功能，我们将了解其背后的动机及其用法。一如既往，所有内容都附有实际示例。

## 2. 先决条件

通常，Spring Kafka模块允许我们作为用户指定有关要发送的POJO的一些元数据。它通常采用Kafka消息头的形式。例如，如果我们以这种方式配置ProducerFactory：

```java
@Bean
public ProducerFactory<Object, SomeData> producerFactory() {
    JsonSerializer<SomeData> jsonSerializer = new JsonSerializer<>();
    jsonSerializer.setAddTypeInfo(true);
    return new DefaultKafkaProducerFactory<>(
            producerFactoryConfig(),
            new StringOrBytesSerializer(),
            jsonSerializer
    );
}

@Data
@AllArgsConstructor
static class SomeData {

    private String id;
    private String type;
    private String status;
    private Instant timestamp;
}
```

然后我们将在一个主题中生成一条新消息，例如，使用上面配置了producerFactory的KafkaTemplate：

```java
public void sendDataIntoKafka() {
    SomeData someData = new SomeData("1", "active", "sent", Instant.now());
    kafkaTemplate.send(new ProducerRecord<>("sourceTopic", null, someData));
}
```

然后，在这种情况下，我们将在Kafka消费者的控制台中收到以下消息：

```text
CreateTime:1701021806470 __TypeId__:cn.tuyucheng.taketoday.example.SomeData null {"id":"1","type":"active","status":"sent","timestamp":1701021806.153965150}
```

我们可以看到，消息中的POJO的类型信息位于标头中。当然，这是Spring唯一能识别的Spring Kafka功能。也就是说，这些标头只是Kafka或其他框架的元数据。因此，我们可以假设消费者和生产者都使用Spring来处理Kafka消息传递。

## 3. 受信任的软件包功能

话虽如此，我们还是可以说，在某些情况下，这是一个非常有用的功能。当主题中的消息具有不同的有效负载架构时，向消费者提示有效负载类型将非常有用。

![](/assets/images/2025/springboot/springkafkatrustedpackagesfeature01.png)

但是，一般来说，我们知道哪些消息(就其架构而言)可以出现在主题中。因此，限制消费者可能接受的有效负载架构可能是一个好主意。这就是Spring Kafka受信任包功能的目的。

## 4. 使用示例

受信任的包Spring Kafka功能在反序列化器级别配置，如果配置了受信任的包，Spring将查找传入消息的类型标头。然后，它将检查消息中提供的所有类型是否受信任-包括键和值。

本质上意味着，[在相应标头中指定](https://github.com/spring-projects/spring-kafka/blob/1a18d288dc80e7e94db0fd8a2242f1d6c92c3b1e/spring-kafka/src/main/java/org/springframework/kafka/support/mapping/AbstractJavaTypeMapper.java#L48)的键和值的Java类必须位于受信任的包内。如果一切正常，Spring会将消息传递到进一步的反序列化中。如果标头不存在，则Spring将仅反序列化对象，而不会检查受信任的包：

```java
@Bean
public ConsumerFactory<String, SomeData> someDataConsumerFactory() {
    JsonDeserializer<SomeData> payloadJsonDeserializer = new JsonDeserializer<>();
    payloadJsonDeserializer.addTrustedPackages("cn.tuyucheng.taketoday.example");
    return new DefaultKafkaConsumerFactory<>(
            consumerConfigs(),
            new StringDeserializer(),
            payloadJsonDeserializer
    );
}
```

也许还值得一提的是，如果我们用星号(*)替换具体包，Spring可以信任所有包：

```java
JsonDeserializer<SomeData> payloadJsonDeserializer = new JsonDeserializer<>();
payloadJsonDeserializer.trustedPackages("*");
```

但是，在这种情况下，使用受信任的包不会产生任何作用，只会产生额外的开销。现在让我们深入了解一下我们刚刚看到的功能背后的动机。

### 5.1 第一个动机：一致性

此功能很棒，主要有两个原因。首先，如果集群出现问题，我们可以快速失败。想象一下，某个生产者会意外地在某个他不应该发布的主题中发布消息。这可能会导致很多问题，特别是如果我们成功反序列化传入的消息。在这种情况下，整个系统行为可能是不确定的。

因此，如果生产者发布的消息中包含类型信息，并且消费者知道它信任哪些类型，那么这一切都可以避免。当然，这假设生产者的消息类型与消费者期望的不同。但这个假设相当合理，因为这个生产者根本不应该向这个主题发布消息。

### 5.2 第二个动机：安全

但最重要的是安全问题。在我们之前的例子中，我们强调生产者无意中向主题发布了消息，**但这也可能是故意攻击**，恶意生产者可能会故意向特定主题发布消息以利用反序列化漏洞。因此，通过防止不需要的消息反序列化，Spring提供了额外的安全措施来降低安全风险。

**这里真正需要理解的是，可信包功能并不是“标头欺骗”攻击的解决方案**。在这种情况下，攻击者操纵消息的标头，欺骗接收者相信该消息是合法的并且来自可信来源。因此，通过提供正确的类型标头，攻击者可能会欺骗Spring，后者将继续进行消息反序列化。但这个问题相当复杂，不是讨论的主题。一般来说，Spring只是提供了一种额外的安全措施，以最大限度地降低黑客成功的风险。

## 6. 总结

在本文中，我们探讨了Spring Kafka受信任包功能，此功能为我们的分布式消息传递系统提供了额外的一致性和安全性。不过，请务必记住，受信任的包仍然容易受到标头欺骗攻击。不过，Spring Kafka在提供额外的安全措施方面做得很好。