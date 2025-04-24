---
layout: post
title:  Spring Cloud AWS中的FIFO队列支持
category: springcloud
copyright: springcloud
excerpt: Spring Cloud AWS
---

## 1. 概述

AWS SQS中的FIFO(先进先出)队列旨在确保消息按照发送的准确顺序进行处理，并且每条消息只传递一次。

Spring Cloud AWS v3通过易于使用的抽象支持此功能，允许开发人员使用最少的样板代码处理FIFO队列功能，例如消息排序和重复数据删除。

在本教程中，我们将探讨金融交易处理系统中FIFO队列的三个实际用例：

- 确保同一账户内交易的消息严格排序
- 并行处理来自不同账户的交易，同时保持每个账户的FIFO语义
- 处理失败时处理消息重试，确保重试遵循原始消息顺序

我们将通过设置事件驱动的应用程序并创建实时测试来断言行为是否符合预期，并利用[Spring Cloud AWS SQS V3介绍文章](https://www.baeldung.com/java-spring-cloud-aws-v3-intro)中的环境和测试设置来演示这些场景。

## 2. 依赖

首先，我们将通过导入[Spring Cloud AWS BOM](https://mvnrepository.com/artifact/io.awspring.cloud/spring-cloud-aws)来管理依赖并确保版本兼容性：
```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>io.awspring.cloud</groupId>
            <artifactId>spring-cloud-aws</artifactId>
            <version>3.2.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

接下来，我们添加核心功能和[SQS集成](https://mvnrepository.com/artifact/io.awspring.cloud/spring-cloud-aws-starter-sqs)所需的[Spring Cloud AWS Starter](https://mvnrepository.com/artifact/io.awspring.cloud/spring-cloud-aws-starter)：
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

我们还将包括[Spring Boot Web Starter](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web)，**由于我们使用的是Spring Cloud AWS BOM，因此我们不需要指定其版本**：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

最后，为了进行测试，我们将使用[JUnit 5](https://www.baeldung.com/junit-5)添加[LocalStack和TestContainers](https://mvnrepository.com/artifact/org.testcontainers/localstack/)的依赖，添加用于异步操作验证的[Awaitility](https://www.baeldung.com/awaitility-testing)，以及[Spring Boot Test Starter](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-test/3.4.0)：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-testcontainers</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>localstack</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.awaitility</groupId>
    <artifactId>awaitility</artifactId>
    <scope>test</scope>
</dependency>
```

## 3. 设置本地测试环境

接下来，我们将使用Testcontainers和LocalStack配置本地测试环境，我们将创建一个SqsLiveTestConfiguration类：
```java
@Configuration
public class SqsLiveTestConfiguration {
    private static final String LOCAL_STACK_VERSION = "localstack/localstack:3.4.0";

    @Bean
    @ServiceConnection
    LocalStackContainer localStackContainer() {
        return new LocalStackContainer(DockerImageName.parse(LOCAL_STACK_VERSION));
    }
}
```

在这个类中，我们将LocalStack测试容器声明为Spring Bean，并使用@ServiceConnection注解来代表我们处理连接。

## 4. 设置队列名称

我们将在application.yml文件中定义SQS队列名称，利用Spring Boot的配置外部化功能：
```yaml
events:
    queues:
        fifo:
            transactions-queue: "transactions-queue.fifo"
            slow-queue: "slow-queue.fifo"
            failure-queue: "failure-queue.fifo"
```

此结构将队列名称按层级结构组织起来，便于我们在应用程序代码中管理和访问它们，**.fifo后缀对于SQS中的FIFO队列是必需的**。

## 5. 设置应用程序

让我们使用交易微服务的实际示例来说明这些概念，该服务将处理交易事件(TransactionEvent)消息，这些消息表示每个账户内必须保持有序的金融交易。

首先，我们定义Transaction实体：
```java
public record Transaction(UUID transactionId, UUID accountId, double amount, TransactionType type) {}
```

以及TransactionType枚举：
```java
public enum TransactionType {
    DEPOSIT,
    WITHDRAW
}
```

接下来，我们创建TransactionEvent：
```java
public record TransactionEvent(UUID transactionId, UUID accountId, double amount, TransactionType type) {
    public Transaction toEntity() {
        return new Transaction(transactionId, accountId, amount, type);
    }
}
```

TransactionService类负责处理逻辑，并维护一个模拟Repository以用于测试目的：
```java
@Service
public class TransactionService {
    private static final Logger logger = LoggerFactory.getLogger(TransactionService.class);

    private final ConcurrentHashMap<UUID, List<Transaction>> processedTransactions =
            new ConcurrentHashMap<>();

    public void processTransaction(Transaction transaction) {
        logger.info("Processing transaction: {} for account {}",
                transaction.transactionId(), transaction.accountId());
        processedTransactions.computeIfAbsent(transaction.accountId(), k -> new ArrayList<>())
                .add(transaction);
    }

    public List<Transaction> getProcessedTransactionsByAccount(UUID accountId) {
        return processedTransactions.getOrDefault(accountId, new ArrayList<>());
    }
}
```

## 6. 按顺序处理事件

在第一个场景中，我们将创建一个处理事件的监听器，并创建一个测试来断言我们接收事件的顺序与发送顺序相同。我们将使用@RepeatedTest注解运行测试100次以确保其一致性，并查看它在标准SQS队列(而不是FIFO)中的表现。

### 6.1 创建监听器

让我们创建第一个监听器来按顺序接收和处理事件，我们将使用@SqsListener注解，并利用Spring的占位符解析功能从application.yml文件中解析队列名称：
```java
@Component
public class TransactionListener {
    private final TransactionService transactionService;

    public TransactionListener(TransactionService transactionService) {
        this.transactionService = transactionService;
    }

    @SqsListener("${events.queues.fifo.transactions-queue}")
    public void processTransaction(TransactionEvent transactionEvent) {
        transactionService.processTransaction(transactionEvent.toEntity());
    }
}
```

请注意，无需进一步设置，**框架将在后台检测队列类型是否为FIFO，并进行所有必要的调整，以确保监听器方法以正确的顺序接收消息**。

### 6.2 创建测试

让我们创建一个测试，断言接收消息的顺序与发送消息的顺序完全一致，我们先创建一个测试套件，它扩展了之前创建的BaseSqsLiveTest：
```java
@SpringBootTest
public class SpringCloudAwsSQSTransactionProcessingTest extends BaseSqsLiveTest {
    @Autowired
    private SqsTemplate sqsTemplate;

    @Autowired
    private TransactionService transactionService;

    @Value("${events.queues.fifo.transactions-queue}")
    String transactionsQueue;

    @Test
    void givenTransactionsFromSameAccount_whenSend_shouldReceiveInOrder() {
        var accountId = UUID.randomUUID();
        var transactions = List.of(createDeposit(accountId, 100.0),
                createWithdraw(accountId, 50.0), createDeposit(accountId, 25.0));
        var messages = createTransactionMessages(accountId, transactions);

        sqsTemplate.sendMany(transactionsQueue, messages);

        await().atMost(Duration.ofSeconds(5))
                .until(() -> transactionService.getProcessedTransactionsByAccount(accountId),
                        isEqual(eventsToEntities(transactions)));
    }
}
```

**在此测试中，我们利用SqsTemplate的sendMany()方法，该方法使我们能够在同一批次中发送最多10条消息**，然后我们最多等待5秒钟才能按顺序接收消息。

我们还将创建一些辅助方法来帮助我们保持测试逻辑清晰，sendMany()方法需要一个List<Pojo\>，因此createTransactionMessages()方法将accountId的每个交易映射到一条消息：
```java
private List<Message<TransactionEvent>> createTransactionMessages(UUID accountId,
                                                                  Collection<TransactionEvent> transactions) {
    return transactions.stream()
            .map(transaction -> MessageBuilder.withPayload(transaction)
                    .setHeader(SqsHeaders.MessageSystemAttributes.SQS_MESSAGE_GROUP_ID_HEADER,
                            accountId.toString())
                    .build())
            .toList();
}
```

**在SQS FIFO中，MessageGroupId属性用于通知哪些消息应分组在一起并按顺序接收**。在我们的场景中，我们必须确保一个帐户的交易保持有序，但我们不需要帐户之间的任何顺序，因此我们将使用accountId作为MessageGroupId。**为此，我们可以使用SqsHeaders中的标头，框架会将它们映射到SQS消息属性**：

其余的辅助方法是将事件映射到交易并创建TransactionEvents的简单方法：
```java
private List<Transaction> eventsToEntities(List<TransactionEvent> transactionEvents) {
    return transactionEvents.stream()
            .map(TransactionEvent::toEntity)
            .toList();
}

private TransactionEvent createWithdraw(UUID accountId1, double amount) {
    return new TransactionEvent(UUID.randomUUID(), accountId1, amount, TransactionType.WITHDRAW);
}

private TransactionEvent createDeposit(UUID accountId1, double amount) {
    return new TransactionEvent(UUID.randomUUID(), accountId1, amount, TransactionType.DEPOSIT);
}
```

### 6.3 运行测试

当我们运行测试时，我们将看到测试通过并生成类似这样的日志，并且交易按照我们声明的顺序发生：
```text
TransactionService : Processing transaction: DEPOSIT:100.0 for account f97876f9-5ef9-4b62-a69d-a5d87b5b8e7e
TransactionService : Processing transaction: WITHDRAW:50.0 for account f97876f9-5ef9-4b62-a69d-a5d87b5b8e7e
TransactionService : Processing transaction: DEPOSIT:25.0 for account f97876f9-5ef9-4b62-a69d-a5d87b5b8e7e
```

如果我们仍然不相信，想确保这不是巧合，我们可以添加@RepeatableTest注解来运行测试100次。
```java
@RepeatedTest(100)
void givenTransactionsFromSameAccount_whenSend_shouldReceiveInOrder() {
   // ...test remains the same
}
```

所有100次运行都应以相同的顺序通过日志。

**为了进行额外的健全性检查，让我们使用标准队列而不是FIFO，并验证它的行为方式**。

为此，我们需要从application.yml中的队列名称中删除.fifo后缀：
```yaml
transactions-queue: "transactions-queue"
```

接下来，我们将注释掉在createTransactionMessages()方法中添加MessageId标头的代码，因为标准SQS队列不支持该属性：
```java
// .setHeader(SqsHeaders.MessageSystemAttributes.SQS_MESSAGE_GROUP_ID_HEADER, accountId.toString())
```

现在，让我们再次运行测试100次。**你会注意到，测试有时会通过，因为消息恰好按照预期的顺序到达，但有时它会失败，因为标准队列中无法保证消息的顺序**。

在结束本节之前，让我们撤消这些更改并在队列中添加.fifo后缀，删除@RepeatedTest注解，并取消注释MessageGroupId代码。

## 7. 并行处理多个消息组

**在SQS FIFO中，为了最大化消息消费吞吐量，我们可以并行处理来自不同消息组的消息，同时保持消息组内的消息顺序**。Spring Cloud AWS SQS开箱即用地支持该行为，无需进一步配置。

为了说明这种行为，让我们向TransactionService添加一个模拟慢速连接的方法：
```java
public void simulateSlowProcessing(Transaction transaction) {
    try {
        processTransaction(transaction);
        Thread.sleep(Thread.sleep(100));
        logger.info("Transaction processing completed: {}:{} for account {}",
                transaction.type(), transaction.amount(), transaction.accountId());
    } catch (InterruptedException e) {
        Thread.currentThread()
                .interrupt();
        throw new RuntimeException(e);
    }
}
```

慢速连接将帮助我们确保来自不同账户的消息正在并行处理，同时保留每个账户内的交易顺序。

现在，让我们创建一个监听器，它将使用TransactionListener类中的新方法：
```java
@SqsListener("${events.queues.fifo.slow-queue}")
public void processParallelTransaction(TransactionEvent transactionEvent) {
    transactionService.simulateSlowProcessing(transactionEvent.toEntity());
}
```

最后，让我们创建一个测试来断言行为：
```java
@Test
void givenTransactionsFromDifferentAccounts_whenSend_shouldProcessInParallel() {
    var accountId1 = UUID.randomUUID();
    var accountId2 = UUID.randomUUID();

    var account1Transactions = List.of(createDeposit(accountId1, 100.0),
            createWithdraw(accountId1, 50.0), createDeposit(accountId1, 25.0));
    var account2Transactions = List.of(createDeposit(accountId2, 50.0),
            createWithdraw(accountId2, 25.0), createDeposit(accountId2, 50.0));

    var allMessages = Stream.concat(createTransactionMessages(accountId1, account1Transactions).stream(),
            createTransactionMessages(accountId2, account2Transactions).stream()).toList();

    sqsTemplate.sendMany(slowQueue, allMessages);

    await().atMost(Duration.ofSeconds(5))
            .until(() -> transactionService.getProcessedTransactionsByAccount(accountId1),
                    isEqual(eventsToEntities(account1Transactions)));

    await().atMost(Duration.ofSeconds(5))
            .until(() -> transactionService.getProcessedTransactionsByAccount(accountId2),
                    isEqual(eventsToEntities(account2Transactions)));
}
```

在此测试中，我们为两个不同的帐户发送两组交易事件。我们再次利用sendMany()方法在同一批次中发送所有消息，并断言消息是按照预期顺序接收的。

当我们运行测试时，应该看到类似这样的日志：
```text
TransactionService : Processing transaction: DEPOSIT:50.0 for account 639eba64-a40d-458a-be74-2457dff9d6d1
TransactionService : Processing transaction: DEPOSIT:100.0 for account 1a813756-520c-4713-a0ed-791b66e4551c
TransactionService : Transaction processing completed: DEPOSIT:100.0 for account 1a813756-520c-4713-a0ed-791b66e4551c
TransactionService : Transaction processing completed: DEPOSIT:50.0 for account 639eba64-a40d-458a-be74-2457dff9d6d1
TransactionService : Processing transaction: WITHDRAW:50.0 for account 1a813756-520c-4713-a0ed-791b66e4551c
TransactionService : Processing transaction: WITHDRAW:25.0 for account 639eba64-a40d-458a-be74-2457dff9d6d1
TransactionService : Transaction processing completed: WITHDRAW:50.0 for account 1a813756-520c-4713-a0ed-791b66e4551c
TransactionService : Transaction processing completed: WITHDRAW:25.0 for account 639eba64-a40d-458a-be74-2457dff9d6d1
TransactionService : Processing transaction: DEPOSIT:50.0 for account 639eba64-a40d-458a-be74-2457dff9d6d1
TransactionService : Processing transaction: DEPOSIT:25.0 for account 1a813756-520c-4713-a0ed-791b66e4551c
```

**可以看到两个账户正在并行处理，同时保持每个账户内的顺序，这也通过测试通过得到了验证**。

## 8. 按顺序重试处理

在最后一个场景中，我们将模拟网络故障并确保处理顺序保持一致。**当监听器方法抛出错误时，框架将暂停该消息组的执行并且不确认这些消息**。可见性窗口过期后，SQS会再次处理剩余的消息。

为了说明这种行为，我们将向TransactionService添加一种新方法，该方法在第一次处理消息时始终会失败。

首先，让我们添加一个Set来保存已经失败的ID：
```java
private final Set<UUID> failedTransactions = ConcurrentHashMap.newKeySet();
```

然后我们添加processTransactionWithFailure()方法：
```java
public void processTransactionWithFailure(Transaction transaction) {
    if (!failedTransactions.contains(transaction.transactionId())) {
        failedTransactions.add(transaction.transactionId());
        throw new RuntimeException("Simulated failure for transaction " + transaction.type() + ":" + transaction.amount());
    }
    processTransaction(transaction);
}
```

**此方法在第一次处理交易时会抛出错误，但在后续重试中会正常处理**。

现在，让我们添加监听器来处理消息。我们将messageVisibilitySeconds设置为1，以缩小可见性窗口并加快测试中的重试速度：
```java
@SqsListener(value = "${events.queues.fifo.failure-queue}", messageVisibilitySeconds = "1")
public void retryFailedTransaction(TransactionEvent transactionEvent) {
    transactionService.processTransactionWithFailure(transactionEvent.toEntity());
}
```

最后，让我们创建一个测试来断言行为是否符合预期：
```java
@Test
void givenTransactionProcessingFailure_whenSend_shouldRetryInOrder() {
    var accountId = UUID.randomUUID();
    var transactions = List.of(createDeposit(accountId, 100.0),
            createWithdraw(accountId, 50.0), createDeposit(accountId, 25.0));
    var messages = createTransactionMessages(accountId, transactions);

    sqsTemplate.sendMany(failureQueue, messages);

    await().atMost(Duration.ofSeconds(10))
            .until(() -> transactionService.getProcessedTransactionsByAccount(accountId),
                    isEqual(eventsToEntities(transactions)));
}
```

在这个测试中，我们发送了3个事件并断言它们按照预期的顺序进行处理。

当我们运行测试时，我们应该在异常堆栈跟踪中看到类似这样的日志：
```text
Caused by: java.lang.RuntimeException: Simulated failure for transaction DEPOSIT:100.0

```

其次是：
```text
TransactionService : Processing transaction: DEPOSIT:100.0 for account 3f684ccb-80e8-4e40-9136-c3b59bdd980b
```

表明该事件在第二次尝试时已成功处理。

在接下来的事件中，我们应该会看到2个类似的对：
```text
Caused by: java.lang.RuntimeException: Simulated failure for transaction WITHDRAW:50.0
TransactionService : Processing transaction: WITHDRAW:50.0 for account 3f684ccb-80e8-4e40-9136-c3b59bdd980b
Caused by: java.lang.RuntimeException: Simulated failure for transaction DEPOSIT:25.0
TransactionService : Processing transaction: DEPOSIT:25.0 for account 3f684ccb-80e8-4e40-9136-c3b59bdd980b
```

**这表明即使出现故障，事件也按照正确的顺序进行处理**。

## 9. 总结

在本文中，我们探讨了Spring Cloud AWS v3对FIFO队列的支持。我们创建了一个事务处理服务，该服务依赖于按顺序处理的事件，并在三种不同情况下尊重断言的消息顺序：处理单个消息组、并行处理多个消息组以及在失败后重试消息。

我们通过设置本地测试环境和创建实时测试来确认我们的逻辑，从而测试了每种场景。