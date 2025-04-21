---
layout: post
title:  理解Kafka消费者偏移量
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 概述

Kafka消费者[偏移量](https://www.baeldung.com/kafka-commit-offsets#what-is-offset)是一个唯一、单调递增的整数，用于标识事件记录在分区中的位置。组中的每个消费者都会为每个分区维护一个特定的偏移量，以便跟踪进度。另一方面，Kafka[消费者组](https://www.baeldung.com/apache-kafka#1-producers-amp-consumers)由负责通过轮询从多个分区读取主题消息的消费者组成。

Kafka中的[组协调器](https://www.baeldung.com/kafka-manage-consumer-groups#1-the-group-coordinator-and-the-group-leader)负责管理消费者组，并为组内的消费者分配分区。当消费者启动时，它会定位其组的协调器并请求加入，协调器会触发组重平衡，为新成员分配其应得的分区份额。

在本教程中，让我们探索这些偏移量的保存位置以及消费者如何使用它们来跟踪和启动或恢复他们的进度。

## 2. 设置

**让我们首先使用[Docker Compose](https://www.baeldung.com/ops/docker-compose)脚本在Kraft模式下[设置单实例Kafka集群](https://docs.confluent.io/platform/current/get-started/platform-quickstart.html#step-1-download-and-start-cp)**：

```yaml
broker:
    image: confluentinc/cp-kafka:7.7.0
    hostname: broker
    container_name: broker
    ports:
        - "9092:9092"
        - "9101:9101"
    expose:
        - '29092'
    environment:
        KAFKA_NODE_ID: 1
        KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: 'CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT'
        KAFKA_ADVERTISED_LISTENERS: 'PLAINTEXT://broker:29092,PLAINTEXT_HOST://localhost:9092'
        KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
        KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
        KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
        KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
        KAFKA_JMX_PORT: 9101
        KAFKA_JMX_HOSTNAME: localhost
        KAFKA_PROCESS_ROLES: 'broker,controller'
        KAFKA_CONTROLLER_QUORUM_VOTERS: '1@broker:29093'
        KAFKA_LISTENERS: 'PLAINTEXT://broker:29092,CONTROLLER://broker:29093,PLAINTEXT_HOST://0.0.0.0:9092'
        KAFKA_INTER_BROKER_LISTENER_NAME: 'PLAINTEXT'
        KAFKA_CONTROLLER_LISTENER_NAMES: 'CONTROLLER'
        KAFKA_LOG_DIRS: '/tmp/kraft-combined-logs'
        CLUSTER_ID: 'MkU3OEVBNTcwNTJENDM2Qk'
        KAFKA_LOG_CLEANUP_POLICY: 'delete'
```

这应该使集群在http://localhost:9092/上可用。

接下来，让我们创建一个具有两个分区的主题：

```yaml
init-kafka:
    image: confluentinc/cp-kafka:7.7.0
    depends_on:
        - broker
    entrypoint: [ '/bin/sh', '-c' ]
    command: |
        " # blocks until kafka is reachable
        kafka-topics --bootstrap-server broker:29092 --list
        echo -e 'Creating kafka topics'
        kafka-topics --bootstrap-server broker:29092 --create \
          --if-not-exists --topic user-data --partitions 2 "
```

作为可选步骤，让我们设置Kafka UI以轻松查看消息，但在本文中我们将使用CLI检查详细信息：

```yaml
kafka-ui:
    image: provectuslabs/kafka-ui:latest
    ports:
        - "3030:8080"
    depends_on:
        - broker
        - init-kafka
    environment:
        KAFKA_CLUSTERS_0_NAME: broker
        KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: broker:29092
```

这使得Kafka UI可以通过http://localhost:3030/访问：

![](/assets/images/2025/kafka/kafkaconsumeroffset01.png)

## 3. 消费者偏移量参考配置

**当消费者第一次加入组时，它会根据[auto.offset.reset](https://www.baeldung.com/java-kafka-consumer-api-read#1-consumer-properties)配置来确定获取记录的偏移位置，设置为early或latest**。

我们作为生产者来推送几条消息：

```shell
docker exec -i <CONTAINER-ID> kafka-console-producer \
  --broker-list localhost:9092 \
  --topic user-data <<< '{"id": 1, "first_name": "John", "last_name": "Doe"}
{"id": 2, "first_name": "Alice", "last_name": "Johnson"}'

```

接下来，让我们通过注册一个消费者来消费这些消息，从主题用户数据中读取这些消息，并将所有分区的auto.offset.reset设置为earliest：

```shell
docker exec -it <CONTAINER_ID> kafka-console-consumer \
  --bootstrap-server localhost:9092 \
  --topic user-data \
  --consumer-property auto.offset.reset=earliest \
  --group consumer-user-data
```

这会将新的消费者添加到consumer-user-data组，我们可以在Broker日志和Kafka UI中检查重平衡操作，它还会列出基于最早重置策略的所有消息。

我们需要记住，消费者在终端中保持打开状态，以便持续消费消息。为了检查中断后的行为，我们终止了此会话。

## 4. 消费者偏移量参考主题

**当消费者加入群组时，代理会创建一个内部主题__consumer_offsets，用于在主题和分区级别存储客户偏移量状态**。如果启用了Kafka[自动提交功能](https://www.baeldung.com/kafka-commit-offsets#1-auto-commit)，消费者会定期将最后处理的消息偏移量提交到此主题。这样，在中断后恢复消费时，就可以使用该状态。

当组中的某个消费者因崩溃或断开连接而失败时，Kafka会检测到丢失的心跳并触发重平衡，它会将失败消费者的分区重新分配给活跃消费者，以确保消息消费的继续进行，内部主题的持久状态可用于恢复消费。

让我们首先验证内部主题中已提交的偏移状态：

```shell
docker exec -it <CONTAINER_ID> kafka-console-consumer \
  --bootstrap-server localhost:9092 \
  --topic __consumer_offsets \
  --formatter "kafka.coordinator.group.GroupMetadataManager\$OffsetsMessageFormatter" \
  --from-beginning
```

该脚本使用特定格式以提高可读性，因为默认格式为二进制，并且该脚本记录来自主题的记录，显示消费者组(consumer-user-data)、主题(user-data)、分区(0和1)和偏移元数据(offset = 2)：

```text
[consumer-user-data,user-data,0]::OffsetAndMetadata(offset=2, leaderEpoch=Optional[0], metadata=, commitTimestamp=1726601656308, expireTimestamp=None)
[consumer-user-data,user-data,1]::OffsetAndMetadata(offset=0, leaderEpoch=Optional.empty, metadata=, commitTimestamp=1726601661314, expireTimestamp=None)
```

在这种情况下，分区0已收到所有消息，并且消费者提交了状态以跟踪进度/恢复。

接下来，让我们以生产者的身份推送更多消息来验证恢复行为：

```shell
docker exec -i <CONTAINER-ID> kafka-console-producer \
  --broker-list localhost:9092 \
  --topic user-data <<< '{"id": 3, "first_name": "Alice", "last_name": "Johnson"} 
{"id": 4, "first_name": "Michael", "last_name": "Brown"}'

```

然后，让我们重新启动先前终止的消费者，以检查它是否从最后一个已知偏移量恢复消费记录：

```shell
docker exec -it <CONTAINER_ID> kafka-console-consumer \
  --bootstrap-server localhost:9092 \
  --topic user-data \
  --consumer-property auto.offset.reset=earliest \
  --group consumer-user-data
```

即使auto.offset.reset设置为early，这应该也会记录用户ID为3和用户ID为4的记录，因为偏移量状态存储在内部主题中。最后，我们可以通过再次运行相同的命令来验证__consumer_offsets主题中的状态：

```text
[consumer-user-data, user-data, 1] :: OffsetAndMetadata(offset=0, leaderEpoch=Optional.empty, metadata=, commitTimestamp=1726611172398, expireTimestamp=None)
[consumer-user-data, user-data, 0] :: OffsetAndMetadata(offset=4, leaderEpoch=Optional[0], metadata=, commitTimestamp=1726611172398, expireTimestamp=None)
```

我们可以看到__consumer_offsets主题使用已提交的偏移量(值为4)进行了更新，从而有效地从最后提交的偏移量恢复了消费，因为状态保留在主题中。

## 5. 总结

在本文中，我们探讨了Kafka如何管理消费者偏移量以及当消费者第一次加入一个组时auto.offset.reset属性如何工作。

我们还了解了如何使用内部__consumer_offsets主题的状态在暂停或中断后恢复消费。