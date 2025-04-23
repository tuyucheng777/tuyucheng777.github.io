---
layout: post
title:  Spring Cloud AWS SQS v3中的消息确认
category: springcloud
copyright: springcloud
excerpt: Spring Cloud AWS SQS
---

## 1. 概述

消息确认是消息传递系统中的标准机制，用于向消息代理发出信号，告知消息已收到，不应再次发送。**在Amazon的[SQS](https://aws.amazon.com/sqs/)(简单队列服务)中，确认是通过删除队列中的消息来执行的**。

在本教程中，我们将探讨Spring Cloud AWS SQS v3提供的三种开箱即用的确认模式：ON_SUCCESS、MANUAL和ALWAYS。

我们将使用事件驱动的场景来说明我们的用例，利用[Spring Cloud AWS SQS V3介绍文章](https://www.baeldung.com/java-spring-cloud-aws-v3-intro)中的环境和测试设置。

## 2. 依赖

我们首先导入[Spring Cloud AWS BOM](https://mvnrepository.com/artifact/io.awspring.cloud/spring-cloud-aws)，以确保我们的pom.xml中的所有依赖彼此兼容：
```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>io.awspring.cloud</groupId>
            <artifactId>spring-cloud-aws</artifactId>
            <version>3.1.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

我们还将添加Core和SQS Starter依赖：
```xml
<dependency>
    <groupId>io.awspring.cloud</groupId>
    <artifactId>spring-cloud-aws-starter-sqs</artifactId>
</dependency>
<dependency>
    <groupId>io.awspring.cloud</groupId>
    <artifactId>spring-cloud-aws-starter</artifactId>
</dependency>
```

最后，我们将添加测试所需的依赖，即使用[JUnit 5](https://www.baeldung.com/junit-5)的[LocalStack和TestContainers](https://mvnrepository.com/artifact/org.testcontainers/localstack/)、用于验证异步消息使用的[awaitility库](https://www.baeldung.com/awaitility-testing)以及用于处理断言的[AssertJ](https://www.baeldung.com/introduction-to-assertj)：
```xml
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

首先，我们将使用Testcontainers配置LocalStack环境以进行本地测试：
```java
@Testcontainers
public class BaseSqsLiveTest {

    private static final String LOCAL_STACK_VERSION = "localstack/localstack:2.3.2";

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

虽然这种设置使测试变得简单且可重复，但请注意，本教程中的代码也可以直接用于AWS。

## 4. 设置队列名称

默认情况下，**Spring Cloud AWS SQS会自动创建任何@SqsListener注解方法中指定的队列**，作为第一步，我们将在application.yaml文件中定义队列名称：
```yaml
events:
    queues:
        order-processing-retry-queue: order_processing_retry_queue
        order-processing-async-queue: order_processing_async_queue
        order-processing-no-retries-queue: order_processing_no_retries_queue
    acknowledgment:
        order-processing-no-retries-queue: ALWAYS
```

acknowledgment属性ALWAYS也将被我们监听器使用。

我们还向同一个文件中添加一些产品ID，以便在示例中使用：
```yaml
product:
    id:
        smartphone: 123e4567-e89b-12d3-a456-426614174000
        wireless-headphones: 123e4567-e89b-12d3-a456-426614174001
        laptop: 123e4567-e89b-12d3-a456-426614174002
        tablet: 123e4567-e89b-12d3-a456-426614174004
```

为了在我们的应用程序中将这些属性作为POJO获取，我们将创建两个@ConfigurationProperties类，一个用于队列：
```java
@ConfigurationProperties(prefix = "events.queues")
public class EventsQueuesProperties {

    private String orderProcessingRetryQueue;

    private String orderProcessingAsyncQueue;

    private String orderProcessingNoRetriesQueue;

    // getters and setters
}
```

另一个用于产品：
```java
@ConfigurationProperties("product.id")
public class ProductIdProperties {

    private UUID smartphone;

    private UUID wirelessHeadphones;

    private UUID laptop;

    // getters and setters
}
```

最后，我们使用[@EnableConfigurationProperties](https://www.baeldung.com/spring-boot-annotations)在@Configuration类中启用配置属性：
```java
@EnableConfigurationProperties({ EventsQueuesProperties.class, ProductIdProperties.class})
@Configuration
public class OrderProcessingConfiguration {
}
```

## 5. 确认处理成功

**@SqsListeners的默认确认模式为ON_SUCCESS**，在此模式下，如果监听器方法完成执行且未引发错误，则消息将被确认。

为了说明这种行为，我们将创建一个简单的监听器，它将接收OrderCreatedEvent，检查InventoryService，并且如果请求的商品和数量有库存，则将订单状态更改为PROCESSED。

### 5.1 创建服务

让我们首先创建OrderService，它将负责更新订单状态：
```java
@Service
public class OrderService {

    Map<UUID, OrderStatus> ORDER_STATUS_STORAGE = new ConcurrentHashMap<>();

    public void updateOrderStatus(UUID orderId, OrderStatus status) {
        ORDER_STATUS_STORAGE.put(orderId, status);
    }

    public OrderStatus getOrderStatus(UUID orderId) {
        return ORDER_STATUS_STORAGE.getOrDefault(orderId, OrderStatus.UNKNOWN);
    }
}
```

然后，我们将创建InventoryService。我们将使用Map模拟存储，并使用ProductIdProperties填充它，它是使用来自我们的application.yaml文件中的值自动装配的：
```java
@Service
public class InventoryService implements InitializingBean {

    private ProductIdProperties productIdProperties;

    private Map<UUID, Integer> inventory;

    public InventoryService(ProductIdProperties productIdProperties) {
        this.productIdProperties = productIdProperties;
    }

    @Override
    public void afterPropertiesSet() {
        this.inventory = new ConcurrentHashMap<>(Map.of(productIdProperties.getSmartphone(), 10,
                productIdProperties.getWirelessHeadphones(), 15,
                productIdProperties.getLaptop(), 5);
    }
}
```

InitializingBean接口提供了afterPropertiesSet，这是一个生命周期方法，在解析了Bean的所有依赖(在我们的例子中是ProductIdProperties Bean)后，Spring会调用该方法。

让我们添加一个checkInventory方法，该方法用于验证库存中是否有所需数量的产品。如果产品不存在，它将抛出ProductNotFoundException，如果产品存在但数量不足，它将抛出OutOfStockException。在第二种情况下，我们还将模拟随机补货，以便经过几次重试后，处理最终将成功：
```java
public void checkInventory(UUID productId, int quantity) {
    Integer stock = inventory.get(productId);
    if (stock < quantity) {
        inventory.put(productId, stock + (int) (Math.random() * 5));
        throw new OutOfStockException("Product with id %s is out of stock. Quantity requested: %s ".formatted(productId, quantity));
    };
    inventory.put(productId, stock - quantity);
}
```

### 5.2 创建监听器

一切就绪，可以创建第一个监听器了，我们将使用@Component注解并通过Spring的[构造函数依赖注入](https://www.baeldung.com/constructor-injection-in-spring)机制注入服务：
```java
@Component
public class OrderProcessingListeners {

    private static final Logger logger = LoggerFactory.getLogger(OrderProcessingListeners.class);

    private InventoryService inventoryService;

    private OrderService orderService;

    public OrderProcessingListeners(InventoryService inventoryService, OrderService orderService) {
        this.inventoryService = inventoryService;
        this.orderService = orderService;
    }
}
```

接下来我们来编写监听方法：
```java
@SqsListener(value = "${events.queues.order-processing-retry-queue}", id = "retry-order-processing-container", messageVisibilitySeconds = "1")
public void stockCheckRetry(OrderCreatedEvent orderCreatedEvent) {
    logger.info("Message received: {}", orderCreatedEvent);
    orderService.updateOrderStatus(orderCreatedEvent.id(), OrderStatus.PROCESSING);
    inventoryService.checkInventory(orderCreatedEvent.productId(), orderCreatedEvent.quantity());

    orderService.updateOrderStatus(orderCreatedEvent.id(), OrderStatus.PROCESSED);
    logger.info("Message processed successfully: {}", orderCreatedEvent);
}
```

**value属性是通过application.yaml自动装配的队列名称**，由于ON_SUCCESS是默认确认模式，因此我们不需要在注解中指定它。 

### 5.3 设置测试类

为了断言逻辑按预期工作，让我们创建一个测试类：
```java
@SpringBootTest
class OrderProcessingApplicationLiveTest extends BaseSqsLiveTest {

    private static final Logger logger = LoggerFactory.getLogger(OrderProcessingApplicationLiveTest.class);

    @Autowired
    private EventsQueuesProperties eventsQueuesProperties;

    @Autowired
    private ProductIdProperties productIdProperties;

    @Autowired
    private SqsTemplate sqsTemplate;

    @Autowired
    private OrderService orderService;

    @Autowired
    private MessageListenerContainerRegistry registry;
}
```

我们还将添加一个名为assertQueueIsEmpty的方法，在其中，我们将使用自动装配的MessageListenerContainerRegistry来获取容器，**然后停止容器以确保它没有消费任何消息**，注册表包含由@SqsListener注解创建的所有容器：
```java
private void assertQueueIsEmpty(String queueName, String containerId) {
    logger.info("Stopping container {}", containerId);
    var container = Objects
            .requireNonNull(registry.getContainerById(containerId), () -> "could not find container " + containerId);
    container.stop();
    // ...
}
```

容器停止后，我们将使用SqsTemplate查找队列中的消息。如果确认成功，则不应返回任何消息。我们还将pollTimeout设置为大于可见性超时的值，这样，如果消息尚未被删除，它将在指定的时间间隔内再次传递。

以下是assertQueueIsEmpty方法的延续：
```java
// ...
logger.info("Checking for messages in queue {}", queueName);
var message = sqsTemplate.receive(from -> from.queue(queueName)
    .pollTimeout(Duration.ofSeconds(5)));
assertThat(message).isEmpty();
logger.info("No messages found in queue {}", queueName);
```

### 5.4 测试

在第一个测试中，我们将向队列发送一个OrderCreatedEvent，其中包含一个产品订单，该订单的产品数量大于我们库存的数量。当异常通过监听器方法时，它将向框架发出信号，表示消息处理失败，并且应在消息可见时间窗口过后再次传递该消息。

为了加快测试速度，**我们在注解中将messageVisibilitySeconds设置为1，但通常，此配置是在队列本身中完成的，默认为30秒**。

我们将创建事件并使用Spring Cloud AWS提供的自动配置SqsTemplate发送它，然后，我们将使用Awaitility等待订单状态更改为PROCESSED，最后，我们将断言队列为空，这意味着确认成功：
```java
@Test
public void givenOnSuccessAcknowledgementMode_whenProcessingThrows_shouldRetry() {
    var orderId = UUID.randomUUID();
    var queueName = eventsQueuesProperties.getOrderProcessingRetryQueue();
    sqsTemplate.send(queueName, new OrderCreatedEvent(orderId, productIdProperties.getLaptop(), 10));
    Awaitility.await()
            .atMost(Duration.ofMinutes(1))
            .until(() -> orderService.getOrderStatus(orderId)
                    .equals(OrderStatus.PROCESSED));
    assertQueueIsEmpty(queueName, "retry-order-processing-container");
}
```

注意，我们将@SqsListener注解中指定的containerId传递给assertQueueIsEmpty方法。

现在我们可以运行测试了，首先，我们要确保Docker正在运行，然后执行测试。在容器初始化日志之后，我们应该看到应用程序的日志消息：
```text
Message received: OrderCreatedEvent[id=83f27bf2-1bd4-460a-9006-d784ec7eff47, productId=123e4567-e89b-12d3-a456-426614174002, quantity=10]
```

然后，应该会看到由于库存不足而导致的一个或多个故障：
```text
Caused by: cn.tuyucheng.taketoday.spring.cloud.aws.sqs.acknowledgement.exception.OutOfStockException: Product with id 123e4567-e89b-12d3-a456-426614174002 is out of stock. Quantity requested: 10
```

而且，由于我们添加了补货逻辑，我们最终应该看到消息处理成功：
```text
Message processed successfully: OrderCreatedEvent[id=83f27bf2-1bd4-460a-9006-d784ec7eff47, productId=123e4567-e89b-12d3-a456-426614174002, quantity=10]
```

最后，我们将确保确认已成功：
```text
INFO 2699 --- [main] a.s.a.OrderProcessingApplicationLiveTest : Stopping container retry-order-processing-container
INFO 2699 --- [main] a.c.s.l.AbstractMessageListenerContainer : Container retry-order-processing-container stopped
INFO 2699 --- [main] a.s.a.OrderProcessingApplicationLiveTest : Checking for messages in queue order_processing_retry_queue
INFO 2699 --- [main] a.s.a.OrderProcessingApplicationLiveTest : No messages found in queue order_processing_retry_queue
```

请注意，测试完成后可能会抛出“连接被拒绝”错误-这是因为Docker容器在框架停止轮询消息之前就停止了，我们可以放心地忽略这些错误。

## 6. 手动确认

该框架支持手动确认消息，**这对于我们需要更好地控制确认过程的场景很有用**。

### 6.1 创建监听器

为了说明这一点，我们将创建一个异步场景，其中InventoryService的连接速度很慢，我们希望在它完成之前释放监听器线程：
```java
@SqsListener(value = "${events.queues.order-processing-async-queue}", acknowledgementMode = SqsListenerAcknowledgementMode.MANUAL, id = "async-order-processing-container", messageVisibilitySeconds = "3")
public void slowStockCheckAsynchronous(OrderCreatedEvent orderCreatedEvent, Acknowledgement acknowledgement) {
    orderService.updateOrderStatus(orderCreatedEvent.id(), OrderStatus.PROCESSING);
    CompletableFuture.runAsync(() -> inventoryService.slowCheckInventory(orderCreatedEvent.productId(), orderCreatedEvent.quantity()))
        .thenRun(() -> orderService.updateOrderStatus(orderCreatedEvent.id(), OrderStatus.PROCESSED))
        .thenCompose(voidFuture -> acknowledgement.acknowledgeAsync())
        .thenRun(() -> logger.info("Message for order {} acknowledged", orderCreatedEvent.id()));
    logger.info("Releasing processing thread.");
}
```

在这个逻辑中，我们使用Java的CompletableFuture异步运行库存检查。我们将Acknowledge对象添加到监听器方法，并将SqsListenerAcknowledgementMode.MANUAL添加 到注解的acknowledgementMode属性，此属性是一个字符串，接受属性占位符和[SpEL](https://www.baeldung.com/spring-expression-language)。**仅当我们将AcknowledgementMode设置为MANUAL时，Acknowledgement对象才可用**。

请注意，在此示例中，我们利用Spring Boot自动配置(它提供了合理的默认值)和@SqsListener注解属性来在确认模式之间切换，另一种方法是声明SqsMessageListenerContainerFactory Bean，它允许设置更复杂的配置。

### 6.2 模拟慢速连接

现在，让我们将slowCheckInventory方法添加到InventoryService类，使用Thread.sleep模拟慢速连接：
```java
public void slowCheckInventory(UUID productId, int quantity) {
    simulateBusyConnection();
    checkInventory(productId, quantity);
}

private void simulateBusyConnection() {
    try {
        Thread.sleep(2000);
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
        throw new RuntimeException(e);
    }
}
```

### 6.3 测试

接下来，让我们编写测试：
```java
@Test
public void givenManualAcknowledgementMode_whenManuallyAcknowledge_shouldAcknowledge() {
    var orderId = UUID.randomUUID();
    var queueName = eventsQueuesProperties.getOrderProcessingAsyncQueue();
    sqsTemplate.send(queueName, new OrderCreatedEvent(orderId, productIdProperties.getSmartphone(), 1));
    Awaitility.await()
            .atMost(Duration.ofMinutes(1))
            .until(() -> orderService.getOrderStatus(orderId)
                    .equals(OrderStatus.PROCESSED));
    assertQueueIsEmpty(queueName, "async-order-processing-container");
}
```

这次，我们请求库存中可用的数量，因此我们不应该看到任何错误。

运行测试时，我们将看到一条日志消息，表明已收到消息：
```text
INFO 2786 --- [ing-container-1] c.t.t.s.c.a.s.a.l.OrderProcessingListeners : Message received: OrderCreatedEvent[id=013740a3-0a45-478a-b085-fbd634fbe66d, productId=123e4567-e89b-12d3-a456-426614174000, quantity=1]
```

然后我们会看到线程释放消息：
```text
INFO 2786 --- [ing-container-1] c.t.t.s.c.a.s.a.l.OrderProcessingListeners : Releasing processing thread.
```

**这是因为我们正在异步处理和确认消息**，大约两秒钟后，我们应该看到消息已被确认的日志：
```text
INFO 2786 --- [onPool-worker-1] c.t.t.s.c.a.s.a.l.OrderProcessingListeners : Message for order 013740a3-0a45-478a-b085-fbd634fbe66d acknowledged
```

最后，我们将看到停止容器并断言队列为空的日志：
```text
INFO 2786 --- [main] a.s.a.OrderProcessingApplicationLiveTest : Stopping container async-order-processing-container
INFO 2786 --- [main] a.c.s.l.AbstractMessageListenerContainer : Container async-order-processing-container stopped
INFO 2786 --- [main] a.s.a.OrderProcessingApplicationLiveTest : Checking for messages in queue order_processing_async_queue
INFO 2786 --- [main] a.s.a.OrderProcessingApplicationLiveTest : No messages found in queue order_processing_async_queue
```

## 7. 成功和错误确认

我们将探讨的最后一种确认模式是ALWAYS，它使得**框架无论监听器方法是否引发错误都会确认消息**。

### 7.1 创建监听器

让我们模拟一个促销活动，在此期间我们的库存有限，并且我们不想重新处理任何消息，无论出现任何故障，我们将使用之前在application.yml中定义的属性将确认模式设置为ALWAYS：
```java
@SqsListener(value = "${events.queues.order-processing-no-retries-queue}", acknowledgementMode = ${events.acknowledgment.order-processing-no-retries-queue}, id = "no-retries-order-processing-container", messageVisibilitySeconds = "3")
public void stockCheckNoRetries(OrderCreatedEvent orderCreatedEvent) {
    logger.info("Message received: {}", orderCreatedEvent);
    orderService.updateOrderStatus(orderCreatedEvent.id(), OrderStatus.RECEIVED);
    inventoryService.checkInventory(orderCreatedEvent.productId(), orderCreatedEvent.quantity());

    logger.info("Message processed: {}", orderCreatedEvent);
}
```

在测试中，我们将创建一个数量大于库存的订单：

### 7.2 测试
```java
@Test
public void givenAlwaysAcknowledgementMode_whenProcessThrows_shouldAcknowledge() {
    var orderId = UUID.randomUUID();
    var queueName = eventsQueuesProperties.getOrderProcessingNoRetriesQueue();
    sqsTemplate.send(queueName, new OrderCreatedEvent(orderId, productIdProperties.getWirelessHeadphones(), 20));
    Awaitility.await()
            .atMost(Duration.ofMinutes(1))
            .until(() -> orderService.getOrderStatus(orderId)
                    .equals(OrderStatus.RECEIVED));
    assertQueueIsEmpty(queueName, "no-retries-order-processing-container");
}
```

现在，即使抛出OutOfStockException，消息也会被确认，并且不会尝试重试该消息：
```text
Message received: OrderCreatedEvent[id=7587f1a2-328f-4791-8559-ee8e85b25259, productId=123e4567-e89b-12d3-a456-426614174001, quantity=20]
Caused by: cn.tuyucheng.taketoday.spring.cloud.aws.sqs.acknowledgement.exception.OutOfStockException: Product with id 123e4567-e89b-12d3-a456-426614174001 is out of stock. Quantity requested: 20
INFO 2835 --- [main] a.s.a.OrderProcessingApplicationLiveTest : Stopping container no-retries-order-processing-container
INFO 2835 --- [main] a.c.s.l.AbstractMessageListenerContainer : Container no-retries-order-processing-container stopped
INFO 2835 --- [main] a.s.a.OrderProcessingApplicationLiveTest : Checking for messages in queue order_processing_no_retries_queue
INFO 2835 --- [main] a.s.a.OrderProcessingApplicationLiveTest : No messages found in queue order_processing_no_retries_queue
```

## 8. 总结

在本文中，我们使用事件驱动的场景来展示Spring Cloud AWS v3 SQS集成提供的三种确认模式：ON_SUCCESS(默认)、MANUAL和ALWAYS。

我们利用了自动配置设置并使用@SqsListener注解属性在模式之间切换，我们还创建了实时测试，使用Testcontainers和LocalStack断言行为。