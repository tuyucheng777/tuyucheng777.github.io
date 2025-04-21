---
layout: post
title:  在Apache Kafka中配置消息保留期
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 概述

当生产者向Apache Kafka发送消息时，它会将其附加到日志文件中并保留配置的持续时间。

在本教程中，我们将学习**为Kafka主题配置基于时间的消息保留属性**。

## 2. 基于时间的留存

**有了保留期属性，消息就拥有了TTL(生存时间)**，到期后，消息将被标记为删除，从而释放磁盘空间。

相同的保留期属性适用于给定Kafka主题中的所有消息，此外，**我们可以在创建主题之前设置这些属性，也可以在运行时针对现有主题进行更改**。

在下面的部分中，我们将学习如何通过代理配置来调整新主题的保留期，以及如何**通过主题级配置在运行时对其进行控制**。

## 3. 服务器级配置

**Apache Kafka支持服务器级保留策略，我们可以通过配置以下3个基于时间的配置属性之一来调整该策略**：

- log.retention.hours
- log.retention.minutes
- log.retention.ms

需要注意的是，Kafka会用精度更高的值覆盖精度较低的值，因此，**log.retention.ms的优先级最高**。

### 3.1 基础知识

首先，让我们通过从[Apache Kafka目录](https://kafka.apache.org/documentation/#quickstart)执行[grep](https://www.baeldung.com/linux/grep-sed-awk-differences#grep)命令来检查保留期的默认值：

```shell
$ grep -i 'log.retention.[hms].*\=' config/server.properties
log.retention.hours=168
```

我们可以在这里注意到**默认保留时间是7天**。

为了仅保留消息十分钟，我们可以在config/server.properties中设置log.retention.minutes属性的值：

```properties
log.retention.minutes=10
```

### 3.2 新主题保留期

Apache Kafka软件包包含几个可用于执行管理任务的Shell脚本，我们将使用它们创建一个辅助脚本functions.sh，并在本教程中用到它。

**我们首先在functions.sh中添加两个函数，分别用于创建主题和描述其配置**：

```shell
function create_topic {
    topic_name="$1"
    bin/kafka-topics.sh --create --topic ${topic_name} --if-not-exists \
      --partitions 1 --replication-factor 1 \
      --zookeeper localhost:2181
}

function describe_topic_config {
    topic_name="$1"
    ./bin/kafka-configs.sh --describe --all \
      --bootstrap-server=0.0.0.0:9092 \
      --topic ${topic_name}
}
```

接下来，让我们创建两个独立的脚本，create-topic.sh和get-topic-retention-time.sh：

```shell
bash-5.1# cat create-topic.sh
#!/bin/bash
. ./functions.sh
topic_name="$1"
create_topic "${topic_name}"
exit $?
```

```shell
bash-5.1# cat get-topic-retention-time.sh
#!/bin/bash
. ./functions.sh
topic_name="$1"
describe_topic_config "${topic_name}" | awk 'BEGIN{IFS="=";IRS=" "} /^[ ]*retention.ms/{print $1}'
exit $?
```

需要注意的是，describe_topic_config会列出该主题的所有配置属性，因此，我们使用[awk](https://www.baeldung.com/linux/awk-guide)单行命令为retention.ms属性添加了一个过滤器。

最后，让我们[启动Kafka环境](https://kafka.apache.org/documentation/#quickstart_startserver)并验证新示例主题的保留期配置：

```shell
bash-5.1# ./create-topic.sh test-topic
Created topic test-topic.
bash-5.1# ./get-topic-retention-time.sh test-topic
retention.ms=600000
```

创建并描述主题后，我们会注意到**retention.ms设置为600000(十分钟)**，这实际上是从我们之前在server.properties文件中定义的log.retention.minutes属性派生而来的。

## 4. 主题级配置

**代理服务器启动后，log.retention.{hours|minutes|ms}服务器级属性将变为只读**，另一方面，我们可以访问retention.ms属性，并在主题级别对其进行调整。

让我们在functions.sh脚本中添加一个方法来配置主题的属性：

```shell
function alter_topic_config {
    topic_name="$1"
    config_name="$2"
    config_value="$3"
    ./bin/kafka-configs.sh --alter \
      --add-config ${config_name}=${config_value} \
      --bootstrap-server=0.0.0.0:9092 \
      --topic ${topic_name}
}
```

然后，我们可以在alter-topic-config.sh脚本中使用它：

```shell
#!/bin/sh
. ./functions.sh

alter_topic_retention_config $1 $2 $3
exit $?
```

最后，让我们将test-topic的保留时间设置为五分钟并进行验证：

```shell
bash-5.1# ./alter-topic-config.sh test-topic retention.ms 300000
Completed updating config for topic test-topic.

bash-5.1# ./get-topic-retention-time.sh test-topic
retention.ms=300000
```

## 5. 验证

到目前为止，我们已经了解了如何在Kafka主题中配置消息的保留期限，现在是时候验证消息在保留期超时后是否确实过期了。

### 5.1 生产者-消费者

让我们在functions.sh中添加produce_message和consumer_message函数，在内部，它们分别使用kafka-console-producer.sh和kafka-console-consumer.sh来生成/消费消息：

```shell
function produce_message {
    topic_name="$1"
    message="$2"
    echo "${message}" | ./bin/kafka-console-producer.sh \
    --bootstrap-server=0.0.0.0:9092 \
    --topic ${topic_name}
}

function consume_message {
    topic_name="$1"
    timeout="$2"
    ./bin/kafka-console-consumer.sh \
    --bootstrap-server=0.0.0.0:9092 \
    --from-beginning \
    --topic ${topic_name} \
    --max-messages 1 \
    --timeout-ms $timeout
}
```

我们必须注意，**消费者总是从一开始就读取消息**，因为我们需要一个**读取Kafka中任何可用消息的消费者**。

接下来，让我们创建一个独立的消息生产者：

```shell
bash-5.1# cat producer.sh
#!/bin/sh
. ./functions.sh
topic_name="$1"
message="$2"

produce_message ${topic_name} ${message}
exit $?
```

最后，创建一个独立的消息消费者：

```shell
bash-5.1# cat consumer.sh
#!/bin/sh
. ./functions.sh
topic_name="$1"
timeout="$2"

consume_message ${topic_name} $timeout
exit $?
```

### 5.2 消息过期

现在我们已经准备好基本设置，让我们生成一条消息并立即消费它两次：

```shell
bash-5.1# ./producer.sh "test-topic-2" "message1"
bash-5.1# ./consumer.sh test-topic-2 10000
message1
Processed a total of 1 messages
bash-5.1# ./consumer.sh test-topic-2 10000
message1
Processed a total of 1 messages
```

因此，我们可以看到消费者正在重复消费任何可用的消息。

现在，让我们引入五分钟的睡眠延迟，然后尝试消费该消息：

```shell
bash-5.1# sleep 300 && ./consumer.sh test-topic 10000
[2021-02-06 21:55:00,896] ERROR Error processing message, terminating consumer process:  (kafka.tools.ConsoleConsumer$)
org.apache.kafka.common.errors.TimeoutException
Processed a total of 0 messages
```

正如预期的那样，**消费者没有找到任何可供消费的消息，因为该消息已超出其保留期**。

## 6. 限制

在内部，Kafka代理维护另一个名为log.retention.check.interval.ms的属性，此属性决定检查消息是否过期的频率。

因此，为了保持保留策略有效，我们必须确保log.retention.check.interval.ms的值低于任何给定主题的retention.ms属性值。

## 7. 总结

在本教程中，我们探索了Apache Kafka，以了解基于时间的消息保留策略。在此过程中，我们创建了简单的Shell脚本来简化管理任务，之后，我们创建了独立的消费者和生产者，用于验证保留期过后的消息是否过期。