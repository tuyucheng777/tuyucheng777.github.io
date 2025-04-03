---
layout: post
title:  使用MQTT和MongoDB的Kafka Connect示例
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 概述

在[之前](https://www.baeldung.com/kafka-connectors-guide)的文章中，我们对Kafka Connect进行了简单介绍，包括不同类型的连接器、Connect的基本功能以及REST API。

在本教程中，我们将使用Kafka连接器构建更“真实世界”的示例。

**我们将使用连接器通过MQTT收集数据，并将收集的数据写入MongoDB**。

## 2. 使用Docker进行设置

我们将使用[Docker Compose](https://docs.docker.com/compose/)设置基础架构，其中包括一个MQTT代理作为源、Zookeeper、一个Kafka代理以及Kafka Connect作为中间件，最后是一个包含GUI工具的MongoDB实例作为接收器。

### 2.1 连接器安装

我们示例所需的连接器(MQTT源以及MongoDB接收器连接器)不包含在普通Kafka或Confluent平台中。

正如我们在上一篇文章中讨论的那样，我们可以从Confluent中心下载连接器([MQTT](https://www.confluent.io/connector/kafka-connect-mqtt/)和[MongoDB](https://www.confluent.io/connector/kafka-connect-mongodb-sink/))。之后，我们必须将jar解压到一个文件夹中，我们将在下一节中将其挂载到Kafka Connect容器中。

让我们使用文件夹/tmp/custom/jars来实现这一点，我们必须在下一节中启动Compose栈之前将jar移到那里，因为Kafka Connect在启动期间会在线加载连接器。

### 2.2 Docker Compose文件

我们将我们的设置描述为一个简单的Docker Compose文件，它由6个容器组成：

```yaml
version: '3.3'

services:
    mosquitto:
        image: eclipse-mosquitto:1.5.5
        hostname: mosquitto
        container_name: mosquitto
        expose:
            - "1883"
        ports:
            - "1883:1883"
    zookeeper:
        image: zookeeper:3.4.9
        restart: unless-stopped
        hostname: zookeeper
        container_name: zookeeper
        ports:
            - "2181:2181"
        environment:
            ZOO_MY_ID: 1
            ZOO_PORT: 2181
            ZOO_SERVERS: server.1=zookeeper:2888:3888
        volumes:
            - ./zookeeper/data:/data
            - ./zookeeper/datalog:/datalog
    kafka:
        image: confluentinc/cp-kafka:5.1.0
        hostname: kafka
        container_name: kafka
        ports:
            - "9092:9092"
        environment:
            KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
            KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092,PLAINTEXT_HOST://localhost:29092
            KAFKA_ZOOKEEPER_CONNECT: "zookeeper:2181"
            KAFKA_BROKER_ID: 1
            KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
        volumes:
            - ./kafka/data:/var/lib/kafka/data
        depends_on:
            - zookeeper
    kafka-connect:
        image: confluentinc/cp-kafka-connect:5.1.0
        hostname: kafka-connect
        container_name: kafka-connect
        ports:
            - "8083:8083"
        environment:
            CONNECT_BOOTSTRAP_SERVERS: "kafka:9092"
            CONNECT_REST_ADVERTISED_HOST_NAME: connect
            CONNECT_REST_PORT: 8083
            CONNECT_GROUP_ID: compose-connect-group
            CONNECT_CONFIG_STORAGE_TOPIC: docker-connect-configs
            CONNECT_OFFSET_STORAGE_TOPIC: docker-connect-offsets
            CONNECT_STATUS_STORAGE_TOPIC: docker-connect-status
            CONNECT_KEY_CONVERTER: org.apache.kafka.connect.json.JsonConverter
            CONNECT_VALUE_CONVERTER: org.apache.kafka.connect.json.JsonConverter
            CONNECT_INTERNAL_KEY_CONVERTER: "org.apache.kafka.connect.json.JsonConverter"
            CONNECT_INTERNAL_VALUE_CONVERTER: "org.apache.kafka.connect.json.JsonConverter"
            CONNECT_CONFIG_STORAGE_REPLICATION_FACTOR: "1"
            CONNECT_OFFSET_STORAGE_REPLICATION_FACTOR: "1"
            CONNECT_STATUS_STORAGE_REPLICATION_FACTOR: "1"
            CONNECT_PLUGIN_PATH: '/usr/share/java,/etc/kafka-connect/jars'
            CONNECT_CONFLUENT_TOPIC_REPLICATION_FACTOR: 1
        volumes:
            - /tmp/custom/jars:/etc/kafka-connect/jars
        depends_on:
            - zookeeper
            - kafka
            - mosquitto
    mongo-db:
        image: mongo:4.0.5
        hostname: mongo-db
        container_name: mongo-db
        expose:
            - "27017"
        ports:
            - "27017:27017"
        command: --bind_ip_all --smallfiles
        volumes:
            - ./mongo-db:/data
    mongoclient:
        image: mongoclient/mongoclient:2.2.0
        container_name: mongoclient
        hostname: mongoclient
        depends_on:
            - mongo-db
        ports:
            - 3000:3000
        environment:
            MONGO_URL: "mongodb://mongo-db:27017"
            PORT: 3000
        expose:
            - "3000"
```

mosquitto容器提供了一个基于Eclipse Mosquitto的简单MQTT代理。

容器zookeeper和kafka定义单节点Kafka集群。

kafka-connect以分布式模式定义我们的Connect应用程序。

最后，mongo-db定义了我们的接收数据库，以及基于Web的mongoclient，它可以帮助我们验证发送的数据是否正确到达数据库。

我们可以使用以下命令启动容器栈：

```shell
docker-compose up
```

## 3. 连接器配置

由于Kafka Connect现已启动并运行，现在我们可以配置连接器。

### 3.1 配置源连接器

让我们使用REST API配置源连接器：

```shell
curl -d @<path-to-config-file>/connect-mqtt-source.json -H "Content-Type: application/json" -X POST http://localhost:8083/connectors
```

我们的connect-mqtt-source.json文件如下所示：

```json
{
    "name": "mqtt-source",
    "config": {
        "connector.class": "io.confluent.connect.mqtt.MqttSourceConnector",
        "tasks.max": 1,
        "mqtt.server.uri": "tcp://mosquitto:1883",
        "mqtt.topics": "tuyucheng",
        "kafka.topic": "connect-custom",
        "value.converter": "org.apache.kafka.connect.converters.ByteArrayConverter",
        "confluent.topic.bootstrap.servers": "kafka:9092",
        "confluent.topic.replication.factor": 1
    }
}
```

有一些属性我们之前没有使用过：

- mqtt.server.uri是我们的连接器将连接到的端点
- mqtt.topics是我们的连接器将订阅的MQTT主题
- kafka.topic定义连接器将接收的数据发送到的Kafka主题
- value.converter定义将应用于接收的有效负载的转换器，我们需要ByteArrayConverter，因为MQTT连接器默认使用Base64，而我们想使用纯文本
- confluent.topic.bootstrap.servers对于最新版本的连接器需要
- confluent.topic.replication.factor定义了Confluent内部主题的复制因子-由于我们的集群中只有一个节点，因此我们必须将该值设置为1

### 3.2 测试源连接器

让我们通过向MQTT代理发布一条简短消息来进行快速测试：

```shell
docker run \
-it --rm --name mqtt-publisher --network 04_custom_default \
efrecon/mqtt-client \
pub -h mosquitto  -t "tuyucheng" -m "{\"id\":1234,\"message\":\"This is a test\"}"
```

如果我们监听这个主题connect-custom：

```shell
docker run \
--rm \
confluentinc/cp-kafka:5.1.0 \
kafka-console-consumer --network 04_custom_default --bootstrap-server kafka:9092 --topic connect-custom --from-beginning
```

然后我们就会看到我们的测试消息。

### 3.3 设置接收器连接器

接下来，我们需要接收器连接器，让我们再次使用REST API：

```shell
curl -d @<path-to-config file>/connect-mongodb-sink.json -H "Content-Type: application/json" -X POST http://localhost:8083/connectors
```

connect-mongodb-sink.json文件如下所示：

```json
{
    "name": "mongodb-sink",
    "config": {
        "connector.class": "at.grahsl.kafka.connect.mongodb.MongoDbSinkConnector",
        "tasks.max": 1,
        "topics": "connect-custom",
        "mongodb.connection.uri": "mongodb://mongo-db/test?retryWrites=true",
        "mongodb.collection": "MyCollection",
        "key.converter": "org.apache.kafka.connect.json.JsonConverter",
        "key.converter.schemas.enable": false,
        "value.converter": "org.apache.kafka.connect.json.JsonConverter",
        "value.converter.schemas.enable": false
    }
}
```

这里有以下MongoDB特定的属性：

- mongodb.connection.uri包含我们的MongoDB实例的连接字符串
- mongodb.collection定义集合
- 由于MongoDB连接器需要JSON，因此我们必须为key.converter和value.converter设置JsonConverter
- 而且我们还需要MongoDB的无模式JSON，因此我们必须将key.converter.schemas.enable和value.converter.schemas.enable设置为false

### 3.4 测试接收器连接器

由于我们的主题connect-custom已经包含来自MQTT连接器测试的消息，因此**MongoDB连接器应该在创建后直接获取它们**。

因此，我们应该立即在MongoDB中找到它们。**我们可以使用Web界面，打开URL [http://localhost:3000/](http://localhost:3000/)**。登录后，我们可以在左侧选择我们的MyCollection，点击Execute，然后应该会显示我们的测试消息。

### 3.5 端到端测试

现在，我们可以使用MQTT客户端发送任何JSON结构：

```json
{
    "firstName": "John",
    "lastName": "Smith",
    "age": 25,
    "address": {
        "streetAddress": "21 2nd Street",
        "city": "New York",
        "state": "NY",
        "postalCode": "10021"
    },
    "phoneNumber": [{
        "type": "home",
        "number": "212 555-1234"
    }, {
        "type": "fax",
        "number": "646 555-4567"
    }],
    "gender": {
        "type": "male"
    }
}
```

MongoDB支持无模式的JSON文档，并且当我们为转换器禁用模式时，任何结构都会立即通过我们的连接器链并存储在数据库中。

再次，我们可以使用[http://localhost:3000/](http://localhost:3000/)上的Web界面。

### 3.6 清理

完成后，我们可以清理实验并移除两个连接器：

```shell
curl -X DELETE http://localhost:8083/connectors/mqtt-source
curl -X DELETE http://localhost:8083/connectors/mongodb-sink
```

之后，我们可以使用Ctrl + C关闭Compose栈。

## 4. 总结

在本教程中，我们使用Kafka Connect构建了一个示例，通过MQTT收集数据，并将收集的数据写入MongoDB。