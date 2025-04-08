---
layout: post
title:  使用Reactive Kafka Stream和Spring WebFlux
category: springreactive
copyright: springreactive
excerpt: Spring Reactive
---

## 1. 概述

在本文中，我们将探索Reactive [Kafka Streams](https://kafka.apache.org/documentation/streams/)，将它们集成到示例Spring WebFlux应用程序中，并研究这种组合如何使我们能够构建具有可扩展性、效率和实时处理的完全响应式、数据密集型应用程序。

为了实现这一点，我们将使用[Spring Cloud Stream Reactive Kafka Binder](https://docs.spring.io/spring-cloud-stream/docs/current/reference/html/spring-cloud-stream-binder-kafka.html#_reactive_kafka_binder)、[Spring WebFlux](https://docs.spring.io/spring-framework/reference/web/webflux.html)和[ClickHouse](https://clickhouse.com/)。

## 2. Spring Cloud Stream Reactive Kafka Binder

Spring Cloud Stream在基于流和消息驱动的微服务上提供了一个抽象层，**Reactive Kafka Binder通过连接Kafka主题、消息代理或Spring Cloud Stream应用程序，支持创建完全响应式的管道**。这些管道利用[Project Reactor](https://www.baeldung.com/reactor-core)响应式地处理数据流，确保整个数据流中的处理无阻塞、异步和背压感知。

**与同步运行的传统[Kafka Streams](https://www.baeldung.com/java-kafka-streams)不同，Reactive Kafka Streams使开发人员能够定义端到端的响应式管道，其中每块数据都可以实时映射、转换、过滤或缩减，同时仍保持高效的资源利用率**。

这种方法特别适合高吞吐量、事件驱动的应用程序，这些应用程序需要响应式范式来实现更好的可扩展性和响应能力。

### 2.1 使用Spring实现Reactive Kafka Streams

借助Spring Cloud Stream Reactive Kafka Binder，我们可以将Reactive Kafka Streams无缝集成到Spring WebFlux应用程序中，从而实现完全响应式、非阻塞的数据处理。通过利用Project Reactor提供的响应式API，我们可以处理背压、实现异步数据流并高效处理流而不会阻塞线程。

**Reactive Kafka Streams和Spring WebFlux的组合为构建需要分布式、实时和响应式数据管道的应用程序提供了强大的解决方案**。

接下来，让我们深入研究一个示例应用程序来演示这些功能的实际效果。

## 3. 构建Reactive Kafka Stream应用程序

**在此示例应用程序中，我们将模拟一个股票分析应用程序，用于接收、处理和分发股票价格数据**。此应用程序将展示Spring Cloud Stream、Kafka和响应式编程范例在Spring生态系统中如何协同工作。

首先，让我们添加使用[Spring Boot](https://spring.io/projects/spring-boot)构建此类应用程序所需的所有依赖：

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>2023.0.2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

对于此示例，我们将使用[Spring Cloud BOM](https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-dependencies)，它解决了所有依赖的版本问题，我们还将使用[Spring Boot](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-dependencies)和以下模块：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-kafka</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-stream-binder-kafka-reactive</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

这些模块允许我们以响应式方式构建Web层和数据提取管道，尽管我们有数据处理管道，但我们仍需要一些数据持久化来保存其结果。让我们使用一个简单且功能非常强大的分析数据库来实现这一点：

```xml
<dependency>
    <groupId>com.clickhouse</groupId>
    <artifactId>clickhouse-r2dbc</artifactId>
    <version>0.7.1</version>
</dependency>
```

[ClickHouse](https://clickhouse.com/)是一款快速、开源、面向列的数据库管理系统，可使用SQL查询生成实时分析数据报告。由于我们的目标是构建一个完全响应式的应用程序，因此我们将使用它的[R2DB驱动程序](https://mvnrepository.com/artifact/com.clickhouse/clickhouse-r2dbc)。

### 3.1 响应式Kafka生产者设置

为了启动我们的数据处理管道，我们需要一个生产者，负责创建数据并将其提交给我们的应用程序进行数据提取。接下来，我们将看到Spring如何帮助我们轻松定义和使用生产者：

```java
@Component
public class StockPriceProducer {
    public static final String[] STOCKS = {"AAPL", "GOOG", "MSFT", "AMZN", "TSLA"};
    private static final String CURRENCY = "USD";

    private final ReactiveKafkaProducerTemplate<String, StockUpdate> kafkaProducer;
    private final NewTopic topic;
    private final Random random = new Random();

    public StockPriceProducer(KafkaProperties properties,
                              @Qualifier(TopicConfig.STOCK_PRICES_IN) NewTopic topic) {
        this.kafkaProducer = new ReactiveKafkaProducerTemplate<>(
                SenderOptions.create(properties.buildProducerProperties())
        );
        this.topic = topic;
    }

    public Flux<SenderResult<Void>> produceStockPrices(int count) {
        return Flux.range(0, count)
                .map(i -> {
                    String stock = STOCKS[random.nextInt(STOCKS.length)];
                    double price = 100 + (200 * random.nextDouble());
                    return MessageBuilder.withPayload(new StockUpdate(stock, price, CURRENCY, Instant.now()))
                            .setHeader(MessageHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
                            .build();
                })
                .flatMap(stock -> {
                    var newRecord = new ProducerRecord<>(
                            topic.name(),
                            stock.getPayload().symbol(),
                            stock.getPayload());

                    stock.getHeaders()
                            .forEach((key, value) -> newRecord.headers().add(key, value.toString().getBytes()));

                    return kafkaProducer.send(newRecord);
                });
    }
}
```

**此类生成股票价格更新并将其发送到我们的Kafka主题**。

在StockPriceProducer中，我们注入应用程序YAML文件中定义的KafkaProperties，其中包含连接到Kafka集群所需的所有信息：

```yaml
spring:
    kafka:
        producer:
            bootstrap-servers: localhost:9092
            key-serializer: org.apache.kafka.common.serialization.StringSerializer
            value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
        properties:
            spring:
                json:
                    trusted:
                        packages: '*'
```

然后，NewTopic保存了对Kafka主题的引用，这就是我们创建ReactiveKafkaProducerTemplate实例所需的全部内容，此类抽象了我们的应用程序与Kafka主题之间通信所涉及的大部分复杂性。

在produceStockPrices()方法中，我们生成StockUpdate对象并将其包装在Message对象中。Spring提供了Message类，该类封装了基于消息的系统详细信息，例如消息负载以及我们可能需要作为消息内容类型包含的任何必要标头。最后，我们创建一个ProducerRecord来定义消息的目标主题及其分区键，然后发送它。

### 3.2 响应式Kafka Streams设置

现在，让我们假设生产者位于同一应用程序之外。我们需要连接到股票价格更新主题并将股票价格从美元转换为欧元，以便其他应用程序部分可以使用该数据。同时，我们需要在特定时间窗口内保存原始股票价格的历史记录。因此，让我们配置我们的数据流管道：

```yaml
spring:
   cloud:
      stream:
         default-binder: kafka
         kafka:
            binder:
               brokers: localhost:9092
         bindings:
            default:
               content-type: application/json
            processStockPrices-in-0:
               destination: stock-prices-in
               group: live-stock-consumers-x
            processStockPrices-out-0:
               destination: stock-prices-out
               group: live-stock-consumers-y
               producer:
                  useNativeEncoding: true
```

**首先，我们使用default-binder属性将Kafka定义为我们的默认绑定器。[Spring Cloud Stream](https://www.baeldung.com/spring-cloud-stream)与供应商无关，允许我们在必要时在同一应用程序中使用不同的消息系统(例如Kafka和RabbitMQ)**。

接下来，我们配置[bindings](https://docs.spring.io/spring-cloud-stream/reference/spring-cloud-stream/bindings.html)，它充当消息系统(例如，Kafka主题)与应用程序的生产者和消费者之间的桥梁：

- 输入通道processStockPrices-in-0绑定到stock-prices-in主题，消息在此被消费。
- 输出通道processStockPrices-out-0绑定到stock-prices-out主题，处理后的消息在此发布。

每个绑定都与processStockPrices()方法相关联，该方法处理来自输入通道的数据、应用转换并将结果发送到输出通道。

我们还将内容类型定义为JSON，确保消息以JSON格式序列化和反序列化。此外，在生产者中使用useNativeEncoding:true可确保Kafka生产者负责对数据进行编码和序列化。

group属性(例如live-stock-consumers-x)可实现消费者之间的消息负载均衡，同一组中的所有消费者负责处理来自某个主题的消息，从而防止重复。

### 3.3 响应式Kafka Streams绑定设置

如前所述，绑定是输入和输出通道之间的桥梁，使我们能够处理传输中的数据。**YAML文件中定义的名称至关重要，因为它必须与绑定实现相对应，在我们的例子中，绑定实现是应用输入和输出消息之间的映射的函数**。

接下来我们看看Spring是如何做的：

```java
@Configuration
public class StockPriceProcessor {
    private static final String USD = "USD";
    private static final String EUR = "EUR";

    @Bean
    public Function<Flux<Message<StockUpdate>>, Flux<Message<StockUpdate>>> processStockPrices(
            ClickHouseRepository repository,
            CurrencyRate currencyRate
    ) {
        return stockPrices -> stockPrices.flatMapSequential(message -> {
            StockUpdate stockUpdate = message.getPayload();
            return repository.saveStockPrice(stockUpdate)
                    .flatMap(success -> Boolean.TRUE.equals(success) ? Mono.just(stockUpdate) : Mono.empty())
                    .flatMap(stock -> currencyRate.convertRate(USD, EUR, stock.price()))
                    .map(newPrice -> convertPrice(stockUpdate, newPrice))
                    .map(priceInEuro -> MessageBuilder.withPayload(priceInEuro)
                            .setHeader(KafkaHeaders.KEY, stockUpdate.symbol())
                            .copyHeaders(message.getHeaders())
                            .build());
        });
    }

    private StockUpdate convertPrice(StockUpdate stockUpdate, double newPrice) {
        return new StockUpdate(stockUpdate.symbol(), newPrice, EUR, stockUpdate.timestamp());
    }
}
```

**此配置演示了如何在两个Kafka主题之间响应式地处理和转换股票价格更新**，processStockPrices()函数将输入的stock-prices-in主题与输出的stock-prices-out主题绑定，在它们之间添加一个处理层，流程如下：

1. **消息处理**：使用flatMapSequential()按顺序处理来自输入主题的每条传入的StockUpdate消息，这可确保处理顺序与输入消息的顺序相匹配，这对于保持一致性非常重要。
2. **数据库持久化**：每个库存更新都使用ClickHouseRepository保存到数据库中以供将来参考，只有成功保存的更新才会继续执行。
3. **货币转换**：股票价格原本以美元计价，使用CurrencyRate服务转换为欧元。
4. **消息转换**：转换后的价格被包装在一个新的StockUpdate对象中，并通过KafkaHeaders.KEY将原始符号保留为Kafka消息键，这可确保Kafka主题中的消息分区正确。
5. **响应式管道**：整个流程是响应式的，利用Project Reactor的非阻塞异步功能实现可扩展性和效率。

### 3.4 辅助服务

ClickHouseRepository和CurrencyRate是简单的接口，为我们提供了简单的实现来说明示例应用程序：

```java
public interface CurrencyRate {
    Mono<Double> convertRate(String from, String to, double amount);
}

public interface ClickHouseRepository {
    Mono<Boolean> saveStockPrice(StockUpdate stockUpdate);
    Flux<StockUpdate> findMinuteAvgStockPrices(Instant from, Instant to);
}
```

这些功能向我们展示了应用程序在处理此类数据管道时可以应用的业务逻辑。

### 3.5 响应式Kafka Streams消费者设置

处理完毕后，发送到输出通道的数据可由同一应用程序或任何其他应用程序消费，这样的消费者也可以使用响应式Kafka模板来实现：

```java
@Component
public class StockPriceConsumer {
    private final ReactiveKafkaConsumerTemplate<String, StockUpdate> kafkaConsumerTemplate;

    public StockPriceConsumer(@NonNull KafkaProperties properties,
                              @Qualifier(TopicConfig.STOCK_PRICES_OUT) NewTopic topic) {
        var receiverOptions = ReceiverOptions
                .<String, StockUpdate>create(properties.buildConsumerProperties())
                .subscription(List.of(topic.name()));
        this.kafkaConsumerTemplate = new ReactiveKafkaConsumerTemplate<>(receiverOptions);
    }

    @PostConstruct
    public void consume() {
        kafkaConsumerTemplate
                .receiveAutoAck()
                .doOnNext(consumerRecord -> {
                    // simulate processing
                    log.info(
                            "received key={}, value={} from topic={}, offset={}, partition={}", consumerRecord.key(),
                            consumerRecord.value(),
                            consumerRecord.topic(),
                            consumerRecord.offset(),
                            consumerRecord.partition());
                })
                .doOnError(e -> log.error("Consumer error",  e))
                .doOnComplete(() -> log.info("Consumed all messages"))
                .subscribe();
    }
}
```

StockPriceConsumer演示了如何以响应式方式消费来自stock-prices-out主题的数据：

1. 初始化：构造函数使用YAML配置中的Kafka属性创建ReceiverOptions，它订阅stock-prices-out主题并明确分配所有分区。
2. 消息处理：consume方法使用acceptAutoAck()订阅输出通道(processStockPrices-out-0)，每条消息都记录了键、值、主题、偏移量和分区详细信息，模拟数据处理。
3. 响应式功能：消费者在消息到达时开始响应式地处理消息，利用非阻塞、背压感知处理。它还会记录错误doOnError()并跟踪完成doOnComplete()。

以下属性配置我们的消费者：

```yaml
spring:
   kafka:
      consumer:
         bootstrap-servers: localhost:9092
         key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
         value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
         group-id: my-group
         properties:
            reactiveAutoCommit: true
```

该消费者以响应式方式处理stock-prices-out主题，此实现强调了响应式编程与Kafka的无缝集成，以实现高效的流处理。

### 3.6 响应式WebFlux应用程序

最后，既然数据已经保存在我们的数据库中，我们就可以充分地向我们的用户提供这些信息，并且可以根据需要进行处理：

```java
@RestController
public class StocksApi {
    private final ClickHouseRepository repository;

    @Autowired
    public StocksApi(ClickHouseRepository repository) {
        this.repository = repository;
    }

    @GetMapping("/stock-prices-out")
    public Flux<StockUpdate> getAvgStockPrices(@RequestParam("from") @NotNull Instant from,
                                               @RequestParam("to") @NotNull Instant to) {
        if (from.isAfter(to)) {
            throw new ResponseStatusException(HttpStatus.BAD_REQUEST, "'from' must come before 'to'");
        }

        return repository.findMinuteAvgStockPrices(from, to);
    }
}
```

## 4. 连接点

我们用最少的代码实现了完全响应式的数据处理管道，连接两个Kafka主题，应用业务逻辑，并确保高吞吐量处理，这种方法非常适合需要实时数据转换的事件驱动系统。**Spring Cloud Stream和Kafka形成了一个强大的组合，其功能超出了我们在此处介绍的范围**。

**例如，绑定支持多个输入和输出，而死信队列([DLQ](https://www.baeldung.com/kafka-spring-dead-letter-queue))可以增强管道稳健性**。还可以集成各种消息传递提供程序，实现通道之间的事务处理等。

Spring Cloud Stream是一款多功能工具，将其与响应式范式相结合，可以解锁具有弹性和高吞吐量的强大数据管道。本文仅涉及使用响应式Kafka Streams和Spring WebFlux的表面，还有更多内容有待探索，但到目前为止，我们已经观察到了关键优势：

- **实时转换**：实现事件流的实时转换和丰富。
- **背压管理**：动态处理数据流，避免系统过载。
- **无缝集成**：将Kafka的事件驱动功能与Spring WebFlux的非阻塞功能相结合。
- **可扩展设计**：支持具有DLQ等强大容错机制的高吞吐量系统。

尽管这种方法有很多好处，但正如本文所讨论的，也有一些需要注意的点。

### 4.1 实际陷阱和最佳实践

虽然响应式Kafka管道提供了许多优点，但它们也带来了挑战：

- **背压处理**：无法管理背压将导致内存膨胀或丢失消息，我们需要在适当的情况下使用.onBackpressureBuffer()或.onBackpressureDrop()。
- **序列化问题**：生产者和消费者之间的模式不匹配可能导致反序列化失败，我们必须确保模式兼容性。
- **错误恢复**：我们必须确保适当的重试机制或使用DLQ来有效地处理瞬态问题。
- **资源管理**：低效的消息处理可能会使应用程序管道不堪重负，在这种情况下，我们可以利用.limitRate()或.take()运算符来控制我们的响应式管道内的处理速率。我们还可以配置Kafka消费者提取大小和轮询间隔，以调整从Kafka检索消息的速率并避免使应用程序管道不堪重负。
- **数据一致性**：如果没有原子操作或适当的重试处理，可能会出现数据处理不一致的情况。我们可以使用Kafka事务来实现原子性或/和编写幂等消费者逻辑来安全地处理重试。
- **模式演进**：在没有正确版本控制的情况下演进模式可能会导致兼容性问题，我们可以使用模式注册表进行版本控制并应用向后兼容的更改(例如，添加可选字段)。
- **监控和可观测性**：监控不足会使识别管道中的瓶颈或故障变得困难，我们必须集成[Micrometer](https://micrometer.io/)和[Grafana](https://grafana.com/)(或任何其他首选提供商)等工具来进行指标和监控。我们还可以将跟踪ID添加到Kafka消息中以进行分布式跟踪。

注意这些点可以保证我们的系统拥有非常稳定和可扩展的数据处理管道。

## 5. 总结

**在本文中，我们演示了如何将Reactive Kafka Streams与Spring WebFlux集成，从而实现完全响应式的数据密集型管道，这些管道可扩展、高效且能够实时处理**。通过利用响应式范式，我们在Kafka主题之间建立了无缝数据流，应用了业务逻辑，并以最少的代码实现了高吞吐量、事件驱动的处理。这种强大的组合凸显了现代响应式技术在创建适合实时数据转换的强大且可扩展的系统方面的潜力。