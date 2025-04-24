---
layout: post
title:  Spring Cloud AWS v3中的消息转换
category: springcloud
copyright: springcloud
excerpt: Spring Cloud AWS
---

## 1. 概述

消息转换是应用程序在传输和接收消息时在不同格式和表示形式之间进行转换的过程。

[AWS SQS](https://aws.amazon.com/sqs/features/)允许文本有效负载，并且Spring Cloud AWS SQS集成提供了熟悉的Spring抽象，以默认使用JSON管理文本有效负载与POJO和记录的序列化和反序列化。

**在本教程中，我们将使用事件驱动的场景来介绍消息转换的三个常见用例：POJO/记录序列化和反序列化，设置自定义ObjectMapper以及反序列化为子类/接口实现**。

为了测试我们的用例，我们将利用[Spring Cloud AWS SQS V3介绍文章](https://www.baeldung.com/java-spring-cloud-aws-v3-intro)中的环境和测试设置。

## 2. 依赖

让我们首先导入[Spring Cloud AWS BOM](https://mvnrepository.com/artifact/io.awspring.cloud/spring-cloud-aws)，它管理我们的依赖版本，确保它们之间的版本兼容性：
```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>io.awspring.cloud</groupId>
            <artifactId>spring-cloud-aws</artifactId>
            <version>${spring-cloud-aws.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

现在，我们可以添加核心和SQS Starter依赖：
```xml
<dependency>
    <groupId>io.awspring.cloud</groupId>
    <artifactId>spring-cloud-aws-starter</artifactId>
</dependency>
<dependency>
    <groupId>io.awspring.cloud</groupId>
    <artifactId>spring-cloud-aws-starter-sqs</artifactId>
</dependency>
```

在本教程中，我们将使用Spring Boot Web Starter；**值得注意的是，我们没有指定版本，因为我们导入了Spring Cloud AWS的BOM**：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

最后，除了Spring Boot的测试依赖之外，让我们添加测试依赖-使用[JUnit 5](https://www.baeldung.com/junit-5)的[LocalStack和TestContainers](https://mvnrepository.com/artifact/org.testcontainers/localstack/)、用于验证异步消息使用的[awaitility库](https://www.baeldung.com/awaitility-testing)以及使用流式API处理断言的[AssertJ](https://www.baeldung.com/introduction-to-assertj)：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>localstack</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>junit-jupiter</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.awaitility</groupId>
    <artifactId>awaitility</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.assertj</groupId>
    <artifactId>assertj-core</artifactId>
    <scope>test</scope>
</dependency>
```

## 3. 设置本地测试环境

现在我们已经添加了依赖，我们将通过创建BaseSqsLiveTest来设置我们的测试环境，我们的测试套件应该对其进行扩展：
```java
@Testcontainers
public class BaseSqsLiveTest {

    private static final String LOCAL_STACK_VERSION = "localstack/localstack:3.4.0";

    @Container
    static LocalStackContainer localStack = new LocalStackContainer(DockerImageName.parse(LOCAL_STACK_VERSION));

    @DynamicPropertySource
    static void overrideProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.cloud.aws.region.static", () -> localStack.getRegion());
        registry.add("spring.cloud.aws.credentials.access-key", () -> localStack.getAccessKey());
        registry.add("spring.cloud.aws.credentials.secret-key", () -> localStack.getSecretKey());
        registry.add("spring.cloud.aws.sqs.endpoint", () -> localStack.getEndpointOverride(SQS)
                .toString());
    }
}
```

## 4. 设置队列名称

为了利用Spring Boot的[配置外部化](https://www.baeldung.com/configuration-properties-in-spring-boot)，我们将在application.yml文件中添加队列名称：
```yaml
events:
    queues:
        shipping:
            simple-pojo-conversion-queue: shipping_pojo_conversion_queue
            custom-object-mapper-queue: shipping_custom_object_mapper_queue
            deserializes-subclass: deserializes_subclass_queue
```

我们现在创建一个带有@ConfigurationProperties注解的类，将其注入到我们的测试中以检索队列名称：
```java
@ConfigurationProperties(prefix = "events.queues.shipping")
public class ShipmentEventsQueuesProperties {

    private String simplePojoConversionQueue;

    private String customObjectMapperQueue;

    private String subclassDeserializationQueue;

    // ...getters and setters
}
```

最后，我们将@EnableConfigurationProperties注解添加到@Configuration类：
```java
@EnableConfigurationProperties({ ShipmentEventsQueuesProperties.class })
@Configuration
public class ShipmentServiceConfiguration {
}
```

## 5. 设置应用程序

我们将创建一个对ShipmentRequestedEvent做出反应的Shipment微服务来说明我们的用例。

首先，让我们创建用于保存货物信息的Shipment实体：
```java
public class Shipment {

    private UUID orderId;
    private String customerAddress;
    private LocalDate shipBy;
    private ShipmentStatus status;

    public Shipment(){}

    public Shipment(UUID orderId, String customerAddress, LocalDate shipBy, ShipmentStatus status) {
        this.orderId = orderId;
        this.customerAddress = customerAddress;
        this.shipBy = shipBy;
        this.status = status;
    }
    
    // ...getters and setters
}
```

接下来，让我们添加一个ShipmentStatus枚举：
```java
public enum ShipmentStatus {
    REQUESTED,
    PROCESSED,
    CUSTOMS_CHECK,
    READY_FOR_DISPATCH,
    SENT,
    DELIVERED
}
```

还需要ShipmentRequestedEvent：
```java
public class ShipmentRequestedEvent {

    private UUID orderId;
    private String customerAddress;
    private LocalDate shipBy;

    public ShipmentRequestedEvent() {
    }

    public ShipmentRequestedEvent(UUID orderId, String customerAddress, LocalDate shipBy) {
        this.orderId = orderId;
        this.customerAddress = customerAddress;
        this.shipBy = shipBy;
    }

    public Shipment toDomain() {
        return new Shipment(orderId, customerAddress, shipBy, ShipmentStatus.REQUESTED);
    }

    // ...getters and setters
}
```

为了处理我们的Shipment，我们将创建一个简单的ShipmentService类，其中包含一个模拟存储库，我们将使用它来断言我们的测试：
```java
@Service
public class ShipmentService {

    private static final Logger logger = LoggerFactory.getLogger(ShipmentService.class);

    private final Map<UUID, Shipment> shippingRepository = new ConcurrentHashMap<>();

    public void processShippingRequest(Shipment shipment) {
        logger.info("Processing shipping for order: {}", shipment.getOrderId());
        shipment.setStatus(ShipmentStatus.PROCESSED);
        shippingRepository.put(shipment.getOrderId(), shipment);
        logger.info("Shipping request processed: {}", shipment.getOrderId());
    }

    public Shipment getShipment(UUID requestId) {
        return shippingRepository.get(requestId);
    }
}
```

## 6. 使用默认配置处理POJO和记录

**Spring Cloud AWS SQS预先配置了一个SqsMessagingMessageConverter，它在使用SqsTemplate、@SqsListener注解或手动实例化的SqsMessageListenerContainer发送和接收消息时，将POJO和记录序列化为JSON并反序列化**。

我们的第一个用例是发送和接收一个简单的POJO来说明此默认配置，我们将使用@SqsListener注解来接收消息，并使用Spring Boot的自动配置在必要时配置反序列化。

首先，我们将创建发送消息的测试：
```java
@SpringBootTest
public class ShipmentServiceApplicationLiveTest extends BaseSqsLiveTest {

    @Autowired
    private SqsTemplate sqsTemplate;

    @Autowired
    private ShipmentService shipmentService;

    @Autowired
    private ShipmentEventsQueuesProperties queuesProperties;

    @Test
    void givenPojoPayload_whenMessageReceived_thenDeserializesCorrectly() {
        UUID orderId = UUID.randomUUID();
        ShipmentRequestedEvent shipmentRequestedEvent = new ShipmentRequestedEvent(orderId, "123 Main St", LocalDate.parse("2024-05-12"));

        sqsTemplate.send(queuesProperties.getSimplePojoConversionQueue(), shipmentRequestedEvent);

        await().atMost(Duration.ofSeconds(10))
                .untilAsserted(() -> {
                    Shipment shipment = shipmentService.getShipment(orderId);
                    assertThat(shipment).isNotNull();
                    assertThat(shipment).usingRecursiveComparison()
                            .ignoringFields("status")
                            .isEqualTo(shipmentRequestedEvent);
                    assertThat(shipment
                            .getStatus()).isEqualTo(ShipmentStatus.PROCESSED);
                });
    }
}
```

在这里，我们创建事件，使用自动配置的SqsTemplate将其发送到队列，并等待状态变为PROCESSED，这表明消息已成功接收和处理。

当测试被触发时，它会在10秒后失败，因为我们还没有队列的监听器。

让我们通过创建第一个@SqsListener来解决这个问题：
```java
@Component
public class ShipmentRequestListener {

    private final ShipmentService shippingService;

    public ShipmentRequestListener(ShipmentService shippingService) {
        this.shippingService = shippingService;
    }

    @SqsListener("${events.queues.shipping.simple-pojo-conversion-queue}")
    public void receiveShipmentRequest(ShipmentRequestedEvent shipmentRequestedEvent) {
        shippingService.processShippingRequest(shipmentRequestedEvent.toDomain());
    }
}
```

当我们再次运行测试时，它会在片刻之后通过。

**值得注意的是，监听器有@Component注解，并且我们引用了在application.yml文件中设置的队列名称**。

此示例展示了Spring Cloud AWS如何开箱即用地处理POJO转换，其工作方式与Java记录相同。

## 7. 配置自定义对象映射器

消息转换的一个常见用例是使用特定于应用程序的配置设置自定义ObjectMapper。

**对于我们的下一个场景，我们将配置一个带有LocalDateDeserializer的ObjectMapper来读取“dd-MM-yyyy”格式的日期**。

再次，我们首先创建测试场景。在本例中，我们将直接通过框架自动配置的SqsAsyncClient发送原始JSON负载：
```java
@Autowired
private SqsAsyncClient sqsAsyncClient;

@Test
void givenShipmentRequestWithCustomDateFormat_whenMessageReceived_thenDeserializesDateCorrectly() {
    UUID orderId = UUID.randomUUID();
    String shipBy = LocalDate.parse("2024-05-12")
            .format(DateTimeFormatter.ofPattern("dd-MM-yyyy"));

    var jsonMessage = """
            {
                "orderId": "%s",
                "customerAddress": "123 Main St",
                "shipBy": "%s"
            }
            """.formatted(orderId, shipBy);

    sendRawMessage(queuesProperties.getCustomObjectMapperQueue(), jsonMessage);

    await().atMost(Duration.ofSeconds(10))
            .untilAsserted(() -> {
                var shipment = shipmentService.getShipment(orderId);
                assertThat(shipment).isNotNull();
                assertThat(shipment.getShipBy()).isEqualTo(LocalDate.parse(shipBy, DateTimeFormatter.ofPattern("dd-MM-yyyy")));
            });
}

private void sendRawMessage(String queueName, String jsonMessage) {
    sqsAsyncClient.getQueueUrl(req -> req.queueName(queueName))
            .thenCompose(resp -> sqsAsyncClient.sendMessage(req -> req.messageBody(jsonMessage)
                    .queueUrl(resp.queueUrl())))
            .join();
}
```

我们还为这个队列添加监听器：
```java
@SqsListener("${events.queues.shipping.custom-object-mapper-queue}")
public void receiveShipmentRequestWithCustomObjectMapper(ShipmentRequestedEvent shipmentRequestedEvent) {
    shippingService.processShippingRequest(shipmentRequestedEvent.toDomain());
}
```

当我们现在运行测试时，它失败了，并且我们在堆栈跟踪中看到类似这样的消息：
```text
Cannot deserialize value of type `java.time.LocalDate` from String "12-05-2024"
```

这是因为我们没有使用标准的“yyyy-MM-dd”日期格式。

为了解决这个问题，我们需要配置一个能够解析此日期格式的ObjectMapper，**我们可以简单地将其声明为@Configuration注解类中的Bean，然后自动配置会将其正确设置为自动配置的SqsTemplate和@SqsListener方法**：
```java
@Bean
public ObjectMapper objectMapper() {
    ObjectMapper mapper = new ObjectMapper();
    JavaTimeModule module = new JavaTimeModule();
    LocalDateDeserializer customDeserializer = new LocalDateDeserializer(DateTimeFormatter.ofPattern("dd-MM-yyyy", Locale.getDefault()));
    module.addDeserializer(LocalDate.class, customDeserializer);
    mapper.registerModule(module);
    return mapper;
}
```

当我们再次运行测试时，它按预期通过。

## 8. 配置继承和接口反序列化

另一种常见的情况是有一个超类或接口，它有各种子类或实现，并且**需要根据标准(例如MessageHeader或消息的一部分)告知框架应该将消息反序列化到哪个特定类**。

为了说明这个用例，让我们为场景添加一些复杂性，并包括两种类型的Shipment：InternationalShipment和DomesticShipment，每个都是具有特定属性的Shipment的子类。

### 8.1 创建实体和事件
```java
public class InternationalShipment extends Shipment {

    private String destinationCountry;
    private String customsInfo;

    public InternationalShipment(UUID orderId, String customerAddress, LocalDate shipBy, ShipmentStatus status,
                                 String destinationCountry, String customsInfo) {
        super(orderId, customerAddress, shipBy, status);
        this.destinationCountry = destinationCountry;
        this.customsInfo = customsInfo;
    }

    // ...getters and setters
}

public class DomesticShipment extends Shipment {

    private String deliveryRouteCode;

    public DomesticShipment(UUID orderId, String customerAddress, LocalDate shipBy, ShipmentStatus status,
                            String deliveryRouteCode) {
        super(orderId, customerAddress, shipBy, status);
        this.deliveryRouteCode = deliveryRouteCode;
    }

    public String getDeliveryRouteCode() {
        return deliveryRouteCode;
    }

    public void setDeliveryRouteCode(String deliveryRouteCode) {
        this.deliveryRouteCode = deliveryRouteCode;
    }
}
```

让我们添加它们各自的事件：
```java
public class DomesticShipmentRequestedEvent extends ShipmentRequestedEvent {

    private String deliveryRouteCode;

    public DomesticShipmentRequestedEvent(){}

    public DomesticShipmentRequestedEvent(UUID orderId, String customerAddress, LocalDate shipBy, String deliveryRouteCode) {
        super(orderId, customerAddress, shipBy);
        this.deliveryRouteCode = deliveryRouteCode;
    }

    public DomesticShipment toDomain() {
        return new DomesticShipment(getOrderId(), getCustomerAddress(), getShipBy(), ShipmentStatus.REQUESTED, deliveryRouteCode);
    }

    // ...getters and setters
}

public class InternationalShipmentRequestedEvent extends ShipmentRequestedEvent {

    private String destinationCountry;
    private String customsInfo;

    public InternationalShipmentRequestedEvent(){}

    public InternationalShipmentRequestedEvent(UUID orderId, String customerAddress, LocalDate shipBy, String destinationCountry,
                                               String customsInfo) {
        super(orderId, customerAddress, shipBy);
        this.destinationCountry = destinationCountry;
        this.customsInfo = customsInfo;
    }

    public InternationalShipment toDomain() {
        return new InternationalShipment(getOrderId(), getCustomerAddress(), getShipBy(), ShipmentStatus.REQUESTED, destinationCountry,
                customsInfo);
    }

    // ...getters and setters
}
```

### 8.2 添加服务和监听器逻辑

我们将向Service添加两种方法，每种方法用于处理不同类型的Shipment：
```java
@Service
public class ShipmentService {

    // ...previous code stays the same

    public void processDomesticShipping(DomesticShipment shipment) {
        logger.info("Processing domestic shipping for order: {}", shipment.getOrderId());
        shipment.setStatus(ShipmentStatus.READY_FOR_DISPATCH);
        shippingRepository.put(shipment.getOrderId(), shipment);
        logger.info("Domestic shipping processed: {}", shipment.getOrderId());
    }

    public void processInternationalShipping(InternationalShipment shipment) {
        logger.info("Processing international shipping for order: {}", shipment.getOrderId());
        shipment.setStatus(ShipmentStatus.CUSTOMS_CHECK);
        shippingRepository.put(shipment.getOrderId(), shipment);
        logger.info("International shipping processed: {}", shipment.getOrderId());
    }
}
```

现在让我们添加处理消息的监听器，值得注意的是，我们在监听器方法中使用超类类型，因为此方法接收来自两个子类型的消息：
```java
@SqsListener(queueNames = "${events.queues.shipping.subclass-deserialization-queue}")
public void receiveShippingRequestWithType(ShipmentRequestedEvent shipmentRequestedEvent) {
    if (shipmentRequestedEvent instanceof InternationalShipmentRequestedEvent event) {
        shippingService.processInternationalShipping(event.toDomain());
    } else if (shipmentRequestedEvent instanceof DomesticShipmentRequestedEvent event) {
        shippingService.processDomesticShipping(event.toDomain());
    } else {
        throw new RuntimeException("Event type not supported " + shipmentRequestedEvent.getClass().getSimpleName());
    }
}
```

### 8.3 使用默认类型标头映射进行反序列化

设置好场景后，我们就可以创建测试了。首先，让我们为每种类型创建一个事件：
```java
@Test
void givenPayloadWithSubclasses_whenMessageReceived_thenDeserializesCorrectType() {
    var domesticOrderId = UUID.randomUUID();
    var domesticEvent = new DomesticShipmentRequestedEvent(domesticOrderId, "123 Main St", LocalDate.parse("2024-05-12"), "XPTO1234");
    var internationalOrderId = UUID.randomUUID();
    InternationalShipmentRequestedEvent internationalEvent = new InternationalShipmentRequestedEvent(internationalOrderId, "123 Main St", LocalDate.parse("2024-05-24"), "Canada", "HS Code: 8471.30, Origin: China, Value: $500");
}
```

继续使用相同的测试方法，我们现在将发送事件。**默认情况下，SqsTemplate会发送一个带有特定类型信息的标头以进行反序列化**。利用这一点，我们可以简单地使用自动配置的SqsTemplate发送消息，它会正确地反序列化消息：
```java
sqsTemplate.send(queuesProperties.getSubclassDeserializationQueue(), internationalEvent);
sqsTemplate.send(queuesProperties.getSubclassDeserializationQueue(), domesticEvent);
```

最后，我们断言每批货物的状态根据其类型对应相应的状态：
```java
await().atMost(Duration.ofSeconds(10))
    .untilAsserted(() -> {
        var domesticShipment = (DomesticShipment) shipmentService.getShipment(domesticOrderId);
        assertThat(domesticShipment).isNotNull();
        assertThat(domesticShipment).usingRecursiveComparison()
            .ignoringFields("status")
            .isEqualTo(domesticEvent);
        assertThat(domesticShipment.getStatus()).isEqualTo(ShipmentStatus.READY_FOR_DISPATCH);

        var internationalShipment = (InternationalShipment) shipmentService.getShipment(internationalOrderId);
        assertThat(internationalShipment).isNotNull();
        assertThat(internationalShipment).usingRecursiveComparison()
            .ignoringFields("status")
            .isEqualTo(internationalEvent);
        assertThat(internationalShipment.getStatus()).isEqualTo(ShipmentStatus.CUSTOMS_CHECK);
    });
```

当我们现在运行测试时，它通过了，这表明每个子类都使用正确的类型和信息正确地反序列化。

### 8.4 使用自定义类型标头映射进行反序列化

从可能不使用SqsTemplate发送消息的服务接收消息是很常见的，或者代表事件的POJO或记录可能位于不同的包中。

为了模拟这种情况，让我们在测试方法中创建一个自定义SqsTemplate，并将其配置为发送不包含标头中类型信息的消息。**对于这种情况，我们还需要注入一个能够序列化LocalDate实例的ObjectMapper，例如我们之前配置的实例或Spring Boot自动配置的实例**：
```java
@Autowired
private ObjectMapper objectMapper;

var customTemplate = SqsTemplate.builder()
    .sqsAsyncClient(sqsAsyncClient)
    .configureDefaultConverter(converter -> {
          converter.doNotSendPayloadTypeHeader();
          converter.setObjectMapper(objectMapper);
      })
    .build();

customTemplate.send(to -> to.queue(queuesProperties.getSubclassDeserializationQueue())
    .payload(internationalEvent);

customTemplate.send(to -> to.queue(queuesProperties.getSubclassDeserializationQueue())
    .payload(domesticEvent);

```

现在，我们的测试失败，并在堆栈跟踪中出现类似这些消息，因为框架无法知道将其反序列化为哪个特定类：
```text
Could not read JSON: Unrecognized field "destinationCountry"
Could not read JSON: Unrecognized field "deliveryRouteCode"
```

**为了解决此用例，SqsMessagingMessageConverter类具有setPayloadTypeMapper方法，该方法可用于让框架根据消息的任何属性了解目标类**。对于此测试，我们将使用自定义标头作为标准。

首先，让我们将标头配置添加到application.yml中：
```yaml
headers:
    types:
        shipping:
            header-name: SHIPPING_TYPE
            international: INTERNATIONAL
            domestic: DOMESTIC
```

我们还将创建一个属性类来保存这些值：
```java
@ConfigurationProperties(prefix = "headers.types.shipping")
public class ShippingHeaderTypesProperties {

    private String headerName;
    private String international;
    private String domestic;

    // ...getters and setters
}
```

接下来，让我们在配置类中启用属性类：
```java
@EnableConfigurationProperties({ ShipmentEventsQueuesProperties.class, ShippingHeaderTypesProperties.class })
@Configuration
public class ShipmentServiceConfiguration {
   // ...rest of code remains the same
}
```

我们现在将配置一个自定义SqsMessagingMessageConverter来使用这些标头并将其设置为defaultSqsListenerContainerFactory Bean：
```java
@Bean
public SqsMessageListenerContainerFactory defaultSqsListenerContainerFactory(ObjectMapper objectMapper) {
    SqsMessagingMessageConverter converter = new SqsMessagingMessageConverter();
    converter.setPayloadTypeMapper(message -> {
        if (!message.getHeaders()
                .containsKey(typesProperties.getHeaderName())) {
            return Object.class;
        }
        String eventTypeHeader = MessageHeaderUtils.getHeaderAsString(message, typesProperties.getHeaderName());
        if (eventTypeHeader.equals(typesProperties.getDomestic())) {
            return DomesticShipmentRequestedEvent.class;
        } else if (eventTypeHeader.equals(typesProperties.getInternational())) {
            return InternationalShipmentRequestedEvent.class;
        }
        throw new RuntimeException("Invalid shipping type");
    });
    converter.setObjectMapper(objectMapper);

    return SqsMessageListenerContainerFactory.builder()
            .sqsAsyncClient(sqsAsyncClient)
            .configure(configure -> configure.messageConverter(converter))
            .build();
}
```

之后，我们在测试方法中将标头添加到我们的自定义模板中：
```java
customTemplate.send(to -> to.queue(queuesProperties.getSubclassDeserializationQueue())
    .payload(internationalEvent)
    .header(headerTypesProperties.getHeaderName(), headerTypesProperties.getInternational()));

customTemplate.send(to -> to.queue(queuesProperties.getSubclassDeserializationQueue())
    .payload(domesticEvent)
    .header(headerTypesProperties.getHeaderName(), headerTypesProperties.getDomestic()));
```

当我们再次运行测试时，它通过了，断言每个事件的正确子类类型都被反序列化了。

## 9. 总结

在本文中，我们介绍了消息转换的三种常见用例：具有开箱即用设置的POJO/记录序列化和反序列化、使用自定义ObjectMapper处理不同的日期格式和其他特定配置，以及将消息反序列化为子类/接口实现的两种不同方法。

我们通过设置本地测试环境和创建实时测试来确认我们的逻辑，从而测试了每种场景。