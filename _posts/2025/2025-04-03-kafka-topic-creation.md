---
layout: post
title:  使用Java创建Kafka主题
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 概述

在本教程中，我们将简要介绍[Apache Kafka](https://kafka.apache.org/)，然后了解如何以编程方式在Kafka集群中创建和配置主题。

## 2. Kafka简介

Apache Kafka是一个强大、高性能、分布式事件流平台。

通常，生产者应用程序将事件发布到Kafka，而消费者订阅这些事件以读取和处理它们。**Kafka使用主题来存储和分类这些事件**，例如，在电子商务应用程序中，可能有一个“orders”主题。

Kafka主题是分区的，它将数据分布在多个代理上以实现可扩展性。它们可以被复制，以使数据具有容错性和高可用性。主题还可以在消费后根据需要保留事件，**所有这些都是通过Kafka命令行工具和键值配置按主题进行管理的**。

但是，除了命令行工具之外，**Kafka还提供了[Admin API](https://kafka.apache.org/28/javadoc/org/apache/kafka/clients/admin/Admin.html)来管理和检查主题、代理和其他Kafka对象**。在我们的示例中，我们将使用此API来创建新主题。

## 3. 依赖

要使用Admin API，让我们将[kafka-clients](https://mvnrepository.com/artifact/org.apache.kafka/kafka-clients)依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>3.4.0</version>
</dependency>
```

## 4. 设置Kafka

在创建新主题之前，我们至少需要一个单节点Kafka集群。

在本教程中，我们将使用[Testcontainers](https://www.testcontainers.org/)框架实例化Kafka容器。然后，我们可以运行可靠且独立的集成测试，这些测试不依赖于运行的外部Kafka服务器。为此，我们需要另外两个专门用于测试的依赖。

首先，让我们将[Testcontainers Kafka](https://mvnrepository.com/artifact/org.testcontainers/kafka)依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>kafka</artifactId>
    <version>1.19.3</version>
    <scope>test</scope>
</dependency>
```

接下来，我们将添加[junit-jupiter](https://mvnrepository.com/artifact/org.testcontainers/junit-jupiter)以使用JUnit 5运行Testcontainer测试：

```xml
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>1.19.3</version>
    <scope>test</scope>
</dependency>
```

现在我们已经配置了所有必要的依赖，我们可以编写一个简单的应用程序来以编程方式创建新主题。

## 5. Admin API

让我们首先为本地代理创建一个具有最少配置的新[Properties](https://www.baeldung.com/java-properties)实例：

```java
Properties properties = new Properties();
properties.put(
    AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, KAFKA_CONTAINER.getBootstrapServers()
);
```

现在我们可以获取一个Admin实例：

```java
Admin admin = Admin.create(properties)
```

create方法接收具有[bootstrap.servers](https://kafka.apache.org/documentation/#adminclientconfigs_bootstrap.servers)属性的Properties对象(或Map)并返回线程安全的实例。

管理客户端使用此属性来发现集群中的代理，然后执行任何管理操作。因此，通常包含2个或3个代理地址就足以覆盖某些实例不可用的可能性。

[AdminClientConfig](https://kafka.apache.org/28/javadoc/org/apache/kafka/clients/admin/AdminClientConfig.html)类包含所有[管理客户端配置](https://kafka.apache.org/documentation/#adminclientconfigs)条目的常量。

## 6. 创建主题

让我们首先[使用Testcontainers创建JUnit 5测试](https://www.testcontainers.org/quickstart/junit_5_quickstart/)来验证主题创建是否成功，我们将使用[Kafka模块](https://www.testcontainers.org/modules/kafka/)，该模块使用[Confluent OSS平台](https://hub.docker.com/r/confluentinc/cp-kafka/)的官方Kafka Docker镜像：

```java
@Test
void givenTopicName_whenCreateNewTopic_thenTopicIsCreated() throws Exception {
    kafkaTopicApplication.createTopic("test-topic");

    String topicCommand = "/usr/bin/kafka-topics --bootstrap-server=localhost:9092 --list";
    String stdout = KAFKA_CONTAINER.execInContainer("/bin/sh", "-c", topicCommand)
            .getStdout();

    assertThat(stdout).contains("test-topic");
}
```

在这里，Testcontainers将在测试执行期间自动实例化和管理Kafka容器，我们只需调用我们的应用程序代码并验证主题是否已在正在运行的容器中成功创建。

### 6.1 使用默认选项创建

[主题分区和复制因子](https://www.baeldung.com/apache-kafka-data-modeling)是新主题的关键考虑因素，我们将尽量简单，并创建具有1个分区和复制因子为1的示例主题：

```java
try (Admin admin = Admin.create(properties)) {
    int partitions = 1;
    short replicationFactor = 1;
    NewTopic newTopic = new NewTopic(topicName, partitions, replicationFactor);
    
    CreateTopicsResult result = admin.createTopics(
        Collections.singleton(newTopic)
    );

    KafkaFuture<Void> future = result.values().get(topicName);
    future.get();
}
```

在这里，我们**使用Admin.createTopics方法创建一批具有默认选项的新主题**。由于Admin接口扩展了AutoCloseable接口，因此我们使用[try-with-resources](https://www.baeldung.com/java-try-with-resources)来执行操作，这可确保适当释放资源。

重要的是，此方法与Controller Broker通信并异步执行。返回的CreateTopicsResult对象公开一个KafkaFuture，用于访问请求批次中每个项目的结果。这遵循Java异步编程模式，并允许调用者使用[Future.get](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/Future.html#get())方法获取操作的结果。

对于同步行为，我们可以立即调用此方法来检索操作的结果。这将阻塞，直到操作完成或失败。如果失败，则会导致ExecutionException，其中包装了根本原因。

### 6.2 使用选项创建

除了默认选项外，我们**还可以使用Admin.createTopics方法的重载形式，并通过[CreateTopicsOptions](https://kafka.apache.org/28/javadoc/org/apache/kafka/clients/admin/CreateTopicsOptions.html)对象提供一些选项**，我们可以使用这些选项在创建新主题时修改管理客户端行为：

```java
CreateTopicsOptions topicOptions = new CreateTopicsOptions()
    .validateOnly(true)
    .retryOnQuotaViolation(false);

CreateTopicsResult result = admin.createTopics(
    Collections.singleton(newTopic), topicOptions
);
```

这里，我们将validateOnly选项设置为true，这意味着客户端只会进行验证而不会真正创建主题。同样，retryOnQuotaViolation选项设置为false，以便在配额违规的情况下不会重试该操作。

### 6.3 新建主题配置

Kafka具有广泛的[主题配置](https://kafka.apache.org/documentation.html#topicconfigs)来控制主题行为，例如[数据保留](https://www.baeldung.com/kafka-message-retention)和压缩等。这些配置既有服务器默认值，也有可选的每个主题覆盖。

我们可以**使用新主题的配置Map来提供主题配置**：

```java
// Create a compacted topic with 'lz4' compression codec
Map<String, String> newTopicConfig = new HashMap<>();
newTopicConfig.put(TopicConfig.CLEANUP_POLICY_CONFIG, TopicConfig.CLEANUP_POLICY_COMPACT);
newTopicConfig.put(TopicConfig.COMPRESSION_TYPE_CONFIG, "lz4");

NewTopic newTopic = new NewTopic(topicName, partitions, replicationFactor)
    .configs(newTopicConfig);
```

Admin API中的[TopicConfig](https://kafka.apache.org/28/javadoc/org/apache/kafka/common/config/TopicConfig.html)类包含可用于在创建时配置主题的键。

## 7. 其他主题操作

除了创建新主题的功能外，**Admin API还具有[删除](https://kafka.apache.org/28/javadoc/org/apache/kafka/clients/admin/Admin.html#deleteTopics(java.util.Collection))、[列出](https://kafka.apache.org/28/javadoc/org/apache/kafka/clients/admin/Admin.html#listTopics())和[描述](https://kafka.apache.org/28/javadoc/org/apache/kafka/clients/admin/Admin.html#describeTopics(java.util.Collection))主题的操作**，所有这些与主题相关的操作都遵循与主题创建相同的模式。

这些操作方法中的每一个都有一个重载版本，该版本以xxxTopicOptions对象作为输入，所有这些方法都返回相应的xxxTopicsResult对象，这反过来又提供了KafkaFuture来访问异步操作的结果。

最后，值得一提的是，自从Kafka 0.11.0.0版本引入以来，Admin API仍在不断发展，如InterfaceStability.Evolving注解所示。这意味着API将来可能会发生变化，小版本可能会破坏兼容性。

## 8. 总结

在本教程中，我们了解了如何使用Java管理客户端在Kafka中创建新主题。

首先，我们创建了一个具有默认选项的主题，然后指定了自定义选项。接下来，我们了解了如何使用各种属性配置新主题。最后，我们简要介绍了使用管理客户端的其他主题相关操作。

在此过程中，我们还了解了如何使用Testcontainers从我们的测试中设置一个简单的单节点集群。