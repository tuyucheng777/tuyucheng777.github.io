---
layout: post
title:  Apache Kafka主题清除指南
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 概述

在本文中，我们将探讨一些**从Apache Kafka主题中清除数据的策略**。

## 2. 清理场景

在我们学习清理数据的策略之前，让我们先熟悉一个需要清除数据的简单场景。

### 2.1 场景

**Apache Kafka中的消息会在配置的[保留时间](https://www.baeldung.com/kafka-message-retention)后自动过期**，尽管如此，在某些情况下，我们可能希望立即删除消息。

假设在Kafka主题中生成消息的应用程序代码中出现了一个缺陷，当错误修复集成到Kafka主题中时，我们已经有大量损坏的消息可供消费。

这类问题在开发环境中最为常见，我们希望快速解决问题，因此，批量删除消息是明智之举。

### 2.2 模拟

为了模拟该场景，我们首先从Kafka安装目录创建一个purge-scenario主题：

```shell
$ bin/kafka-topics.sh \
  --create --topic purge-scenario --if-not-exists \
  --partitions 2 --replication-factor 1 \
  --zookeeper localhost:2181
```

接下来，让我们**使用[shuf](https://www.baeldung.com/linux/read-random-line-from-file#using-shuf)命令生成随机数据**并将其提供给kafka-console-producer.sh脚本：

```shell
$ /usr/bin/shuf -i 1-100000 -n 50000000 \
  | tee -a /tmp/kafka-random-data \
  | bin/kafka-console-producer.sh \
  --bootstrap-server=0.0.0.0:9092 \
  --topic purge-scenario
```

必须注意，我们使用[tee](https://www.baeldung.com/linux/tee-command)命令保存模拟数据以供日后使用。

最后，让我们验证消费者是否可以消费来自主题的消息：

```shell
$ bin/kafka-console-consumer.sh \
  --bootstrap-server=0.0.0.0:9092 \
  --from-beginning --topic purge-scenario \
  --max-messages 3
76696
49425
1744
Processed a total of 3 messages
```

## 3. 消息过期

purge-scenario主题中生成的消息默认[保留期](https://www.baeldung.com/kafka-message-retention#basics)为7天，为了清除消息，我们可以**暂时将主题级属性retention.ms重置为10秒**，并等待消息过期：

```shell
$ bin/kafka-configs.sh --alter \
  --add-config retention.ms=10000 \
  --bootstrap-server=0.0.0.0:9092 \
  --topic purge-scenario \
  && sleep 10
```

接下来，让我们验证该主题的消息是否已过期：

```shell
$ bin/kafka-console-consumer.sh  \
  --bootstrap-server=0.0.0.0:9092 \
  --from-beginning --topic purge-scenario \
  --max-messages 1 --timeout-ms 1000
[2021-02-28 11:20:15,951] ERROR Error processing message, terminating consumer process:  (kafka.tools.ConsoleConsumer$)
org.apache.kafka.common.errors.TimeoutException
Processed a total of 0 messages
```

最后，我们可以将主题的保留期恢复为原来的7天：

```shell
$ bin/kafka-configs.sh --alter \
  --add-config retention.ms=604800000 \
  --bootstrap-server=0.0.0.0:9092 \
  --topic purge-scenario
```

通过这种方法，Kafka将清除purge-scenario主题的所有分区中的消息。

## 4. 选择性记录删除

有时，我们可能希望**选择性地删除特定主题中一个或多个分区内的记录**，我们可以通过使用kafka-delete-records.sh脚本来满足此类需求。

首先，我们需要在delete-config.json配置文件中指定分区级别的偏移量。

让我们使用offset = -1清除partition = 1中的所有消息：

```json
{
    "partitions": [
        {
            "topic": "purge-scenario",
            "partition": 1,
            "offset": -1
        }
    ],
    "version": 1
}
```

接下来我们进行记录删除：

```shell
$ bin/kafka-delete-records.sh \
  --bootstrap-server localhost:9092 \
  --offset-json-file delete-config.json
```

我们可以验证仍然能够从partition = 0读取：

```shell
$ bin/kafka-console-consumer.sh \
  --bootstrap-server=0.0.0.0:9092 \
  --from-beginning --topic purge-scenario --partition=0 \
  --max-messages 1 --timeout-ms 1000
  44017
  Processed a total of 1 messages
```

但是，当我们从partition = 1读取时，将没有记录需要处理：

```shell
$ bin/kafka-console-consumer.sh \
  --bootstrap-server=0.0.0.0:9092 \
  --from-beginning --topic purge-scenario \
  --partition=1 \
  --max-messages 1 --timeout-ms 1000
[2021-02-28 11:48:03,548] ERROR Error processing message, terminating consumer process:  (kafka.tools.ConsoleConsumer$)
org.apache.kafka.common.errors.TimeoutException
Processed a total of 0 messages
```

## 5. 删除并重新创建主题

清除Kafka主题所有消息的另一种方法是删除并重新创建该主题，但是，**只有在启动Kafka服务器时将delete.topic.enable属性设置为true才可行**： 

```shell
$ bin/kafka-server-start.sh config/server.properties \
  --override delete.topic.enable=true
```

要删除主题，我们可以使用kafka-topics.sh脚本：

```shell
$ bin/kafka-topics.sh \
  --delete --topic purge-scenario \
  --zookeeper localhost:2181
Topic purge-scenario is marked for deletion.
Note: This will have no impact if delete.topic.enable is not set to true.
```

让我们通过列出主题来验证一下：

```shell
$ bin/kafka-topics.sh --zookeeper localhost:2181 --list
```

确认该主题不再列出后，我们现在可以继续重新创建它。

## 6. 总结

在本教程中，我们模拟了需要清除Apache Kafka主题的场景。此外，我们还探索了多种策略，可以完全清除主题，也可以跨分区选择性清除主题。