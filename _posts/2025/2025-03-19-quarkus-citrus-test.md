---
layout: post
title:  使用Citrus测试Quarkus
category: quarkus
copyright: quarkus
excerpt: Quarkus
---

## 1. 概述

**[Quarkus](https://www.baeldung.com/quarkus-io)承诺提供小巧的工件、极快的启动时间和更短的首次请求时间**，我们可以将其理解为一个集成Java标准技术(Jakarta EE、MicroProfile等)的框架，并允许构建可部署在任何容器运行时中的独立应用程序，轻松满足云原生应用程序的要求。

在本文中，我们将学习如何使用[Citrus](https://citrusframework.org/)(由RedHat首席软件工程师[Christoph Deppisch](https://github.com/christophd)编写的框架)实现集成测试。

## 2. Citrus的用途

我们开发的应用程序通常不是独立运行的，而是与其他系统(例如数据库、消息传递系统或在线服务)进行通信。**在测试我们的应用程序时，我们可以通过Mock相应的对象以独立的方式进行测试。但我们也可能想测试我们的应用程序与外部系统的通信，这就是Citrus发挥作用的地方**。

让我们仔细看看最常见的交互场景。

### 2.1 HTTP

我们的Web应用程序可能具有基于HTTP的API(例如REST API)。**Citrus可以充当HTTP客户端，调用我们应用程序的HTTP API并验证响应(就像[REST-Assured](https://www.baeldung.com/rest-assured-tutorial)所做的那样)**。我们的应用程序也可能是另一个应用程序HTTP API的使用者，在这种情况下，Citrus可以运行嵌入式HTTP服务器并充当Mock：

![](/assets/images/2025/quarkus/quarkuscitrustest01.png)

### 2.2 Kafka

在这种情况下，我们的应用程序是[Kafka](https://www.baeldung.com/apache-kafka)消费者。**Citrus可以充当Kafka生产者，将记录发送到主题，以便我们的应用程序通过消费该记录来触发**。我们的应用程序也可以是Kafka生产者。

Citrus可以充当消费者，在测试期间验证我们的应用程序发送到主题的消息。此外，Citrus还提供了一个嵌入式Kafka服务器，在测试期间独立于任何外部服务器：

![](/assets/images/2025/quarkus/quarkuscitrustest02.png)

### 2.3 关系型数据库

我们的应用程序可能使用关系型数据库，**Citrus可以充当JDBC客户端，用于验证数据库是否具有预期状态**。此外，Citrus还提供了JDBC驱动程序和嵌入式数据库Mock，可以对其进行检测以返回特定于测试用例的结果并验证已执行的数据库查询：

![](/assets/images/2025/quarkus/quarkuscitrustest03.png)

### 2.4 进一步支持

Citrus支持更多外部系统，例如REST、SOAP、JMS、Websocket、Mail、FTP和Apache Camel端点，我们可以在[文档](https://citrusframework.org/citrus/reference/4.2.1/html/index.html)中找到完整列表。

## 3. 使用Citrus测试Quarkus

**Quarkus为[编写集成测试](https://www.baeldung.com/java-quarkus-testing)提供了广泛的支持，包括Mock、测试Profile和测试本机可执行文件**。Citrus提供了[QuarkusTest运行时](https://citrusframework.org/citrus/reference/4.2.1/html/index.html#runtime-quarkus)，这是一个[Quarkus测试资源](https://quarkus.io/guides/getting-started-testing#quarkus-test-resource)，可通过包含Citrus功能来扩展基于Quarkus的测试。

让我们来看一个使用最常见技术的示例-REST服务提供者，它将数据存储在关系型数据库中，并在创建新元素时向Kafka发送消息。对于Citrus来说，我们如何详细实现这一点并不重要。我们的应用程序是一个黑匣子，只有外部系统和通信渠道才是至关重要的：

![](/assets/images/2025/quarkus/quarkuscitrustest04.png)

### 3.1 Maven依赖项

为了在基于Quarkus的项目中利用Citrus，我们可以使用[citrus-bom](https://mvnrepository.com/artifact/org.citrusframework/citrus-bom/)：

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.citrusframework</groupId>
            <artifactId>citrus-bom</artifactId>
            <version>4.2.1</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.citrusframework</groupId>
        <artifactId>citrus-quarkus</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

根据所使用的技术，我们可以选择性地添加更多模块：

```xml
<dependency>
    <groupId>org.citrusframework</groupId>
    <artifactId>citrus-openapi</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.citrusframework</groupId>
    <artifactId>citrus-http</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.citrusframework</groupId>
    <artifactId>citrus-validation-json</artifactId>
</dependency>
<dependency>
    <groupId>org.citrusframework</groupId>
    <artifactId>citrus-validation-hamcrest</artifactId>
    <version>${citrus.version}</version>
</dependency>
<dependency>
    <groupId>org.citrusframework</groupId>
    <artifactId>citrus-sql</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.citrusframework</groupId>
    <artifactId>citrus-kafka</artifactId>
    <scope>test</scope>
</dependency>
```

### 3.2 应用程序配置

**Citrus不需要进行任何全局Quarkus配置**，日志中只有关于拆分包的警告，我们可以通过在application.properties文件中添加此行来避免这种情况：

```properties
%test.quarkus.arc.ignored-split-packages=org.citrusframework.*
```

### 3.3 边界测试设置

Citrus的典型测试包含以下内容：

- @CitrusSupport注解，添加Quarkus测试资源以扩展基于Quarkus的测试处理
- @CitrusConfiguration注解，其中包含一个或多个Citrus配置类，用于通信端点的全局配置和对测试类的依赖注入
- 字段以获取端点和其他Citrus提供的对象注入

因此，如果我们想测试边界，我们需要一个HTTP客户端向我们的应用程序发送请求并验证响应。首先，我们需要创建Citrus配置类：

```java
public class BoundaryCitrusConfig {

    public static final String API_CLIENT = "apiClient";

    @BindToRegistry(name = API_CLIENT)
    public HttpClient apiClient() {
        return http()
                .client()
                .requestUrl("http://localhost:8081")
                .build();
    }
}
```

然后，我们创建测试类：

```java
@QuarkusTest
@CitrusSupport
@CitrusConfiguration(classes = {
        BoundaryCitrusConfig.class
})
class CitrusTests {

    @CitrusEndpoint(name = BoundaryCitrusConfig.API_CLIENT)
    HttpClient apiClient;
}
```

按照惯例，如果声明方法和测试类中的字段具有相同的名称，我们可以跳过注解的name属性。这可能更短，但由于缺少编译器检查，容易出错。

### 3.4 测试边界

为了编写测试，我们需要知道Citrus有一个声明性概念，定义了组件：

![](/assets/images/2025/quarkus/quarkuscitrustest05.png)

- Test Context是一个提供测试变量和函数的对象，它可以替换消息有效负载和标头中的动态内容。
- Test Action是测试中每个步骤的抽象，这可能是一个交互，例如发送请求或接收响应，包括验证和确认。它也可能只是一个简单的输出或一个计时器。Citrus提供了Java DSL和[XML](https://citrusframework.org/citrus/reference/4.2.1/html/index.html#run-xml-tests)作为使用测试操作定义测试定义的替代方案，我们可以在[文档](https://citrusframework.org/citrus/reference/4.2.1/html/index.html#actions)中找到预定义测试操作的列表。
- Test Action Builder用于定义和构建测试操作，Citrus在这里使用了构建器模式。
- Test Action Runner使用测试操作生成器来构建测试操作，然后，它执行测试操作，提供测试上下文。对于BBD风格，我们可以使用GherkinTestActionRunner。

我们也可以注入Test Action Runner。以下是一个测试，它使用JSON体向http://localhost:8081/api/v1/todos发送HTTP POST请求，并期望收到带有201状态代码的响应：

```java
@CitrusResource
GherkinTestActionRunner t;

@Test
void shouldReturn201OnCreateItem() {
    t.when(
            http()
                    .client(apiClient)
                    .send()
                    .post("/api/v1/todos")
                    .message()
                    .contentType(MediaType.APPLICATION_JSON)
                    .body("{\"title\": \"test\"}")
    );
    t.then(
            http()
                    .client(apiClient)
                    .receive()
                    .response(HttpStatus.CREATED)
    );
}
```

正文直接以JSON字符串形式提供，或者，我们可以使用[此示例](https://github.com/citrusframework/citrus-samples/blob/1744b7c8f2ba109cebe5da70ce33ae0d9c697ecd/demo/sample-quarkus/src/test/java/org/apache/camel/demo/FoodMarketOpenApiTest.java#L87C43-L87C63)中所示的数据字典。

对于消息验证，我们有[多种可能性](https://citrusframework.org/citrus/reference/4.2.1/html/index.html#message-validation)。例如，使用JSON-Path与Hamcrest组合，我们可以扩展then块：

```java
t.then(
    http()
        .client(apiClient)
        .receive()
        .response(HttpStatus.CREATED)
        .message()
        .type(MessageType.JSON)
        .validate(
            jsonPath()
                .expression("$.title", "test")
                .expression("$.id", is(notNullValue()))
        )
);
```

不幸的是，仅支持Hamcrest。对于AssertJ，2016年创建了一个[GitHub issues](https://github.com/citrusframework/citrus/issues/168)。

### 3.5 基于OpenAPI测试边界

我们还可以根据OpenAPI定义发送请求，**这会自动验证与OpenAPI模式中声明的属性和标头约束有关的响应**。

首先，我们需要加载OpenAPI模式。例如，如果我们的项目中有一个YML文件，我们可以通过定义OpenApiSpecification字段来执行此操作：

```java
final OpenApiSpecification apiSpecification = OpenApiSpecification.from(
    Resources.create("classpath:openapi.yml")
);
```

如果可用的话，我们还可以从正在运行的Quarkus应用程序中读取OpenAPI：

```java
final OpenApiSpecification apiSpecification = OpenApiSpecification.from(
    "http://localhost:8081/q/openapi"
);
```

为了测试，我们可以引用operationId来发送请求或者验证响应：

```java
t.when(
    openapi()
        .specification(apiSpecification)
        .client(apiClient)
        .send("createTodo") // operationId
);
t.then(
    openapi()
        .specification(apiSpecification)
        .client(apiClient)
        .receive("createTodo", HttpStatus.CREATED)
);
```

这将通过创建随机值来生成包含必要主体的请求，目前，无法对标头、参数或主体使用明确定义的值(请参阅此[GitHub Issue](https://github.com/citrusframework/citrus/issues/1168))。此外，生成随机日期值时存在[错误](https://github.com/citrusframework/citrus/issues/1166)，我们至少可以通过跳过随机值来避免可选字段出现此问题：

```java
@BeforeEach
void setup() {
    this.apiSpecification.setGenerateOptionalFields(false);
    this.apiSpecification.setValidateOptionalFields(false);
}
```

在这种情况下，我们还必须[禁用严格验证](https://citrusframework.org/citrus/reference/4.2.1/html/index.html#json-message-validation)，这将失败，因为服务返回可选字段(请参阅此[GitHub Issue](https://github.com/citrusframework/citrus/issues/1173))。我们可以通过使用JUnit Pioneer来实现这一点，为此，我们添加了[junit-pioneer](https://mvnrepository.com/artifact/org.junit-pioneer/junit-pioneer/)依赖：

```xml
<dependency>
    <groupId>org.junit-pioneer</groupId>
    <artifactId>junit-pioneer</artifactId>
    <version>2.2.0</version>
    <scope>test</scope>
</dependency>
```

然后，我们可以在@CitrusSupport注解之前将@SystemProperty注解添加到我们的测试类中：

```java
@SetSystemProperty(
    key = "citrus.json.message.validation.strict",
    value = "false"
)
```

### 3.6 测试数据库访问

当我们调用REST API的创建操作时，它应该将新元素存储在数据库中。为了评估这一点，**我们可以查询数据库以查找新创建的ID**。

首先，我们需要一个数据源。我们可以轻松地从Quarkus中注入它：

```java
@Inject
DataSource dataSource;
```

然后，我们需要从响应主体中提取新创建元素的ID，并将其存储为测试上下文变量：

```java
t.when(
    http()
        .client(apiClient)
        .send()
        .post("/api/v1/todos")
        .message()
        .contentType(MediaType.APPLICATION_JSON)
        .body("{"\title\": "\test\"}")
);
t.then(
    http()
        .client(apiClient)
        .receive()
        .response(HttpStatus.CREATED)
        // save new id to test context variable "todoId"
        .extract(fromBody().expression("$.id", "todoId"))
);
```

现在我们可以使用使用变量的查询来检查数据库：

```java
t.then(
    sql()
        .dataSource(dataSource)
        .query()
        .statement("select title from todos where id=${todoId}")
        .validate("title", "test")
);
```

### 3.7 测试消息

当我们调用REST API的创建操作时，它应该将新元素发送到Kafka主题。为了评估这一点，我们可以订阅该主题并消费该消息。

为此，我们需要一个Citrus端点：

```java
public class KafkaCitrusConfig {

    public static final String TODOS_EVENTS_TOPIC = "todosEvents";

    @BindToRegistry(name = TODOS_EVENTS_TOPIC)
    public KafkaEndpoint todosEvents() {
        return kafka()
                .asynchronous()
                .topic("todo-events")
                .build();
    }
}
```

然后，我们希望Citrus将此端点注入到我们的测试中：

```java
@QuarkusTest
@CitrusSupport
@CitrusConfiguration(classes = {
        BoundaryCitrusConfig.class,
        KafkaCitrusConfig.class
})
class MessagingCitrusTest {

    @CitrusEndpoint(name = KafkaCitrusConfig.TODOS_EVENTS_TOPIC)
    KafkaEndpoint todosEvents;

    // ...
}
```

如前所述，发送和接收请求后，我们可以订阅主题并消费和验证消息：

```java
t.and(
    receive()
        .endpoint(todosEvents)
        .message()
        .type(MessageType.JSON)
        .validate(
            jsonPath()
                .expression("$.title", "test")
                .expression("$.id", "${todoId}")
        )
);
```

### 3.8 Mock服务器

**Citrus可以Mock外部系统**，这有助于避免出于测试目的而需要这些外部系统，并直接验证发送到这些系统的消息并Mock响应，而不是在消息处理后验证系统的状态。

对于Kafka，[Quarkus Dev Services](https://quarkus.io/guides/kafka-dev-services)功能运行带有Kafka服务器的Docker容器。我们可以改用Citrus Mock，然后，我们必须在application.properties文件中禁用Dev Services功能：

```properties
%test.quarkus.kafka.devservices.enabled=false
```

然后，我们配置Citrus Mock服务器：

```java
public class EmbeddedKafkaCitrusConfig {

    private EmbeddedKafkaServer kafkaServer;

    @BindToRegistry
    public EmbeddedKafkaServer kafka() {
        if (null == kafkaServer) {
            kafkaServer = new EmbeddedKafkaServerBuilder()
                    .kafkaServerPort(9092)
                    .topics("todo-events")
                    .build();
        }
        return kafkaServer;
    }

    // stop the server after the test
    @BindToRegistry
    public AfterSuite afterSuiteActions() {
        return afterSuite()
                .actions(context -> kafka().stop())
                .build();
    }
}
```

然后我们可以通过引用已知的配置类来激活Mock服务器：

```java
@QuarkusTest
@CitrusSupport
@CitrusConfiguration(classes = {
        BoundaryCitrusConfig.class,
        KafkaCitrusConfig.class,
        EmbeddedKafkaCitrusConfig.class
})
class MessagingCitrusTest {

    // ...
}
```

我们还可以找到用于外部[HTTP服务](https://citrusframework.org/citrus/reference/4.2.1/html/index.html#http-rest-server)和[关系型数据库](https://citrusframework.org/citrus/reference/4.2.1/html/index.html#jdbc-server)的Mock服务器。

## 4. 挑战

使用Citrus编写测试也存在挑战，API并不总是直观的，缺少Assert集成。验证失败时，Citrus会抛出异常而不是AssertionError，导致测试报告混乱。在线文档非常详尽，但代码示例包含Groovy代码，有时还包含XML。GitHub中有一个[包含Java代码示例的仓库](https://github.com/citrusframework/citrus-samples)，可能会有所帮助。Javadocs 不完整。

似乎重点是与Spring框架的集成，文档经常提到Spring中的Citrus配置。citrus-jdbc模块依赖于Spring Core和Spring JDBC，除非我们排除它们，否则它们将成为我们测试中不必要的传递依赖项。

## 5. 总结

在本教程中，我们学习了如何使用Citrus实现Quarkus测试。**Citrus提供了许多功能来测试我们的应用程序与外部系统的通信**，这还包括Mock这些系统进行测试。它有很好的文档记录，但所包含的代码示例适用于除集成到Quarkus之外的其他用例。幸运的是，有一个[GitHub仓库](https://github.com/citrusframework/citrus-samples/tree/main/demo/sample-quarkus)包含Quarkus的示例。