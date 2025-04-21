---
layout: post
title:  Kafka连接器简介
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 概述

Apache Kafka是一个分布式流平台，在之前的教程中，我们讨论了如何[使用Spring实现Kafka消费者和生产者](https://www.baeldung.com/spring-kafka)。

在本教程中，我们将学习如何使用Kafka连接器。

主要关注：

- 不同类型的Kafka连接器
- Kafka Connect的功能和模式
- 使用属性文件以及REST API进行连接器配置

## 2. Kafka Connect和Kafka Connectors的基础知识

**Kafka Connect是一个使用所谓的连接器将Kafka与外部系统(如数据库、键值存储、搜索索引和文件系统)连接起来的框架**。

**Kafka连接器是现成的组件，可以帮助我们将数据从外部系统导入Kafka主题，并将数据从Kafka主题导出到外部系统**。我们可以将现有的连接器实现用于常见的数据源和接收器，也可以实现我们自己的连接器。

源连接器从系统收集数据，源系统可以是整个数据库、流表或消息代理。源连接器还可以将来自应用服务器的指标收集到Kafka主题中，从而使数据可以低延迟地进行流处理。

接收器连接器将来自Kafka主题的数据传送到其他系统，这些系统可能是索引(例如Elasticsearch)、批处理系统(例如Hadoop)或任何类型的数据库。

**有些连接器由社区维护，而其他连接器则由Confluent或其合作伙伴支持。实际上，我们可以找到大多数流行系统的连接器，例如S3、JDBC和Cassandra，仅举几例**。

## 3. 特点

Kafka Connect功能包括：

- 用于将外部系统与Kafka连接的框架：**简化连接器的开发、部署和管理**
- 分布式和独立模式：**利用Kafka的分布式特性帮助我们部署大型集群，以及用于开发、测试和小规模生产部署的设置**
- REST接口：我们可以使用REST API管理连接器
- 自动偏移管理：**Kafka Connect帮助我们处理偏移提交过程**，省去了我们手动实现连接器开发中这个容易出错的部分的麻烦
- 默认分布式和可扩展：**Kafka Connect使用现有的组管理协议；我们可以添加更多工作器来扩展Kafka Connect集群**
- 流式和批处理集成：Kafka Connect是连接流式和批处理数据系统以及Kafka现有功能的理想解决方案
- 转换：这些使我们能够对单个消息进行简单而轻松的修改

## 4. 设置

**我们不使用普通的Kafka发行版，而是下载Confluent Platform，这是Kafka背后的公司Confluent提供的Kafka发行版。与普通的Kafka相比，Confluent Platform附带了一些额外的工具和客户端，以及一些额外的预构建连接器**。

对于我们的情况来说，开源版本就足够了，可以在[Confluent网站](https://www.confluent.io/download/)上找到。

## 5. 快速启动Kafka Connect

首先，我们来讨论一下Kafka Connect的原理，**使用它的最基本的连接器，也就是文件源连接器和文件接收器连接器**。

方便的是，Confluent Platform配备了这两种连接器以及参考配置。

### 5.1 源连接器配置

对于源连接器，参考配置可在$CONFLUENT_HOME/etc/kafka/connect-file-source.properties中找到：

```properties
name=local-file-source
connector.class=FileStreamSource
tasks.max=1
topic=connect-test
file=test.txt
```

此配置具有所有源连接器共有的一些属性：

- name是用户指定的连接器实例名称
- Connector.class指定实现类，基本上是连接器的类型
- task.max指定源连接器应并行运行的实例数
- topic定义连接器应发送输出的主题

**在这种情况下，我们还有一个连接器特定的属性**：

- **file定义连接器应从中读取输入的文件**

为了使其正常工作，让我们创建一个包含一些内容的基本文件：

```shell
echo -e "foo\nbar\n" > $CONFLUENT_HOME/test.txt
```

**请注意，工作目录是$CONFLUENT_HOME**。

### 5.2 接收器连接器配置

对于我们的接收器连接器，我们将使用$CONFLUENT_HOME/etc/kafka/connect-file-sink.properties处的参考配置：

```properties
name=local-file-sink
connector.class=FileStreamSink
tasks.max=1
file=test.sink.txt
topics=connect-test
```

从逻辑上讲，它包含完全相同的参数，**不过这次connector.class指定了接收器连接器的实现，而file是连接器应该写入内容的位置**。

### 5.3 Worker配置

最后，我们必须配置Connect Worker，它将整合我们的两个连接器，并完成从源连接器读取和写入接收器连接器的工作。

为此，我们可以使用$CONFLUENT_HOME/etc/kafka/connect-standalone.properties：

```properties
bootstrap.servers=localhost:9092
key.converter=org.apache.kafka.connect.json.JsonConverter
value.converter=org.apache.kafka.connect.json.JsonConverter
key.converter.schemas.enable=false
value.converter.schemas.enable=false
offset.storage.file.filename=/tmp/connect.offsets
offset.flush.interval.ms=10000
plugin.path=/share/java
```

请注意，plugin.path可以保存路径列表，其中有可用的连接器实现。

由于我们将使用与Kafka捆绑在一起的连接器，因此我们可以将plugin.path设置为$CONFLUENT_HOME/share/java。使用Windows时，可能需要在此处提供绝对路径。

对于其他参数，我们可以保留默认值：

- bootstrap.servers包含Kafka代理的地址
- key.converter和value.converter定义转换器类，当数据从源流入Kafka，然后从Kafka流向接收器时，转换器类会对其进行序列化和反序列化
- key.converter.schemas.enable和value.converter.schemas.enable是转换器特定的设置
- **offset.storage.file.filename是在独立模式下运行Connect时最重要的设置：它定义了Connect应将其偏移数据存储在何处**
- offset.flush.interval.ms定义Worker尝试提交任务偏移量的间隔

参数列表已经相当成熟，因此请查看[官方文档](http://kafka.apache.org/documentation/#connectconfigs)以获取完整列表。

### 5.4 独立模式下的Kafka Connect

有了它，我们就可以开始第一个连接器设置：

```shell
$CONFLUENT_HOME/bin/connect-standalone \
  $CONFLUENT_HOME/etc/kafka/connect-standalone.properties \
  $CONFLUENT_HOME/etc/kafka/connect-file-source.properties \
  $CONFLUENT_HOME/etc/kafka/connect-file-sink.properties
```

首先，我们可以使用命令行检查主题的内容：

```shell
$CONFLUENT_HOME/bin/kafka-console-consumer --bootstrap-server localhost:9092 --topic connect-test --from-beginning
```

我们可以看到，源连接器从test.txt文件中获取数据，将其转换为JSON，然后将其发送给Kafka：

```json
{"schema":{"type":"string","optional":false},"payload":"foo"}
{"schema":{"type":"string","optional":false},"payload":"bar"}
```

而且，如果我们查看文件夹$CONFLUENT_HOME，我们可以看到这里创建了一个文件test.sink.txt：

```shell
cat $CONFLUENT_HOME/test.sink.txt
foo
bar
```

当接收器连接器从有效负载属性中提取值并将其写入目标文件时，test.sink.txt中的数据具有原始test.txt文件的内容。

现在让我们向test.txt添加更多行。

**当我们这样做时，我们会看到源连接器自动检测到这些变化**。

**我们只需要确保在末尾插入换行符，否则源连接器将不会考虑最后一行**。

此时，让我们停止Connect进程，因为我们将在几行代码中以分布式模式启动Connect。

## 6. Connect的REST API

到目前为止，我们通过命令行传递属性文件来进行所有配置。**但是，由于Connect旨在作为服务运行，因此还有一个REST API可用**。

默认情况下，它位于http://localhost:8083，一些端点是：

- GET/connectors：返回所有正在使用的连接器的列表
- GET/connectors/{name}：返回有关特定连接器的详细信息
- POST/connectors：创建一个新的连接器；请求主体应该是一个JSON对象，其中包含一个字符串名称字段和一个带有连接器配置参数的对象配置字段
- GET/connectors/{name}/status：返回连接器的当前状态-包括它是否正在运行、失败或暂停，它被分配给哪个工作器、如果失败则返回错误信息以及其所有任务的状态
- DELETE/connectors/{name}：删除连接器，正常停止所有任务并删除其配置
- GET/connector-plugins：返回Kafka Connect集群中安装的连接器插件列表

[官方文档](http://kafka.apache.org/documentation/#connect_rest)提供了所有端点的列表。

**在下一节中，我们将使用REST API来创建新的连接器**。

## 7. 分布式模式下的Kafka Connect

独立模式非常适合开发和测试以及较小的设置，**但是，如果我们想充分利用Kafka的分布式特性，我们必须以分布式模式启动Connect**。

这样，连接器设置和元数据就存储在Kafka主题中，而不是文件系统中。因此，工作节点实际上是无状态的。

### 7.1 启动连接器

分布式模式的参考配置可以在$CONFLUENT_HOME/etc/kafka/connect-distributed.properties找到。

**参数与独立模式基本相同，只有少数不同之处**：

- group.id定义Connect集群组的名称，该值必须与任何消费者组ID不同
- offset.storage.topic、config.storage.topic和status.storage.topic为这些设置定义主题，对于每个主题，我们还可以定义一个复制因子

再次，[官方文档](http://kafka.apache.org/documentation/#connectconfigs)提供了所有参数的列表。

我们可以按如下方式以分布式模式启动Connect：

```shell
$CONFLUENT_HOME/bin/connect-distributed $CONFLUENT_HOME/etc/kafka/connect-distributed.properties
```

### 7.2 使用REST API添加连接器

现在，与独立启动命令相比，我们没有传递任何连接器配置作为参数。相反，我们必须使用REST API创建连接器。

为了设置我们之前的示例，我们必须向http://localhost:8083/connectors发送两个POST请求，其中包含以下JSON结构。

首先，我们需要将源连接器POST的主体创建为JSON文件，这里，我们将其命名为connect-file-source.json：

```json
{
    "name": "local-file-source",
    "config": {
        "connector.class": "FileStreamSource",
        "tasks.max": 1,
        "file": "test-distributed.txt",
        "topic": "connect-distributed"
    }
}
```

**请注意，这看起来与我们第一次使用的参考配置文件非常相似**。

然后我们调用POST：

```shell
curl -d @"$CONFLUENT_HOME/connect-file-source.json" \
  -H "Content-Type: application/json" \
  -X POST http://localhost:8083/connectors
```

然后，我们对接收器连接器执行相同的操作，文件为connect-file-sink.json：

```json
{
    "name": "local-file-sink",
    "config": {
        "connector.class": "FileStreamSink",
        "tasks.max": 1,
        "file": "test-distributed.sink.txt",
        "topics": "connect-distributed"
    }
}
```

并像以前一样执行POST：

```shell
curl -d @$CONFLUENT_HOME/connect-file-sink.json \
  -H "Content-Type: application/json" \
  -X POST http://localhost:8083/connectors
```

如果需要，我们可以验证此设置是否正常工作：

```shell
$CONFLUENT_HOME/bin/kafka-console-consumer --bootstrap-server localhost:9092 --topic connect-distributed --from-beginning
{"schema":{"type":"string","optional":false},"payload":"foo"}
{"schema":{"type":"string","optional":false},"payload":"bar"}
```

而且，如果我们查看文件夹$CONFLUENT_HOME，我们可以看到这里创建了一个文件test-distributed.sink.txt：

```shell
cat $CONFLUENT_HOME/test-distributed.sink.txt
foo
bar
```

测试分布式设置后，我们来清理一下，移除两个连接器：

```shell
curl -X DELETE http://localhost:8083/connectors/local-file-source
curl -X DELETE http://localhost:8083/connectors/local-file-sink
```

## 8. 转换数据

### 8.1 支持的转换

转换使我们能够对单个消息进行简单、轻量的修改。

Kafka Connect支持以下内置转换：

- InsertField：使用静态数据或记录元数据添加字段
- ReplaceField：过滤或重命名字段
- MaskField：将字段替换为该类型的有效空值(例如0或空字符串)
- HoistField：将整个事件包装为结构或Map中的单个字段
- ExtractField：从结构和Map中提取特定字段，并在结果中仅包含此字段
- SetSchemaMetadata：修改模式名称或版本
- TimestampRouter：根据原始主题和时间戳修改记录的主题
- RegexRouter：根据原始主题、替换字符串和正则表达式修改记录的主题

使用以下参数配置转换：

- transforms：转换的别名的逗号分隔列表
- transforms.\$alias.type：转换的类名
- transforms.\$alias.\$transformationSpecificConfig：相应转换的配置

### 8.2 应用转换器

为了测试一些转换特性，让我们设置以下两个转换：

- 首先，让我们将整个消息包装为JSON结构
- 之后，让我们向该结构添加一个字段

在应用转换之前，我们必须通过修改connect-distributed.properties将Connect配置为使用无模式JSON：

```properties
key.converter.schemas.enable=false
value.converter.schemas.enable=false
```

之后，我们必须重新启动Connect，再次以分布式模式：

```shell
$CONFLUENT_HOME/bin/connect-distributed $CONFLUENT_HOME/etc/kafka/connect-distributed.properties
```

再次，我们需要将源连接器POST的主体创建为JSON文件。这里，我们将其命名为connect-file-source-transform.json。

除了已知的参数之外，我们还为两个所需的转换添加几行：

```json
{
    "name": "local-file-source",
    "config": {
        "connector.class": "FileStreamSource",
        "tasks.max": 1,
        "file": "test-transformation.txt",
        "topic": "connect-transformation",
        "transforms": "MakeMap,InsertSource",
        "transforms.MakeMap.type": "org.apache.kafka.connect.transforms.HoistField$Value",
        "transforms.MakeMap.field": "line",
        "transforms.InsertSource.type": "org.apache.kafka.connect.transforms.InsertField$Value",
        "transforms.InsertSource.static.field": "data_source",
        "transforms.InsertSource.static.value": "test-file-source"
    }
}
```

之后，我们来执行POST：

```shell
curl -d @$CONFLUENT_HOME/connect-file-source-transform.json \
  -H "Content-Type: application/json" \
  -X POST http://localhost:8083/connectors
```

让我们在test-transformation.txt中写入几行：

```text
Foo
Bar
```

如果我们现在检查connect-transformation主题，我们应该得到以下几行：

```json
{"line":"Foo","data_source":"test-file-source"}
{"line":"Bar","data_source":"test-file-source"}
```

## 9. 使用现成的连接器

使用这些简单的连接器后，让我们看看更高级的即用型连接器，以及如何安装它们。

### 9.1 在哪里可以找到连接器

**预建连接器可从不同来源获得**：

- 一些连接器与普通的Apache Kafka(文件和控制台的源和接收器)捆绑在一起
- Confluent Platform还捆绑了一些连接器(ElasticSearch、HDFS、JDBC和AWS S3)
- 还可以查看Confluent Hub，它是Kafka连接器的一种托管中心，提供的连接器数量在不断增长：

  - Confluent连接器(由Confluent开发、测试、记录并完全支持)
  - 经过认证的连接器(由第三方实现并经Confluent认证)
  - 社区开发和支持的连接器

- 除此之外，Confluent还提供了一个[连接器页面](https://www.confluent.io/product/connectors/)，其中的一些连接器也可以在Confluent Hub上使用，还有一些社区连接器

- 最后，还有一些供应商将连接器作为其产品的一部分提供。例如，Landoop提供了一个名为[Lenses](https://www.landoop.com/downloads/)的流处理库，其中还包含一组约25个开源连接器

### 9.2 从Confluent Hub安装连接器

Confluent企业版提供了从Confluent Hub安装连接器和其他组件的脚本(该脚本不包含在开源版本中)，如果我们使用企业版，我们可以使用以下命令安装连接器：

```shell
$CONFLUENT_HOME/bin/confluent-hub install confluentinc/kafka-connect-mqtt:1.0.0-preview
```

### 9.3 手动安装连接器

如果我们需要一个连接器，而Confluent Hub上没有该连接器，或者我们只有Confluent的开源版本，我们可以手动安装所需的连接器。为此，我们必须下载并解压缩连接器，并将包含的库移动到plugin.path指定的文件夹中。

对于每个连接器，存档应包含两个我们感兴趣的文件夹：

- lib文件夹包含连接器jar，例如kafka-connect-mqtt-1.0.0-preview.jar，以及连接器所需的一些jar
- etc文件夹包含一个或多个参考配置文件

**我们必须将lib文件夹移动到$CONFLUENT_HOME/share/java，或者我们在connect-standalone.properties和connect-distributed.properties中指定为plugin.path的任何路径，在这样做时，将文件夹重命名为有意义的名称是推荐的**。

我们可以在独立模式启动时引用来自等的配置文件，或者直接获取属性并从中创建JSON文件。

## 10. 总结

在本教程中，我们了解了如何安装和使用Kafka Connect。

我们研究了连接器的类型，包括源连接器和接收器连接器，我们还研究了Connect可以运行的一些功能和模式。然后，我们回顾了转换器。最后，我们了解了从哪里获得以及如何安装自定义连接器。