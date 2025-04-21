---
layout: post
title:  AutoMQ简介：经济高效的Kafka替代方案
category: messaging
copyright: messaging
excerpt: AutoMQ
---

## 1. 概述

[Apache Kafka](https://www.baeldung.com/apache-kafka)已成为最受欢迎且使用最广泛的消息传递和事件流平台之一，但是，设置和管理Kafka集群是一个复杂的过程，通常由大型组织中的专门团队完成，以确保高可用性、可靠性、负载平衡和扩展性。

[AutoMQ](https://github.com/AutoMQ/automq)是Apache Kafka的云原生替代方案，专注于[降低成本](https://docs.automq.com/automq/benchmarks/cost-effective-automq-vs-apache-kafka)和[提高效率](https://docs.automq.com/automq/benchmarks/benchmark-automq-vs-apache-kafka)。它采用共享存储架构，将数据存储在[Amazon Simple Storage Service(S3)](https://www.baeldung.com/java-aws-s3)中，并通过[Amazon Elastic Block Store(EBS)](https://docs.aws.amazon.com/ebs/latest/userguide/what-is-ebs)保证持久性。

**在本教程中，我们将探讨如何在[Spring Boot](https://www.baeldung.com/spring-boot)应用程序中集成AutoMQ**，我们将逐步介绍如何设置本地AutoMQ集群，并实现基本的生产者-消费者模式。

## 2. 使用Testcontainers设置AutoMQ

为了方便本地开发和测试，我们将使用[Testcontainers](https://www.baeldung.com/tag/testcontainers)搭建AutoMQ集群。通过Testcontainers运行AutoMQ集群的先决条件是：一个运行的[Docker](https://www.baeldung.com/ops/docker-guide)实例和[Docker Compose](https://www.baeldung.com/ops/docker-compose)。

**AutoMQ提供了一个用于本地部署的Docker Compose文件，该文件使用[LocalStack](https://github.com/localstack/localstack)模拟Amazon S3服务，并使用本地文件系统模拟Amazon EBS**，我们将在设置中使用这个Compose文件。

值得注意的是，以下设置不适用于生产环境。

### 2.1 依赖

让我们首先向项目的pom.xml文件添加必要的依赖：

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
    <version>3.3.0</version>
</dependency>
```

**AutoMQ与Apache Kafka完全兼容**，这意味着它们实现相同的API，并使用相同的协议和配置属性，这使我们能够使用熟悉的[Spring Kafka依赖](https://mvnrepository.com/artifact/org.springframework.kafka/spring-kafka)将AutoMQ集成到我们的应用程序中。

接下来，我们将添加几个测试依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-testcontainers</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.awaitility</groupId>
    <artifactId>awaitility</artifactId>
    <scope>test</scope>
</dependency>
```

[spring-boot-testcontainers依赖](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-testcontainers/latest)为我们提供了启动AutoMQ集群所需的临时Docker实例所需的类。

此外，我们还添加了[awaitility](https://www.baeldung.com/awaitility-testing)库，它将帮助我们在本教程的后面测试异步生产者-消费者实现。

### 2.2 定义测试容器Bean

接下来，让我们创建一个[@TestConfiguration](https://www.baeldung.com/spring-boot-testing#test-configuration-withtestconfiguration)类来定义我们的Testcontainers Bean：

```java
@TestConfiguration(proxyBeanMethods = false)
class TestcontainersConfiguration {

    private static final String COMPOSE_URL = "https://download.automq.com/community_edition/standalone_deployment/docker-compose.yaml";

    @Bean
    public ComposeContainer composeContainer() {
        File dockerCompose = downloadComposeFile();
        return new ComposeContainer(dockerCompose)
                .withLocalCompose(true);
    }

    private File downloadComposeFile() {
        File dockerCompose = Files.createTempFile("docker-compose", ".yaml").toFile();
        FileUtils.copyURLToFile(URI.create(COMPOSE_URL).toURL(), dockerCompose);
        return dockerCompose;
    }
}
```

这里我们使用Testcontainers的[Docker Compose模块](https://java.testcontainers.org/modules/docker_compose/)，首先，**我们下载AutoMQ Docker Compose文件，并根据其内容创建一个ComposeContainer Bean**。

我们使用withLocalCompose()方法并将其设置为true，指示Testcontainers使用安装在我们的开发或CI机器上的Docker Compose二进制文件。

但是，**Docker Compose的container_name属性[目前不受Testcontainers支持](https://github.com/testcontainers/testcontainers-java/issues/2472)**，让我们暂时解决这个问题：

```java
private File downloadComposeFile() {
    // ... same as above
    return removeContainerNames(dockerCompose);
}

private File removeContainerNames(File composeFile) {
    List<String> filteredLines = Files.readAllLines(composeFile.toPath())
        .stream()
        .filter(line -> !line.contains("container_name:"))
        .toList();
    Files.write(composeFile.toPath(), filteredLines);
    return composeFile;
}
```

私有的removeContainerNames()方法会从下载的Docker Compose文件中删除container_name属性，此解决方法可确保用于实例化ComposeContainer Bean的Docker Compose不包含container_name属性。

最后，为了允许我们的应用程序连接到AutoMQ集群，我们将配置bootstrap-servers属性：

```java
@Bean
public DynamicPropertyRegistrar dynamicPropertyRegistrar() {
    return registry -> {
        registry.add("spring.kafka.bootstrap-servers", () -> "localhost:9094,localhost:9095");
    };
}
```

**我们在定义DynamicPropertyRegistrar Bean时配置了localhost:9094,localhost:9095的默认AutoMQ引导服务器**。

配置正确的连接详细信息后，Spring Boot会自动创建KafkaTemplate的Bean，我们将在本教程的后面使用它。

### 2.3 在开发过程中使用测试容器

虽然Testcontainers主要用于集成测试，但我们也可以在本地开发期间使用它。

为了实现这一点，我们将在src/test/java目录中创建一个单独的主类：

```java
public class TestApplication {

    public static void main(String[] args) {
        SpringApplication.from(Application::main)
            .with(TestcontainersConfiguration.class)
            .run(args);
    }
}
```

我们创建一个TestApplication类，并在其main()方法内，使用TestcontainersConfiguration类启动我们的主Application类。

**此设置可帮助我们在本地设置和管理外部服务，我们可以运行Spring Boot应用程序，并将其连接到通过Testcontainers启动的外部服务**。

## 3. 实现生产者-消费者模式

现在我们已经设置了本地AutoMQ集群，让我们使用它实现一个基本的生产者-消费者模式。

### 3.1 配置AutoMQ消费者

首先，让我们在application.yml文件中定义消费者监听的主题名称：

```yaml
cn:
    tuyucheng:
        taketoday: 
            topic:
                onboarding-initiated: user-service.onboarding.initiated.v1
```

接下来，让我们创建一个类来消费来自配置主题的消息：

```java
@Configuration
class UserOnboardingInitiatedListener {

    private static final Logger log = LoggerFactory.getLogger(UserOnboardingInitiatedListener.class);

    @KafkaListener(topics = "${cn.tuyucheng.taketoday.topic.onboarding-initiated}", groupId = "user-service")
    public void listen(User user) {
        log.info("Dispatching user account confirmation email to {}", user.email());
    }

}

record User(String email) {
}
```

这里，**我们在listen()方法上使用@KafkaListener注解来指定主题和消费者组**，每当有消息发布到user-service.onboarding.initiated.v1主题时，都会调用此方法。

我们定义一个User[记录](https://www.baeldung.com/java-record-keyword)来表示我们的消息负载。

最后，我们将以下配置添加到application.yml文件：

```yaml
spring:
    kafka:
        consumer:
            key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
            value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
        producer:
            key-serializer: org.apache.kafka.common.serialization.StringSerializer
            value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
        properties:
            spring.json.value.default.type: cn.tuyucheng.taketoday.automq.User
            allow.auto.create.topics: true
```

我们为消费者和生产者配置了键和值的序列化和反序列化属性，此外，我们将User记录指定为默认消息负载类型。

最后，**我们启用主题的自动创建**，因此如果主题不存在，AutoMQ会自动创建一个。

### 3.2 测试消息消费

现在我们已经配置了消费者，让我们验证它是否消费并记录发布到配置主题的消息：

```java
@SpringBootTest
@ExtendWith(OutputCaptureExtension.class)
@Import(TestcontainersConfiguration.class)
class UserOnboardingInitiatedListenerLiveTest {

    @Autowired
    private KafkaTemplate<String, User> kafkaTemplate;

    @Value("${cn.tuyucheng.taketoday.topic.onboarding-initiated}")
    private String onboardingInitiatedTopic;

    @Test
    void whenMessagePublishedToTopic_thenProcessedByListener(CapturedOutput capturedOutput) {
        User user = new User("test@tuyucheng.com");
        kafkaTemplate.send(onboardingInitiatedTopic, user);

        String expectedConsumerLog = String.format("Dispatching user account confirmation email to %s", user.email());
        Awaitility
                .await()
                .atMost(1, TimeUnit.SECONDS)
                .until(() -> capturedOutput.getAll().contains(expectedConsumerLog));
    }
}
```

在这里，我们自动注入KafkaTemplate类的实例，并使用[@Value](https://www.baeldung.com/spring-value-annotation)注入存储在application.yaml文件中配置的主题名称。

我们首先创建一个User对象，并使用KafkaTemplate将其发送到已配置的主题。然后，**使用awaitility和[OutputCaptureExtension](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/test/system/OutputCaptureExtension.html)提供的CapturedOutput实例，我们断言消费者已记录了预期的日志消息**。

**我们的测试用例可能会间歇性地失败，因为消费者需要一些时间才能启动并订阅主题**，为了解决这个问题，让我们在测试用例执行之前等待消费者分配到分区：

```java
@BeforeAll
void setUp(CapturedOutput capturedOutput) {
    String expectedLog = "partitions assigned";
    Awaitility
        .await()
        .atMost(Durations.ONE_MINUTE)
        .pollDelay(Durations.ONE_SECOND)
        .until(() -> capturedOutput.getAll().contains(expectedLog));
}
```

在用@BeforeAll标注的setUp()方法中，我们最多等待一分钟，每秒轮询一次，直到CapturedOutput实例包含日志以确认分区分配。

我们的测试类还展示了awaitility库测试异步操作的强大功能。

## 4. 总结

在本文中，我们探讨了将AutoMQ集成到Spring Boot应用程序中。

使用Testcontainers的Docker Compose模块，我们启动了AutoMQ集群，创建了本地测试环境。

然后，我们实现了一个基本的生产者-消费者架构并成功测试了它。