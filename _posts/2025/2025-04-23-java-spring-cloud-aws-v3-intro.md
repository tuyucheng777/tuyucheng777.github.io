---
layout: post
title:  Spring Cloud AWS 3.0简介-SQS集成
category: springcloud
copyright: springcloud
excerpt: Spring Cloud AWS
---

## 1. 概述

[Spring Cloud AWS](https://docs.awspring.io/spring-cloud-aws/docs/3.0.4/reference/html/index.html)是一个旨在简化与AWS服务交互的项目，[SQS](https://aws.amazon.com/sqs/)是一种以可扩展方式发送和接收异步消息的AWS解决方案。

在本教程中，我们将重新**介绍Spring Cloud AWS SQS集成，该集成已针对Spring Cloud AWS 3.0完全重写**。

该框架提供了熟悉的Spring抽象来处理SQS队列，例如SqsTemplate和@SqsListener注解。

我们将通过[发送和接收消息](https://www.baeldung.com/java-spring-cloud-aws-v3-intro#send-receive-messages)的示例来介绍一个事件驱动的场景，并展示使用[Testcontainers](https://www.baeldung.com/spring-boot-built-in-testcontainers)(一种管理一次性Docker容器的工具)和[LocalStack](https://github.com/localstack/localstack)(在本地模拟类似AWS的环境以测试我们的逻辑)[设置集成测试](https://www.baeldung.com/java-spring-cloud-aws-v3-intro#set-up-it)的策略。

## 2. 依赖

[Spring Cloud AWS BOM](https://mvnrepository.com/artifact/io.awspring.cloud/spring-cloud-aws)确保项目之间的版本兼容，它声明了许多依赖(包括Spring Boot)的版本，**应该使用它来代替Spring Boot自己的BOM**。

以下我们在pom.xml文件中导入它：

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

我们需要的主要依赖是[SQS Starter](https://docs.awspring.io/spring-cloud-aws/docs/3.2.0/reference/html/index.html#sqs-integration)，它包含项目的所有与SQS相关的类。SQS集成不依赖于Spring Boot，可以在任何标准Java应用程序中独立使用：

```xml
<dependency>
    <groupId>io.awspring.cloud</groupId>
    <artifactId>spring-cloud-aws-starter-sqs</artifactId>
</dependency>
```

对于Spring Boot应用程序(例如我们在本教程中构建的应用程序)，我们应该添加项目的[Core Starter](https://docs.awspring.io/spring-cloud-aws/docs/3.0.4/reference/html/index.html#spring-cloud-aws-core)，因为它允许我们利用Spring Boot的SQS自动配置和AWS配置(例如凭证和区域)：
```xml
<dependency>
    <groupId>io.awspring.cloud</groupId>
    <artifactId>spring-cloud-aws-starter</artifactId>
</dependency>
```

## 3. 设置本地测试环境

在本节中，我们将逐步介绍如何使用Testcontainers设置LocalStack环境，以在本地环境中测试我们的代码。请注意，**本教程中的示例也可以直接针对AWS执行**。

### 3.1 依赖

为了使用[JUnit 5](https://www.baeldung.com/junit-5)运行[LocalStack和TestContainers](https://mvnrepository.com/artifact/org.testcontainers/localstack/)，我们需要两个额外的依赖：
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
```

我们还包括[awaitility库](https://www.baeldung.com/awaitility-testing)来帮助我们断言异步消息的消费：

```xml
<dependency>
    <groupId>org.awaitility</groupId>
    <artifactId>awaitility</artifactId>
    <scope>test</scope>
</dependency>
```

### 3.2 配置

接下来，我们将使用Testcontainers和LocalStack配置本地测试环境，**我们将创建一个SqsLiveTestConfiguration类**：
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

我们使用Spring Cloud AWS自3.2.0起支持的@ServiceConnection注解，这是使用LocalStack、Testcontainers和Spring Cloud AWS实现Spring Boot测试所需的全部内容。在运行测试之前，我们只需要确保Docker引擎在我们的本地环境中运行。

## 4. 设置队列名称

我们可以利用Spring Boot的application.yml属性机制来设置队列名称。

在本教程中，我们将创建3个队列：

```yaml
events:
    queues:
        user-created-by-name-queue: user_created_by_name_queue
        user-created-record-queue: user_created_record_queue
        user-created-event-type-queue: user_created_event_type_queue
```

让我们创建一个POJO来表示这些属性：

```java
@ConfigurationProperties(prefix = "events.queues")
public class EventQueuesProperties {

    private String userCreatedByNameQueue;
    private String userCreatedRecordQueue;
    private String userCreatedEventTypeQueue;

    // getters and setters
}
```

最后，我们需要在@Configuration标注的类或主Spring Application类中使用@EnableConfigurationProperties注解，以让Spring Boot知道我们想要用我们的application.yml属性填充它：

```java
@SpringBootApplication
@EnableConfigurationProperties(EventQueuesProperties.class)
public class SpringCloudAwsApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudAwsApplication.class, args);
    }
}
```

现在，当我们需要队列名称时，我们可以注入值本身或者POJO。

默认情况下，**Spring Cloud AWS SQS将在未找到队列时创建队列**，这有助于我们快速设置开发环境。在生产环境中，应用程序不应该拥有创建队列的权限，因此如果未找到队列，应用程序应该无法启动。此外，也可以将框架配置为在未找到队列时显式失败。

## 5. 发送和接收消息

使用Spring Cloud AWS向SQS发送和接收消息有多种方式，在这里，我们将介绍最常见的几种，**使用SqsTemplate发送消息，使用@SqsListener注解接收消息**。

### 5.1 场景

在我们的场景中，我们将模拟一个事件驱动的应用程序，该应用程序通过将相关信息保存在其本地存储库中来响应UserCreatedEvent。

让我们创建一个User实体：

```java
public record User(String id, String name, String email) {
}
```

让我们创建一个简单的内存UserRepository：

```java
@Repository
public class UserRepository {

    private final Map<String, User> persistedUsers = new ConcurrentHashMap<>();

    public void save(User userToSave) {
        persistedUsers.put(userToSave.id(), userToSave);
    }

    public Optional<User> findById(String userId) {
        return Optional.ofNullable(persistedUsers.get(userId));
    }

    public Optional<User> findByName(String name) {
        return persistedUsers.values().stream()
                .filter(user -> user.name().equals(name))
                .findFirst();
    }
}
```

最后，让我们创建一个UserCreatedEvent Java记录类：
```java
public record UserCreatedEvent(String id, String username, String email) {
}
```

### 5.2 设置

为了测试我们的场景，我们将创建一个SpringCloudAwsSQSLiveTest类，该类扩展了我们之前创建的BaseSqsIntegrationTest文件。我们将[自动注入](https://www.baeldung.com/spring-autowire)3个依赖项：由框架自动配置的SqsTemplate、UserRepository(以便可以断言我们的消息处理有效)以及包含队列名称的EventQueuesProperties POJO：
```java
public class SpringCloudAwsSQSLiveTest extends BaseSqsIntegrationTest {

    private static final Logger logger = LoggerFactory.getLogger(SpringCloudAwsSQSLiveTest.class);

    @Autowired
    private SqsTemplate sqsTemplate;

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private EventQueuesProperties eventQueuesProperties;

   // ...
}
```

为了包含我们的监听器，让我们创建一个UserEventListeners类并将其声明为Spring @Component：
```java
@Component
public class UserEventListeners {

    private static final Logger logger = LoggerFactory.getLogger(UserEventListeners.class);

    public static final String EVENT_TYPE_CUSTOM_HEADER = "eventType";

    private final UserRepository userRepository;

    public UserEventListeners(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    // Our listeners will be added here 
}
```

### 5.3 字符串有效负载

在第一个示例中，我们将发送一条包含字符串有效负载的消息，在监听器中接收它，并将其持久化到存储库。然后，我们将轮询存储库，以确保应用程序正确地持久化数据。

首先，让我们在测试类中创建一个发送消息的测试：
```java
@Test
void givenAStringPayload_whenSend_shouldReceive() {
    // given
    var userName = "Albert";

    // when
    sqsTemplate.send(to -> to.queue(eventQueuesProperties.getUserCreatedByNameQueue())
            .payload(userName));
    logger.info("Message sent with payload {}", userName);

    // then
    await().atMost(Duration.ofSeconds(3))
            .until(() -> userRepository.findByName(userName)
                    .isPresent());
}
```

我们应该看到类似以下内容的日志：
```text
INFO [ main] c.t.t.s.c.a.sqs.SpringCloudAwsSQSLiveTest : Message sent with payload Albert
```

然后，**请注意测试失败，因为我们还没有此队列的监听器**。

让我们设置监听器来在我们的监听器类中消费来自这个队列的消息并使测试通过：
```java
@SqsListener("${events.queues.user-created-by-name-queue}")
public void receiveStringMessage(String username) {
    logger.info("Received message: {}", username);
    userRepository.save(new User(UUID.randomUUID()
            .toString(), username, null));
}
```

现在，当我们运行测试时，我们应该在日志中看到结果：
```text
INFO [ntContainer#0-1] c.t.t.s.cloud.aws.sqs.UserEventListeners : Received message: Albert
```

测试通过。

请注意，我们正在使用Spring的属性解析功能从我们之前创建的application.yml中获取队列名称。

### 5.4 POJO和记录负载

现在我们已经发送和接收了字符串有效负载，让我们用Java记录(即我们之前创建的UserCreatedEvent)来设置一个场景。

首先，让我们编写失败测试：
```java
@Test
void givenARecordPayload_whenSend_shouldReceive() {
    // given
    String userId = UUID.randomUUID()
            .toString();
    var payload = new UserCreatedEvent(userId, "John", "john@tuyucheng.com");

    // when
    sqsTemplate.send(to -> to.queue(eventQueuesProperties.getUserCreatedRecordQueue())
            .payload(payload));

    // then
    logger.info("Message sent with payload: {}", payload);
    await().atMost(Duration.ofSeconds(3))
            .until(() -> userRepository.findById(userId)
                    .isPresent());
}
```

在测试失败之前我们应该看到类似这样的日志：
```text
INFO [ main] c.t.t.s.c.a.sqs.SpringCloudAwsSQSLiveTest : Message sent with payload: UserCreatedEvent[id=67f52cf6-c750-4200-9a02-345bda0516f8, username=John, email=john@tuyucheng.com]
```

现在，让我们创建相应的监听器来让测试通过：
```java
@SqsListener("${events.queues.user-created-record-queue}")
public void receiveRecordMessage(UserCreatedEvent event) {
    logger.info("Received message: {}", event);
    userRepository.save(new User(event.id(), event.username(), event.email()));
}
```

我们将看到输出表明消息已收到，并且测试通过：
```text
INFO [ntContainer#1-1] c.t.t.s.cloud.aws.sqs.UserEventListeners   : Received message: UserCreatedEvent[id=2d66df3d-2dbd-4aed-8fc0-ddd08416ed12, username=John, email=john@tuyucheng.com]
```

**框架将自动配置Spring Context中可用的任何ObjectMapper Bean来处理消息的序列化和反序列化**，我们可以配置自己的ObjectMapper并以多种方式自定义序列化，但这超出了本教程的范围。

### 5.5 Spring消息和标头

在最后一个场景中，我们将发送一条带有自定义标头的记录，并以Spring Message实例的形式接收该消息，以及我们添加的自定义标头和方法签名中的标准SQS标头。**框架会自动将所有SQS[消息属性](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-message-metadata.html)转换为消息标头**，包括用户提供的任何属性。

让我们首先创建失败的测试：
```java
@Test
void givenCustomHeaders_whenSend_shouldReceive() {
    // given
    String userId = UUID.randomUUID()
            .toString();
    var payload = new UserCreatedEvent(userId, "John", "john@tuyucheng.com");
    var headers = Map.<String, Object> of(EVENT_TYPE_CUSTOM_HEADER, "UserCreatedEvent");

    // when
    sqsTemplate.send(to -> to.queue(eventQueuesProperties.getUserCreatedEventTypeQueue())
            .payload(payload)
            .headers(headers));

    // then
    logger.info("Sent message with payload {} and custom headers: {}", payload, headers);
    await().atMost(Duration.ofSeconds(3))
            .until(() -> userRepository.findById(userId)
                    .isPresent());
}
```

测试失败之前应该生成类似这样的日志：
```text
INFO [ main] c.t.t.s.c.a.sqs.SpringCloudAwsSQSLiveTest  : Sent message with payload UserCreatedEvent[id=575de854-82de-44e4-8dfe-8fdc9f6ae4a1, username=John, email=john@tuyucheng.com] and custom headers: {eventType=UserCreatedEvent}

```

现在，**让我们添加相应的监听器来让测试通过**：
```java
@SqsListener("${events.queues.user-created-event-type-queue}")
public void customHeaderMessage(Message<UserCreatedEvent> message, @Header(EVENT_TYPE_CUSTOM_HEADER) String eventType,
    @Header(SQS_APPROXIMATE_FIRST_RECEIVE_TIMESTAMP) Long firstReceive) {
    logger.info("Received message {} with event type {}. First received at approximately {}.", message, eventType, firstReceive);
    UserCreatedEvent payload = message.getPayload();
    userRepository.save(new User(payload.id(), payload.username(), payload.email()));
}
```

当我们重新运行测试时，我们将看到输出，表明成功：
```text
INFO [ntContainer#2-1] c.t.t.s.cloud.aws.sqs.UserEventListeners   : Received message GenericMessage [payload=UserCreatedEvent[id=575de854-82de-44e4-8dfe-8fdc9f6ae4a1, username=John, email=john@tuyucheng.com], headers=...
```

在此示例中，我们收到一条消息，其中包含反序列化的UserCreatedEvent记录作为有效负载，并包含两个标头。为了确保整个项目的一致性，我们应该使用[SqsHeader](https://github.com/awspring/spring-cloud-aws/blob/main/spring-cloud-aws-sqs/src/main/java/io/awspring/cloud/sqs/listener/SqsHeaders.java)类常量来检索SQS标准标头。

## 6. 总结

在本文中，我们使用事件驱动的场景来介绍使用Spring Cloud AWS SQS 3.0发送和接收消息的不同示例。

我们使用LocalStack和TestContainers设置了本地环境，并配置了框架以使用适当的本地配置进行集成测试。