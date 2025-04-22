---
layout: post
title:  如何将Kafka与ElasticSearch连接
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 概述

在本教程中，我们将学习如何使用[Kafka Connector Sink](https://www.confluent.io/hub/confluentinc/kafka-connect-elasticsearch)将[Apache Kafka](https://kafka.apache.org/)连接到[ElasticSearch](https://www.elastic.co/elasticsearch)。

**Kafka项目提供了[Kafka Connect](https://www.baeldung.com/kafka-connectors-guide)，这是一个强大的工具，可以实现Kafka与外部数据存储源的无缝集成，而无需额外的代码或应用程序**。

## 2. 为什么使用Kafka Connect？

[Kafka Connect](https://www.baeldung.com/kafka-connectors-guide)提供了一种在Kafka和各种数据存储(包括ElasticSearch)之间轻松传输数据的方法，我们无需编写自定义应用程序来从Kafka读取数据并将其转储到ElasticSearch，因为它专为可扩展性、容错性和可管理性而设计；Kafka Connect的优势包括：

- 可扩展性：Kafka Connect可以以分布式模式运行，允许多个Worker分担负载
- 容错：自动处理故障，从而能够保持数据的正确性和完整性，这也使我们的管道更具弹性
- 自服务连接器：无需编写自定义集成组件或服务
- 高度可配置：通过简单的配置和API轻松设置和管理

## 3. Docker设置

让我们使用[Docker](https://www.docker.com/)来部署和管理我们的安装，这将简化设置并减少平台依赖问题，各个团队为所有必需的服务维护官方镜像。

我们将定义一个Docker Compose文件来启动以下服务：Kafka、Zookeeper、ElasticSearch和Kafka Connect。

第一步是创建Docker Compose文件：

```yaml
services:
    zookeeper:
        image: confluentinc/cp-zookeeper:latest
        environment:
            ZOOKEEPER_CLIENT_PORT: 2181
        ports:
            - "2181:2181"
    kafka:
        image: confluentinc/cp-kafka:latest
        depends_on:
            - zookeeper
        ports:
            - "9092:9092"
        environment:
            KAFKA_BROKER_ID: 1
            KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
            KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092
            KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
    elasticsearch:
        image: docker.elastic.co/elasticsearch/elasticsearch:8.6.0
        environment:
            discovery.type: single-node
            xpack.security.enabled: "false"
        ports:
            - "9200:9200"
    kafka-connect:
        image: confluentinc/cp-kafka-connect:latest
        depends_on:
            - kafka
        ports:
            - "8083:8083"
        environment:
            CONNECT_BOOTSTRAP_SERVERS: kafka:9092
            CONNECT_REST_ADVERTISED_HOST_NAME: kafka-connect
            CONNECT_GROUP_ID: kafka-connect-group
            CONNECT_CONFIG_STORAGE_TOPIC: connect-configs
            CONNECT_OFFSET_STORAGE_TOPIC: connect-offsets
            CONNECT_STATUS_STORAGE_TOPIC: connect-status
            CONNECT_KEY_CONVERTER: org.apache.kafka.connect.storage.StringConverter
            CONNECT_VALUE_CONVERTER: org.apache.kafka.connect.json.JsonConverter
            CONNECT_VALUE_CONVERTER_SCHEMAS_ENABLE: "false"
            CONNECT_CONFIG_STORAGE_REPLICATION_FACTOR: 1
            CONNECT_OFFSET_STORAGE_REPLICATION_FACTOR: 1
            CONNECT_STATUS_STORAGE_REPLICATION_FACTOR: 1
```

基本上，我们创建了Zookeeper来保存Kafka集群设置，并创建了一个Kafka Broker来处理主题数据，并将其指向Zookeeper服务。然后，我们还创建了一个ElasticSearch实例，为了简单起见，我们禁用了身份验证。

**我们的Kafka Connect基本属性只需极少的设置即可在本地运行Kafka连接器，它们设置了诸如复制因子、默认转换器和Kafka集群地址等内容**。要了解所有配置，请查看[官方文档页面](https://docs.confluent.io/platform/current/connect/references/allconfigs.html#standalone-worker-configuration)。

需要强调的是，上述配置不建议用于生产环境，它只是一份Kafka连接器的快速入门指南，本文不讨论弹性和容错能力。

一旦我们了解了Docker Compose文件的内容，就可以运行我们的服务：

```shell
# use -d to run in background 
docker compose up
```

容器运行后，我们需要手动安装Elasticsearch Sink连接器，因为它没有内置在Kafka连接器中。为此，我们运行以下命令：

```shell
docker exec -it kafka-elastic-search-kafka-connect-1 bash -c
  "confluent-hub install --no-prompt
  confluentinc/kafka-connect-elasticsearch:latest"
```

然后，接下来我们需要重新启动Kafka Connect服务，以便可以开始使用新的Sink：

```shell
docker restart kafka-elastic-search-kafka-connect-1
```

最后，为了检查一切是否按预期工作，我们可以调用Kafka Connect API来检查可用的接收器：

```shell
curl -s http://localhost:8083/connector-plugins | jq .
```

我们应该在响应中看到io.confluent.connect.elasticsearch.ElasticsearchSinkConnector。

## 4. Hello World

现在，让我们尝试发送第一条消息，该消息从Kafka流向ElasticSearch。为此，我们首先需要创建主题，如下所示：

```shell
docker exec -it $(
  docker ps --filter "name=kafka-elastic-search-kafka-1" --format "{{.ID}}"
) bash -c
  "kafka-topics --create --topic logs
  --bootstrap-server kafka:9092
  --partitions 1
  --replication-factor 1"
```

这将在Kafka Broker中创建我们的Kafka主题，接下来，让我们创建一个名为test-connector.json的文件：

```json
{
    "name": "elasticsearch-sink-connector-test",
    "config": {
        "connector.class": "io.confluent.connect.elasticsearch.ElasticsearchSinkConnector",
        "type.name": "_doc",
        "connection.url": "http://elasticsearch:9200",
        "tasks.max": "1",
        "topics": "logs",
        "key.ignore": "true",
        "schema.ignore": "true"
    }
}
```

该文件包含我们的Kafka连接器接收器及其配置，稍后我们将更好地理解这些配置，但我们需要知道这是通过API创建连接器所需的有效负载。该文件的前4个属性与本文中所有其他示例相同，因此为简单起见，我们将省略它们。

现在让我们创建我们的Kafka连接器：

```shell
curl -X POST -H 'Content-Type: application/json' --data @test-connector.json http://localhost:8083/connectors
```

通过这样做，我们的连接器就创建好了，它应该运行以确认我们可以使用JSON文件中定义的连接器名称并使用另一个Kafka Connect API查询它：

```shell
curl http://localhost:8083/connectors/elasticsearch-sink-connector-test/status
```

这应该可以确认我们的连接器已启动并正在运行。

现在我们知道连接器正在运行，让我们发送第一条消息，为了模拟Kafka生产者，我们可以运行以下代码：

```shell
docker exec -it $(docker ps --filter "name=kafka-elastic-search-kafka-1" --format "{{.ID}}")
  kafka-console-producer --broker-list kafka:9092 --topic logs
```

上面的命令创建了一个交互式提示符，允许我们将消息发送到Kafka日志主题，我们可以创建任何有效的JSON格式，然后按Enter键发送消息：

```json
{"message": "Hello word", "timestamp": "2025-02-05T12:00:00Z"}
{"message": "Test Kafka Connector", "timestamp": "2025-02-05T13:00:00Z"}
```

为了验证数据是否到达ElasticSearch，我们可以打开另一个终端并调用：

```shell
 curl -X GET "http://localhost:9200/logs/_search?pretty"
```

可以观察到，数据自动从Kafka主题流向ElasticSearch；只需将主题绑定到ElasticSearch索引即可。但是，此连接器的功能远不止于此。

## 5. Kafka Connect Elasticsearch Sink的高级场景

如前所述，Kafka连接器是功能强大的工具，它提供了用于集成数据存储和Kafka的强大机制。Kafka Connect提供了广泛的配置选项，允许用户定义数据管道以满足其用例。

处理分布式消息或数据流可能是一个非常复杂的问题，此工具旨在简化这一过程，让我们考虑一些常见的场景。

### 5.1 Kafka Avro消息发送到Elasticsearch

许多项目使用[Avro](https://www.baeldung.com/java-apache-avro)格式，因为它在序列化和模式演化方面非常高效。使用Avro时，Elasticsearch应该会根据模式自动检测字段类型，让我们来看看如何在与Elasticsearch集成时利用Avro模式。

首先，我们需要一个Avro模式注册表：

```yaml
schema-registry:
    image: confluentinc/cp-schema-registry:latest
    depends_on:
        - kafka
    ports:
        - "8081:8081"
    environment:
        SCHEMA_REGISTRY_KAFKASTORE_BOOTSTRAP_SERVERS: "kafka:9092"
        SCHEMA_REGISTRY_HOST_NAME: "schema-registry"
```

第一步是将这个新服务添加到我们的Docker Compose文件中并运行：

```shell
docker compose up -d
```

一旦我们有了模式注册表，我们就可以创建一个新主题来保存我们的Avro消息：

```shell
docker exec -it $(
  docker ps --filter "name=kafka-elastic-search-kafka-1" --format "{{.ID}}"
) bash -c
"kafka-topics --create
  --topic avro_logs
  --bootstrap-server kafka:9092
  --partitions 1
  --replication-factor 1"
```

下一步是创建一个名为avro-sink-config.json的新连接器配置文件：

```json
{
    "name": "avro-elasticsearch-sink",
    "config": {
        ...
        "key.ignore": "true",
        "schema.ignore": "false",
        "value.converter": "io.confluent.connect.avro.AvroConverter",
        "value.converter.schema.registry.url": "http://schema-registry:8081"
    }
}
```

让我们来看看这个文件：

- schema.ignore：这告诉连接器使用消息模式来创建ElasticSearch文档，在这种情况下，模式注册表定义将用于定义索引映射。
- value.converter：告诉连接器消息遵循Avro格式(io.confluent.connect.avro.AvroConverter)。
- value.converter.schema.registry.url：指定模式注册表位置。

了解了配置之后，我们可以继续创建连接器：

```shell
curl -X POST -H "Content-Type: application/json" --data @avro-sink-config.json http://localhost:8083/connectors
```

我们可以通过像之前一样检查状态来确认连接器是否正在运行，确认后，我们可以继续创建Avro消息：

```shell
docker exec -it $(
  docker ps --filter "name=kafka-elastic-search-schema-registry-1" --format "{{.ID}}"
) kafka-avro-console-producer
  --broker-list kafka:9092
  --topic avro_logs
  --property value.schema='{
   "type": "record",
   "name": "LogEntry",
   "fields": [
     {"name": "message", "type": "string"},
     {"name": "timestamp", "type": "long"}
   ]
 }'
```

提示准备好后，让我们发送一条测试消息，例如：

```json
{"message": "My Avro message", "timestamp": 1700000000}
```

最后，让我们看看ElasticSearch并查看我们的消息和映射：

```shell
curl -X GET "http://localhost:9200/avro_logs/_search?pretty"
```

并且：

```shell
curl -X GET "http://localhost:9200/avro_logs/_mapping"
```

我们可以看到，映射是使用模式创建的。

在进行下一个测试之前，让我们先清理一下：

```shell
curl -X DELETE "http://localhost:9200/avro_logs"
```

并且：

```shell
curl -X DELETE "http://localhost:8083/connectors/avro-elasticsearch-sink"
```

这将删除Kafka连接器和ElasticSearch索引。

### 5.2 时间戳转换

让我们使用一个新的连接器配置文件timestamp-transform-sink.json，自动将纪元时间戳转换为[ISO-8601](https://en.wikipedia.org/wiki/ISO_8601)格式，配置如下：

```json
{
    "name": "timestamp-transform-sink",
    "config": {
        ...
        "topics": "epoch_logs",
        "key.ignore": "true",
        "schema.ignore": "true",
        "transforms": "TimestampConverter",
        "transforms.TimestampConverter.type":"org.apache.kafka.connect.transforms.TimestampConverter$Value",
        "transforms.TimestampConverter.field": "timestamp",
        "transforms.TimestampConverter.target.type": "string",
        "transforms.TimestampConverter.format": "yyyy-MM-dd'T'HH:mm:ssZ"
    }
}
```

让我们看看以下亮点：

- transforms：定义转换名称，以便应用于我们的数据处理管道
- TimestampConverter：定义一个转换，从消息中提取一个字段并使用特定格式进行转换

然后，我们创建连接器：

```shell
curl -X POST -H "Content-Type: application/json" --data @timestamp-transform-sink.json http://localhost:8083/connectors 
```

让我们测试一下：

```shell
docker exec -it $(
  docker ps --filter "name=kafka-elastic-search-kafka-1" --format "{{.ID}}"
) kafka-console-producer
  --broker-list kafka:9092
  --topic epoch_logs
```

发送消息：

```json
{"message": "Timestamp transformation", "timestamp": 1700000000000}
```

为了确认这一点，让我们运行：

```shell
curl -X GET "http://localhost:9200/epoch_logs/_search?pretty"
```

并且：

```shell
curl -X GET "http://localhost:9200/epoch_logs/_mapping"
```

在这里，我们看到了时间戳是如何转换的，并且ElasticSearch正确地将字段映射到数据类型。

### 5.3 忽略和记录错误

**默认情况下，连接器具有一个名为errors.tolerance的属性，其定义为none，这意味着当发生错误时，连接器将停止处理**。然而，有时在实时处理时，这可能不是一个好主意。因此，现在让我们看看如何让连接器忽略错误并继续处理。

再次，我们首先创建一个主题：

```shell
docker exec -it $(
  docker ps --filter "name=kafka-elastic-search-kafka-1" --format "{{.ID}}"
) bash -c
"kafka-topics --create
  --topic test-error-handling
  --bootstrap-server kafka:9092
  --partitions 1
  --replication-factor 1"
```

然后，我们将配置连接器error-handling-sink-config.json：

```json
{
    "name": "error-handling-elasticsearch-sink",
    "config": {
        ...
        "topics": "test-error-handling",
        "key.ignore": "true",
        "schema.ignore": "true",
        "behavior.on.malformed.documents": "warn",
        "behavior.on.error": "LOG",
        "errors.tolerance": "all",
        "errors.log.enable": "true",
        "errors.log.include.messages": "true"
    }
}
```

主要属性：

- behavior.on.malformed.documents：记录无效文档而不是停止连接器
- error.tolerance：允许Kafka Connect在出现错误的情况下继续处理有效消息
- error.log.enable：将错误记录到Kafka Connect日志中
- error.log.include.messages：在日志中包含实际的问题消息

现在我们注册连接器：

```shell
curl -X POST -H "Content-Type: application/json" --data @error-handling-sink-config.json http://localhost:8083/connectors
```

然后我们打开控制台来测试一下：

```shell
docker exec -it $(
  docker ps --filter "name=kafka-elastic-search-kafka-1" --format "{{.ID}}"
) kafka-console-producer
  --broker-list kafka:9092
  --topic test-error-handling
```

接下来，我们发送以下消息：

```json
{"message": "Ok", "timestamp": "2025-02-08T12:00:00Z"}
{"message": "NOK", "timestamp": "invalid_timestamp"}
{"message": "Ok Again", "timestamp": "2025-02-08T13:00:00Z"}
```

最后，让我们检查一下ElasticSearch：

```shell
curl -X GET "http://localhost:9200/test-error-handling/_search?pretty"
```

我们可以确认只有第一条和最后一条消息被编入了索引，现在，我们来检查一下连接器日志：

```shell
docker logs kafka-elastic-search-kafka-connect-1 | grep "ERROR"
```

日志显示处理主题偏移量1时出现错误，但是，连接器状态为正在运行，这正是我们希望发生的。

### 5.4 在Elasticsearch中微调批量处理和刷新

在高效处理大规模数据流时，需要考虑许多变量。因此，我们这次不会测试特定场景。相反，让我们花点时间了解一下ElasticSearch Connector Sink提供的不同参数，以便我们根据用例进行微调。

这些配置的组合将直接影响我们的效率和可扩展性，因此，必须合理设计一些容量规划，并针对不同的配置组合执行该规划，以了解它们如何影响我们的工作负载。现在让我们检查与数据提取和刷新相关的最相关的配置：

| 参数名称| 默认值                    |
| ------------------ |------------------------|
| batch.size| 2000(范围从1到1000000)     |
| bulk.size.bytes| 5兆字节(可达到GB)            |
| max.in.flight.requests| 5(可以从1到1000)           |
| max.buffered.records| 20000(范围从1到2147483647) |
|linger.ms| 1(可以从0到604800000)      |
| flush.timeout.ms| 3分钟(最长可达数小时)           |
| flush.synchronously| false                  |
| max.retries| 5                      |
| retry.backoff.ms| 100                    |
| connection.compression| false                  |
| write.method| INSERT(也可以是UPSERT)     |
| read.timeout.ms| 3分钟(最长可达数小时)           |

有关详尽列表，我们可以查看[官方文档页面](https://docs.confluent.io/kafka-connectors/elasticsearch/current/configuration_options.html)。

## 6. 总结

按照本指南，我们使用Kafka Connect Sink成功建立了从Kafka到Elasticsearch的近实时数据管道，额外的测试场景确保了我们能够灵活地处理各种实际数据转换和数据提取策略。我们还了解了该连接器提供的所有控件和机制，以便我们更好地调整流管道。