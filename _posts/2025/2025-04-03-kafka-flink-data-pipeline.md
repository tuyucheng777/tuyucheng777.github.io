---
layout: post
title:  使用Flink和Kafka构建数据管道
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 概述

[Apache Flink](https://flink.apache.org/)是一个可以与Java轻松配合使用的流处理框架，[Apache Kafka](https://kafka.apache.org/)是一个支持高容错的分布式流处理系统。

在本教程中，我们将研究如何使用这两种技术构建[数据管道。](https://www.baeldung.com/cs/data-pipelines)

## 2. 安装

安装和配置Apache Kafka请参考[官方指南](https://kafka.apache.org/quickstart)。安装后，我们可以使用以下命令创建名为flink_input 和flink_output的新主题：

```shell
 bin/kafka-topics.sh --create \
  --zookeeper localhost:2181 \
  --replication-factor 1 --partitions 1 \
  --topic flink_output

 bin/kafka-topics.sh --create \
  --zookeeper localhost:2181 \
  --replication-factor 1 --partitions 1 \
  --topic flink_input
```

为了本教程的目的，我们将使用Apache Kafka的默认配置和默认端口。

## 3. Flink使用

Apache Flink允许实时流处理技术，**该框架允许使用多个第三方系统作为流源或接收器**。

在Flink中，有多种可用的连接器：

- Apache Kafka(源/接收器)
- Apache Cassandra(接收器)
- Amazon Kinesis Streams(源/接收器)
- Elasticsearch(接收器)
- Hadoop文件系统(接收器)
- RabbitMQ(源/接收器)
- Apache NiFi(源/接收器)
- Twitter Streaming API(源)

要将Flink添加到我们的项目中，我们需要包含以下Maven依赖：

```xml
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-core</artifactId>
    <version>1.16.1</version>
</dependency>
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-kafka</artifactId>
    <version>1.16.1</version>
</dependency>
```

添加这些依赖将使我们能够消费Kafka主题并生成数据，你可以在[Maven Central](https://mvnrepository.com/artifact/org.apache.flink)上找到Flink的最新版本。

## 4. Kafka字符串消费者

**要使用Flink从Kafka消费数据，我们需要提供一个主题和一个Kafka地址**。我们还应该提供一个组ID，用于保存偏移量，这样我们就不会总是从头开始读取整个数据。

让我们创建一个静态方法，使FlinkKafkaConsumer的创建更容易：

```java
public static FlinkKafkaConsumer011<String> createStringConsumerForTopic(String topic, String kafkaAddress, String kafkaGroup ) {
    Properties props = new Properties();
    props.setProperty("bootstrap.servers", kafkaAddress);
    props.setProperty("group.id",kafkaGroup);
    FlinkKafkaConsumer011<String> consumer = new FlinkKafkaConsumer011<>(topic, new SimpleStringSchema(), props);

    return consumer;
}
```

此方法接收topic、kafkaAddress和kafkaGroup，并创建FlinkKafkaConsumer，它将以字符串的形式消费来自给定主题的数据，因为我们已经使用SimpleStringSchema来解码数据。

类名称中的数字011指的是Kafka版本。

## 5. Kafka字符串生产者

**要向Kafka生成数据，我们需要提供要使用的Kafka地址和主题**。同样，我们可以创建一个静态方法来帮助我们为不同的主题创建生产者：

```java
public static FlinkKafkaProducer011<String> createStringProducer(String topic, String kafkaAddress){
    return new FlinkKafkaProducer011<>(kafkaAddress, topic, new SimpleStringSchema());
}
```

此方法仅接收topic和kafkaAddress作为参数，因为在生成Kafka主题时无需提供组ID。

## 6. 字符串流处理

当我们拥有一个功能齐全的消费者和生产者时，我们可以尝试处理来自Kafka的数据，然后将结果保存回Kafka。可用于流处理的完整函数列表可在[此处](https://www.baeldung.com/apache-flink)找到。

在这个例子中，我们将把每个Kafka条目中的单词大写，然后将其写回Kafka。

为此，我们需要创建一个自定义的MapFunction：

```java
public class WordsCapitalizer implements MapFunction<String, String> {
    @Override
    public String map(String s) {
        return s.toUpperCase();
    }
}
```

创建函数后，我们可以在流处理中使用它：

```java
public static void capitalize() {
    String inputTopic = "flink_input";
    String outputTopic = "flink_output";
    String consumerGroup = "tuyucheng";
    String address = "localhost:9092";
    StreamExecutionEnvironment environment = StreamExecutionEnvironment
            .getExecutionEnvironment();
    FlinkKafkaConsumer011<String> flinkKafkaConsumer = createStringConsumerForTopic(
            inputTopic, address, consumerGroup);
    DataStream<String> stringInputStream = environment
            .addSource(flinkKafkaConsumer);

    FlinkKafkaProducer011<String> flinkKafkaProducer = createStringProducer(
            outputTopic, address);

    stringInputStream
            .map(new WordsCapitalizer())
            .addSink(flinkKafkaProducer);
}
```

**应用程序将从flink_input主题读取数据，对流执行操作，然后将结果保存到Kafka中的flink_output主题**。

我们已经了解了如何使用Flink和Kafka处理字符串，但通常需要对自定义对象执行操作，我们将在下一章中了解如何执行此操作。

## 7. 自定义对象反序列化

以下类表示带有发送者和接收者信息的简单消息：

```java
@JsonSerialize
public class InputMessage {
    String sender;
    String recipient;
    LocalDateTime sentAt;
    String message;
}
```

之前，我们使用SimpleStringSchema来反序列化来自Kafka的消息，但现在我们想**将数据直接反序列化为自定义对象**。

为此，我们需要一个自定义的DeserializationSchema：

```java
public class InputMessageDeserializationSchema implements DeserializationSchema<InputMessage> {

    static ObjectMapper objectMapper = new ObjectMapper().registerModule(new JavaTimeModule());

    @Override
    public InputMessage deserialize(byte[] bytes) throws IOException {
        return objectMapper.readValue(bytes, InputMessage.class);
    }

    @Override
    public boolean isEndOfStream(InputMessage inputMessage) {
        return false;
    }

    @Override
    public TypeInformation&lt;InputMessage&gt; getProducedType() {
        return TypeInformation.of(InputMessage.class);
    }
}
```

我们在此假设消息在Kafka中以JSON格式保存。

由于我们有一个LocalDateTime类型的字段，我们需要指定JavaTimeModule，它负责将LocalDateTime对象映射到JSON。

**Flink模式不能包含不可序列化的字段**，因为所有操作符(如模式或函数)都在作业开始时被序列化。

Apache Spark中也存在类似的问题，此问题的已知修复方法之一是将字段初始化为static，就像我们上面对ObjectMapper所做的那样。这不是最完美的解决方案，但它相对简单且能完成工作。

方法isEndOfStream可用于特殊情况，即仅在收到某些特定数据时才处理流，但在我们的例子中不需要它。

## 8. 自定义对象序列化

现在，假设我们希望我们的系统能够创建消息备份。我们希望这个过程是自动的，并且每个备份应该由一整天内发送的消息组成。

此外，备份消息应该分配一个唯一的ID。

为了这个目的，我们可以创建以下类：

```java
public class Backup {
    @JsonProperty("inputMessages")
    List<InputMessage> inputMessages;
    @JsonProperty("backupTimestamp")
    LocalDateTime backupTimestamp;
    @JsonProperty("uuid")
    UUID uuid;

    public Backup(List<InputMessage> inputMessages, LocalDateTime backupTimestamp) {
        this.inputMessages = inputMessages;
        this.backupTimestamp = backupTimestamp;
        this.uuid = UUID.randomUUID();
    }
}
```

请注意，UUID生成机制并不完善，因为它允许重复。不过，这对于本示例的范围来说已经足够了。

我们希望将Backup对象作为JSON保存到Kafka，因此我们需要创建SerializationSchema：

```java
public class BackupSerializationSchema implements SerializationSchema<Backup> {

    ObjectMapper objectMapper;
    Logger logger = LoggerFactory.getLogger(BackupSerializationSchema.class);

    @Override
    public byte[] serialize(Backup backupMessage) {
        if(objectMapper == null) {
            objectMapper = new ObjectMapper()
                    .registerModule(new JavaTimeModule());
        }
        try {
            return objectMapper.writeValueAsString(backupMessage).getBytes();
        } catch (com.fasterxml.jackson.core.JsonProcessingException e) {
            logger.error("Failed to parse JSON", e);
        }
        return new byte[0];
    }
}
```

## 9. 时间戳消息

由于我们要为每天的所有消息创建备份，因此消息需要时间戳。

Flink提供了3种不同的时间特性EventTime、ProcessingTime和IngestionTime。

在我们的例子中，我们需要使用发送消息的时间，因此我们将使用EventTime。

**要使用EventTime，我们需要一个TimestampAssigner，它将从我们的输入数据中提取时间戳**：

```java
public class InputMessageTimestampAssigner implements AssignerWithPunctuatedWatermarks<InputMessage> {

    @Override
    public long extractTimestamp(InputMessage element, long previousElementTimestamp) {
        ZoneId zoneId = ZoneId.systemDefault();
        return element.getSentAt().atZone(zoneId).toEpochSecond() * 1000;
    }

    @Nullable
    @Override
    public Watermark checkAndGetNextWatermark(InputMessage lastElement, long extractedTimestamp) {
        return new Watermark(extractedTimestamp - 1500);
    }
}
```

我们需要将LocalDateTime转换为EpochSecond，因为这是Flink期望的格式。分配时间戳后，所有基于时间的操作都将使用sentAt字段中的时间进行操作。

由于Flink期望时间戳以毫秒为单位，而toEpochSecond()返回时间以秒为单位，因此我们需要将其乘以1000，这样Flink才能正确创建窗口。

Flink定义了Watermark的概念，当数据未按发送顺序到达时，Watermark非常有用，**Watermark定义了允许处理元素的最大延迟时间**。

时间戳低于Watermark的元素将根本不会被处理。

## 10. 创建时间窗口

为了确保我们的备份仅收集一天内发送的消息，我们可以在流上使用timeWindowAll方法，它将消息分成窗口。

但是，我们仍然需要汇总来自每个窗口的消息并将它们作为Backup返回。

为此，我们需要一个自定义的AggregateFunction：

```java
public class BackupAggregator implements AggregateFunction<InputMessage, List<InputMessage>, Backup> {

    @Override
    public List<InputMessage> createAccumulator() {
        return new ArrayList<>();
    }

    @Override
    public List<InputMessage> add(InputMessage inputMessage, List<InputMessage> inputMessages) {
        inputMessages.add(inputMessage);
        return inputMessages;
    }

    @Override
    public Backup getResult(List<InputMessage> inputMessages) {
        return new Backup(inputMessages, LocalDateTime.now());
    }

    @Override
    public List<InputMessage> merge(List<InputMessage> inputMessages, List<InputMessage> acc1) {
        inputMessages.addAll(acc1);
        return inputMessages;
    }
}
```

## 11. 聚合备份

分配适当的时间戳并实现我们的AggregateFunction后，我们最终可以获取Kafka输入并处理它：

```java
public static void createBackup () throws Exception {
    String inputTopic = "flink_input";
    String outputTopic = "flink_output";
    String consumerGroup = "tuyucheng";
    String kafkaAddress = "192.168.99.100:9092";
    StreamExecutionEnvironment environment
            = StreamExecutionEnvironment.getExecutionEnvironment();
    environment.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
    FlinkKafkaConsumer011<InputMessage> flinkKafkaConsumer
            = createInputMessageConsumer(inputTopic, kafkaAddress, consumerGroup);
    flinkKafkaConsumer.setStartFromEarliest();

    flinkKafkaConsumer.assignTimestampsAndWatermarks(
            new InputMessageTimestampAssigner());
    FlinkKafkaProducer011<Backup> flinkKafkaProducer
            = createBackupProducer(outputTopic, kafkaAddress);

    DataStream<InputMessage> inputMessagesStream
            = environment.addSource(flinkKafkaConsumer);

    inputMessagesStream
            .timeWindowAll(Time.hours(24))
            .aggregate(new BackupAggregator())
            .addSink(flinkKafkaProducer);

    environment.execute();
}
```

## 12. 总结

在本文中，我们介绍了如何使用Apache Flink和Apache Kafka创建一个简单的数据管道。