---
layout: post
title:  在Kafka中拆分数据流
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 简介

在本教程中，我们将探索如何在[Kafka Streams](https://www.baeldung.com/spring-boot-kafka-streams)中动态路由消息。**当消息的目标主题取决于其内容时，动态路由特别有用，使我们能够根据有效负载中的特定条件或属性来定向消息**。这种条件路由在各种领域都有实际应用，例如物联网事件处理、用户活动跟踪和欺诈检测。

我们将介绍如何消费单个[Kafka主题](https://www.baeldung.com/kafka-topic-creation)中的消息并有条件地将其路由到多个目标主题的问题，主要重点是如何使用Kafka Streams库在[Spring Boot](https://www.baeldung.com/category/spring/spring-boot)应用程序中进行设置。

## 2. Kafka Streams路由技术

**Kafka Streams中的消息动态路由不仅限于单一方法，而是可以使用多种技术实现**，每种方法都有其独特的优势、挑战和对各种场景的适用性：

- KStream条件分支：KStream.split().branch()方法是基于谓词划分流的常规方法，虽然此方法易于实现，但在扩展条件数量时存在局限性，并且变得难以管理。
- 使用KafkaStreamBrancher进行分支：该功能出现在Spring Kafka 2.2.4版本中，它提供了一种更优雅、更易读的方式来在Kafka Stream中创建分支，无需使用“魔法数字”，并允许更流式地链接流操作。
- 使用TopicNameExtractor进行动态路由：主题路由的另一种方法是使用TopicNameExtractor。这允许在运行时根据消息键、值甚至整个记录上下文进行更动态的主题选择。但是，它需要提前创建主题。此方法可以更精细地控制主题选择，并且更适合复杂的用例。
- 自定义处理器：对于需要复杂路由逻辑或多个链式操作的场景，我们可以在Kafka Streams拓扑中应用自定义处理器节点。这种方法最灵活，但实现起来也最复杂。

在本文中，我们将重点介绍实现前三种方法-KStream条件分支、使用KafkaStreamBrancher进行分支以及使用TopicNameExtractor进行动态路由。

## 3. 设置环境

在我们的场景中，我们有一个IoT传感器网络，它将各种类型的数据(例如温度、湿度和运动)传输到名为iot_sensor_data的集中式Kafka主题，每条传入消息都包含一个JSON对象，其中有一个名为sensorType的字段，该字段指示传感器发送的数据类型。我们的目标是将这些消息动态路由到每种传感器数据类型的专用主题。

首先，让我们建立一个正在运行的Kafka实例。我们可以通过创建docker-compose.yml文件，使用[Docker](https://www.baeldung.com/ops/category/docker)以及[Docker Compose](https://www.baeldung.com/ops/docker-compose)设置Kafka、[Zookeeper](https://www.baeldung.com/java-zookeeper)和[Kafka UI](https://github.com/provectus/kafka-ui)：

```yaml
version: '3.8'
services:
    zookeeper:
        image: confluentinc/cp-zookeeper:latest
        environment:
            ZOOKEEPER_CLIENT_PORT: 2181
            ZOOKEEPER_TICK_TIME: 2000
        ports:
            - 22181:2181
    kafka:
        image: confluentinc/cp-kafka:latest
        depends_on:
            - zookeeper
        ports:
            - 9092:9092
        environment:
            KAFKA_BROKER_ID: 1
            KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
            KAFKA_LISTENERS: "INTERNAL://:29092,EXTERNAL://:9092"
            KAFKA_ADVERTISED_LISTENERS: "INTERNAL://kafka:29092,EXTERNAL://localhost:9092"
            KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: "INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT"
            KAFKA_INTER_BROKER_LISTENER_NAME: "INTERNAL"
            KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
    kafka_ui:
        image: provectuslabs/kafka-ui:latest
        depends_on:
            - kafka
        ports:
            - 8082:8080
        environment:
            KAFKA_CLUSTERS_0_ZOOKEEPER: zookeeper:2181
            KAFKA_CLUSTERS_0_NAME: local
            KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: kafka:29092
    kafka-init-topics:
        image: confluentinc/cp-kafka:latest
        depends_on:
            - kafka
        command: "bash -c 'echo Waiting for Kafka to be ready... && \
               cub kafka-ready -b kafka:29092 1 30 && \
               kafka-topics --create --topic iot_sensor_data --partitions 1 --replication-factor 1 --if-not-exists --bootstrap-server kafka:29092'"
```

在这里，我们设置了所有必需的环境变量和服务之间的依赖关系。此外，我们使用kafka-init-topics服务中的特定命令创建iot_sensor_data主题。

现在我们可以通过执行docker-compose up -d在Docker中运行Kafka。

接下来，我们必须将Kafka Streams依赖添加到pom.xml文件中：

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-streams</artifactId>
    <version>3.6.1</version>`
</dependency>
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
    <version>3.1.2</version>
</dependency>
```

第一个依赖是[org.apache.kafka.kafka-streams](https://mvnrepository.com/artifact/org.apache.kafka/kafka-streams)包，它提供Kafka Streams功能。后续的Maven包[org.springframework.kafka.spring -kafka](https://mvnrepository.com/artifact/org.springframework.kafka/spring-kafka)方便Kafka与Spring Boot的配置和集成。

另一个重要方面是配置Kafka代理的地址，这通常是通过在应用程序的属性文件中指定代理详细信息来完成的。让我们将此配置与其他属性一起添加到我们的application.properties文件中：

```properties
spring.kafka.bootstrap-servers=localhost:9092
spring.kafka.streams.application-id=tuyucheng-streams
spring.kafka.consumer.group-id=tuyucheng-group
spring.kafka.streams.properties[default.key.serde]=org.apache.kafka.common.serialization.Serdes$StringSerde
kafka.topics.iot=iot_sensor_data
```

接下来，我们定义一个示例数据类IotSensorData：

```java
public class IotSensorData {
    private String sensorType;
    private String value;
    private String sensorId;
}
```

最后，我们需要配置Serde来进行Kafka中类型化消息的序列化和反序列化：

```java
@Bean
public Serde<IotSensorData> iotSerde() {
    return Serdes.serdeFrom(new JsonSerializer<>(), new JsonDeserializer<>(IotSensorData.class));
}
```

## 4. 在Kafka Streams中实现动态路由

设置环境并安装所需的依赖后，让我们专注于在Kafka Streams中实现动态路由逻辑。

**动态消息路由是事件驱动应用程序的重要组成部分，因为它使系统能够适应各种类型的数据流和条件，而无需更改代码**。

### 4.1 KStream条件分支

Kafka Streams中的分支功能允许我们根据某些条件将单个数据流拆分为多个流，这些条件以谓词形式提供，用于评估通过流的每条消息。

**在Kafka Streams的最新版本中，branch()方法已被弃用，取而代之的是较新的split().branch()方法，该方法旨在提高API的整体可用性和灵活性**。不过，我们可以以相同的方式应用它，根据某些谓词将KStream拆分为多个流。

这里我们定义利用split().branch()方法进行动态主题路由的配置：

```java
@Bean
public KStream<String, IotSensorData> iotStream(StreamsBuilder streamsBuilder) {
    KStream<String, IotSensorData> stream = streamsBuilder.stream(iotTopicName, Consumed.with(Serdes.String(), iotSerde()));
    stream.split()
            .branch((key, value) -> "temp".equals(value.getSensorType()), Branched.withConsumer((ks) -> ks.to(iotTopicName + "_temp")))
            .branch((key, value) -> "move".equals(value.getSensorType()), Branched.withConsumer((ks) -> ks.to(iotTopicName + "_move")))
            .branch((key, value) -> "hum".equals(value.getSensorType()), Branched.withConsumer((ks) -> ks.to(iotTopicName + "_hum")))
            .noDefaultBranch();
    return stream;
}
```

在上面的示例中，我们根据sensorType属性将来自iot_sensor_data主题的初始流拆分为多个流，并相应地将它们路由到其他主题。

如果可以根据消息内容生成目标主题名称，我们可以在to方法中使用Lambda函数实现更动态的主题路由：

```java
@Bean
public KStream<String, IotSensorData> iotStreamDynamic(StreamsBuilder streamsBuilder) {
    KStream<String, IotSensorData> stream = streamsBuilder.stream(iotTopicName, Consumed.with(Serdes.String(), iotSerde()));
    stream.split()
            .branch((key, value) -> value.getSensorType() != null,
                    Branched.withConsumer(ks -> ks.to((key, value, recordContext) -> "%s_%s".formatted(iotTopicName, value.getSensorType()))))
            .noDefaultBranch();
    return stream;
}
```

如果可以根据消息内容生成主题名称，则这种方法可以为根据消息内容动态路由消息提供更大的灵活性。

### 4.2 使用KafkaStreamBrancher进行路由

**KafkaStreamBrancher类提供了一个构建器风格的API，可以更轻松地链接分支条件，从而使代码更具可读性和可维护性**。

主要的好处是消除了与管理分支流数组相关的复杂性，而这正是原始KStream.branch方法的工作方式。相反，KafkaStreamBrancher允许我们定义每个分支以及应该发生在该分支上的操作，从而无需使用魔法数字或复杂的索引来识别正确的分支。由于引入了split().branch()方法，这种方法与前面讨论的方法密切相关。

我们将这种方法应用到流中：

```java
@Bean
public KStream<String, IotSensorData> kStream(StreamsBuilder streamsBuilder) {
    KStream<String, IotSensorData> stream = streamsBuilder.stream(iotTopicName, Consumed.with(Serdes.String(), iotSerde()));
    new KafkaStreamBrancher<String, IotSensorData>()
            .branch((key, value) -> "temp".equals(value.getSensorType()), (ks) -> ks.to(iotTopicName + "_temp"))
            .branch((key, value) -> "move".equals(value.getSensorType()), (ks) -> ks.to(iotTopicName + "_move"))
            .branch((key, value) -> "hum".equals(value.getSensorType()), (ks) -> ks.to(iotTopicName + "_hum"))
            .defaultBranch(ks -> ks.to("%s_unknown".formatted(iotTopicName)))
            .onTopOf(stream);
    return stream;
}
```

我们已应用[流式API](https://www.baeldung.com/java-fluent-interface-vs-builder-pattern)将消息路由到特定主题。类似地，我们可以使用单个branch()方法调用将内容用作主题名称的一部分，从而路由到多个主题：

```java
@Bean
public KStream<String, IotSensorData> iotBrancherStream(StreamsBuilder streamsBuilder) {
    KStream<String, IotSensorData> stream = streamsBuilder.stream(iotTopicName, Consumed.with(Serdes.String(), iotSerde()));
    new KafkaStreamBrancher<String, IotSensorData>()
            .branch((key, value) -> value.getSensorType() != null, (ks) ->
                    ks.to((key, value, recordContext) -> String.format("%s_%s", iotTopicName, value.getSensorType())))
            .defaultBranch(ks -> ks.to("%s_unknown".formatted(iotTopicName)))
            .onTopOf(stream);
    return stream;
}
```

通过为分支逻辑提供更高级别的抽象，KafkaStreamBrancher不仅使代码更简洁，而且增强了其可管理性，特别是对于具有复杂路由要求的应用程序。

### 4.3 使用TopicNameExtractor进行动态主题路由

管理Kafka Streams中的条件分支的另一种方法是使用TopicNameExtractor，顾名思义，它可以动态提取流中每条消息的主题名称。与之前讨论的split().branch()和KafkaStreamBrancher方法相比，这种方法对于某些用例来说可能更直接。

以下是在Spring Boot应用程序中使用TopicNameExtractor的示例配置：

```java
@Bean
public KStream<String, IotSensorData> kStream(StreamsBuilder streamsBuilder) {
    KStream<String, IotSensorData> stream = streamsBuilder.stream(iotTopicName, Consumed.with(Serdes.String(), iotSerde()));
    TopicNameExtractor<String, IotSensorData> sensorTopicExtractor = (key, value, recordContext) -> "%s_%s".formatted(iotTopicName, value.getSensorType());
    stream.to(sensorTopicExtractor);
    return stream;
}
```

虽然TopicNameExtractor方法在将记录路由到特定主题的主要功能上非常出色，但与split().branch()和KafkaStreamBrancher等其他方法相比，它还是存在一些局限性。**具体来说，TopicNameExtractor不提供在同一路由步骤中执行其他转换(如映射或过滤)的选项**。

## 5. 总结

在本文中，我们看到了使用Kafka Streams和Spring Boot进行动态主题路由的不同方法。

我们首先探索了现代分支机制，例如split().branch()方法和KafkaStreamBrancher类。此外，我们还研究了TopicNameExtractor提供的动态主题路由功能。

每种技术都有其优点和挑战。例如，**split().branch()在处理大量条件时可能很麻烦，而TopicNameExtractor提供了结构化流程，但限制了某些内联数据处理**。因此，掌握每种方法的细微差别对于创建有效的路由实现至关重要。