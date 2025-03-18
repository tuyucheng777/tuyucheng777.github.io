---
layout: post
title:  如何使用Moco轻松设置存根服务器
category: mock
copyright: mock
excerpt: Moco
---

## 1. 简介

在软件开发过程中，几乎不可能低估测试的重要性。如果我们谈论测试，就很难忽视Mock工具，它们通常在确保稳健覆盖方面发挥着重要作用。

**本文探讨了[Moco](https://github.com/dreamhead/moco)，一种多功能、轻量级的Mock工具，可简化各种协议的Mock服务器**。

我们将探索Moco的功能和入门方法，并通过实际示例来说明其功能。

## 2. 什么是Moco？

**Moco是一个开源Mock库，允许我们存根和Mock服务**。它很轻量，使用和配置都非常简单。

**Moco支持多种协议**，例如HTTP、HTTPS和套接字，使其成为测试不同类型应用程序(从Web服务到实时通信应用程序)的绝佳选择。

## 3. 开始使用Moco

**Moco的主要优势之一是其易于安装和使用**，将其集成到新项目或现有项目中非常简单。

### 3.1 JSON配置和独立服务器

在深入研究代码之前，值得注意的是，**Mock允许我们使用JSON编写配置并独立运行它们**，这对于无需任何编码即可快速进行设置非常有益。

下面是我们可以在config.json文件中写入的JSON配置的一个简单示例：

```json
[
    {
        "request": {
            "uri": "/hello"
        },
        "response": {
            "text": "Hello, Moco!"
        }
    }
]
```

要使用此配置启动服务器，我们运行moco-runner：

```shell
java -jar moco-runner-1.5.0-standalone.jar http -p 12345 -c config.json
```

此命令使用提供的config.json在端口12345上运行HTTP服务器，Mock将在http://localhost:12345/hello上可用。

这些独立设置提供了很大的灵活性，并获得与代码配置相同级别的开发人员支持。

### 3.2 设置

要开始将Moco与Maven结合使用，我们将包含以下[依赖](https://mvnrepository.com/artifact/com.github.dreamhead/moco-runner)：

```xml
<dependency>
    <groupId>com.github.dreamhead</groupId>
    <artifactId>moco-runner</artifactId>
    <version>1.5.0</version>
    <scope>test</scope>
</dependency>
```

对于Gradle项目，我们需要使用这个：

```groovy
testImplementation 'com.github.dreamhead:moco-runner:1.5.0'
```

## 4. Java实例

**Mock拥有丰富的Java API，这让我们在定义Mock资源时拥有很大的自由度**。但现在，让我们通过一些简单示例来检查服务器初始化的工作原理。

### 4.1 服务器初始化

我们必须了解如何根据协议设置服务器，才能使用Java初始化Moco服务器。下面是简要概述：

```java
HttpServer server = Moco.httpServer(12345); // Initialize server on port 12345 
server.request(Moco.by(Moco.uri("/hello")))
    .response(Moco.text("Hello, Moco!")); // Set up a basic response

Runner runner = Runner.runner(server); 
runner.start(); // Start the server
```

在此示例中，我们首先在端口12345上初始化HTTP服务器。然后，我们将服务器配置为在请求/hello端点时响应“Hello, Moco!”。最后，我们使用Runner的start()方法启动服务器。

另外，完成后不要忘记停止服务器：

```java
runner.stop();
```

例如，在测试中，我们可以用@AfterEach注解将其放入方法中。对于HTTPS，我们需要先创建一个证书，该证书由HttpsCertificate对象表示，然后使用httpsServer()方法：

```java
HttpsCertificate certificate = certificate(pathResource("certificate.jks"), "keystorePassword", "certPassword");
HttpsServer server = httpsServer(12346, certificate);
```

如果我们想使用[套接字](https://www.baeldung.com/a-guide-to-java-sockets)连接，Mock提供了socketServer()：

```java
final SocketServer server = socketServer(12347);
```

我们还可以在创建服务器时使用之前提到的JSON配置：

```java
final HttpServer server = jsonHttpServer(12345, file("config.json"));
```

### 4.2 HTTP代码和响应

现在我们已经有了基本设置，让我们探索更复杂的例子。让我们**返回一个JSON响应**：

```java
server.request(by(uri("/api/user")))
    .response(header("Content-Type", "application/json"))
    .response(json("{\"id\": 1, \"name\": \"Ryland Grace\", \"email\": \"ryland.grace@example.com\"}"));
```

**如果我们将JSON存储在文件中，我们可以提供其路径**：

```java
server.request(by(uri("/api/user")))
    .response(header("Content-Type", "application/json"))
    .response(Moco.file("src/test/resources/user.json"));
```

Mock响应的默认HTTP代码是200。但是我们可以更改它，例如，重现错误响应：

```java
server.request(Moco.match(Moco.uri("/unknown"))).response(Moco.status(404), Moco.text("Not Found"));
```

另外，在上面的例子中，我们没有指定HTTP方法，这意味着默认使用GET。**要Mock POST请求，我们可以使用post()而不是request()**。事实上，在前面的例子中，我们可以明确使用get()。

我们来看一个POST Mock示例：

```java
server.post(by(uri("/resource"))).response("resource updated");
```

Mock针对GET、POST、PUT和DELETE具有单独的get()、post()、put()和delete()方法。

另外，我们可以指定Moco要检查的内容：

```java
server.request(json(file("user.json"))).response("resource updated");
```

### 4.3 更多微调

上面的例子只是Moco潜力的冰山一角。事实上，**Moco允许我们以[多种方式](https://github.com/dreamhead/moco/blob/master/moco-doc/apis.md)微调我们的服务器**。例如，我们可以为预期的请求配置查询参数、cookie、各种媒体类型、自定义条件等等。

在此示例中，我们**使用[JSONPath](https://www.baeldung.com/guide-to-jayway-jsonpath)匹配请求**：

```java
server.request(eq(jsonPath("$.item[*].price"), "0")).response("we have free item");
```

在配置响应时，我们可以模拟代理、重定向、文件附件、更改响应延迟、在循环中返回等。例如，为了模拟随时间改变状态的资源，我们可以配置一个Mock服务器，为每个连续请求返回不同的响应：

```java
// First request will get "Alice", second and third will get "Bob" and "Clyde". Forth request will return "Alice" again.
server.request(by(uri("/user"))).response(seq("Alice", "Bob", "Clyde"));
```

## 5. 在单元测试中使用Moco

**Mock与[JUnit](https://www.baeldung.com/junit)无缝集成**，我们可以通过在测试生命周期中嵌入Moco服务器来有效地Mock外部服务。以下是一个简单的示例：

```java
public class MocoUnitTest {
    private Runner runner;

    @BeforeEach
    public void setup() {
        HttpServer server = Moco.httpServer(12345);
        server.request(Moco.by(Moco.uri("/test"))).response("Test response");
        runner = Runner.runner(server);
        runner.start();
    }

    @AfterEach
    public void tearDown() {
        runner.stop();
    }

    // Our tests
}
```

在setup中，我们创建一个HTTP服务器，该服务器使用“Test response”来响应/test上的请求。我们在每次测试之前启动此服务器，之后停止它。

现在，让我们尝试测试我们的服务器：

```java
@Test
void givenMocoHttpServer_whenClientSendsRequest_thenShouldReturnExpectedResponse() throws Exception {
    HttpClient client = HttpClient.newHttpClient();
    HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create("http://localhost:12345/test"))
            .build();

    HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
    assertEquals("Test response", response.body());
}
```

在此示例中，我们使用Java的内置HttpClient向我们的Moco服务器发送HTTP请求。这种方法避免了额外的依赖，使其成为在我们的单元测试中测试HTTP交互的绝佳选择。

### 5.1 使用@Rule

Mock使用JUnit 4中的[TestRule](https://www.baeldung.com/junit-4-rules)来简化集成，MocoJunitRunner提供了几种将Moco服务器作为TestRule运行的方法，这些方法在测试之前启动服务器，并在测试之后停止服务器：

```java
public class MocoJunitHttpUnitTest {
    private static final HttpServer server = httpServer(12306);

    static {
        server.response(text("JUnit 4 Response"));
    }

    @Rule
    public MocoJunitRunner runner = MocoJunitRunner.httpRunner(server);

    @Test
    public void givenMocoServer_whenClientSendsRequest_thenShouldReturnExpectedResponse() throws IOException, InterruptedException {
        HttpClient client = HttpClient.newHttpClient();
        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("http://localhost:12306"))
                .build();

        HttpResponse<String> response = client
                .send(request, HttpResponse.BodyHandlers.ofString());

        assertEquals(response.body(), "JUnit 4 Response");
    }
}
```

### 5.2 JUnit 5集成

对于[JUnit 5](https://www.baeldung.com/junit-5)，Mock使用@ExtendWith注解来集成其服务器控件：

```java
@ExtendWith(MocoExtension.class)
@MocoHttpServer(filepath = "src/test/resources/foo.json", port = 12345)  // Load JSON config from file system
public class MocoHttpTest {
    @Test
    public void givenMocoServer_whenClientSendsRequest_thenShouldReturnExpectedResponse() throws IOException, InterruptedException {
        HttpClient client = HttpClient.newHttpClient();
        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("http://localhost:12345/hello"))
                .build();

        HttpResponse<String> response = client
                .send(request, HttpResponse.BodyHandlers.ofString());

        assertEquals("Hello, Moco!", response.body());
    }
}
```

@MocoHttpServer注解指定服务器配置，确保每次测试都正确设置和拆除它。

## 6. 使用Moco的良好做法

**虽然Moco提供了强大的配置功能，但我们应努力保持Mock设置简单易管理**。除非必要，否则应避免过于复杂的配置，因为它们可能难以维护和理解。此外，对于简单的场景，我们应考虑使用JSON文件，对于更动态或更复杂的需求，应考虑使用基于Java的配置。JSON文件可以整齐地存储在具有简洁名称的单独目录中。

**在并发运行时使用唯一端口隔离测试是一种很好的做法**，在这种情况下，我们应该确保每个Moco服务器实例使用唯一的端口。这可以防止端口冲突并确保测试不会相互干扰。我们可以使用配置管理库来动态处理端口分配。

**我们应该使用HTTPS进行安全通信**。如果我们的应用程序通过HTTPS进行通信，我们还应该将Moco服务器配置为使用HTTPS，这可确保我们的测试准确反映生产环境。

**值得考虑启用日志记录来跟踪传入的请求和响应**；它可能有助于快速诊断问题：

```java
HttpServer server = httpServer(12345, log()); // we can also specify file for log output as a String parameter in log()
```

## 7. 总结

**在本教程中，我们探索了Moco，这是一个功能强大且灵活的框架，可简化Java应用程序中的Mock**。无论我们需要Mock HTTP、HTTPS还是套接字服务器，Mock都提供了一个简单的API并与JUnit无缝集成。我们还了解了Moco为我们提供的强大工具箱；它允许我们针对许多不同的实际场景微调我们的Mock服务器。

通过将Moco纳入我们的测试策略，我们可以增强测试的可靠性和可维护性，从而使开发更加高效和稳健。