---
layout: post
title:  OkHTTP事件指南
category: libraries
copyright: libraries
excerpt: OkHTTP
---

## 1. 概述

通常，当我们在Web应用程序中处理HTTP调用时，我们需要一种方法来捕获有关请求和响应的某种指标。通常，这是为了监控应用程序发出的HTTP调用的大小和频率。

[OkHttp](https://square.github.io/okhttp/)是适用于Android和Java应用程序的高效HTTP和HTTP/2客户端。在之前的教程中，我们了解了如何使用OkHttp的[基础知识](https://www.baeldung.com/guide-to-okhttp)。

在本教程中，我们将学习如何使用事件捕获这些类型的指标。

## 2. 事件

顾名思义，事件为我们提供了一种强大的机制来记录与整个HTTP调用生命周期相关的应用程序指标。

**为了订阅我们感兴趣的所有事件，我们需要做的是定义一个EventListener并重写我们想要捕获的事件的方法**。

例如，如果我们只想监控失败和成功的调用，这将特别有用。在这种情况下，我们只需在事件监听器类中重写与这些事件相对应的特定方法。我们稍后会更详细地介绍这一点。

在我们的应用程序中使用事件至少有几个优点：

- 我们可以使用事件来监视应用程序进行的HTTP调用的大小和频率
- 这可以帮助我们快速确定应用程序中可能存在瓶颈的地方

最后，我们还可以使用事件来确定我们的网络是否存在潜在问题。

## 3. 依赖

当然，我们需要在pom.xml中添加标准[okhttp依赖](https://mvnrepository.com/artifact/com.squareup.okhttp3/okhttp)：

```xml
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>5.0.0-alpha.12</version>
</dependency>
```

我们还需要另一个专门用于测试的依赖，让我们添加OkHttp [mockwebserver](https://mvnrepository.com/artifact/com.squareup.okhttp3/mockwebserver)：

```xml
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>mockwebserver</artifactId>
    <version>5.0.0-alpha.12</version>
    <scope>test</scope>
</dependency>
```

现在我们已经配置了所有必要的依赖，我们可以继续编写我们的第一个事件监听器。

## 4. 事件方法和顺序

但在开始定义自己的事件监听器之前，**我们将退一步并简要了解一下我们可以使用哪些事件方法以及我们可以预期事件到达的顺序**，这将有助于我们稍后深入研究一些真实的例子。

假设我们正在处理一个成功的HTTP调用，没有重定向或重试，那么我们可以预期这种典型的方法调用流程。

### 4.1 callStart()

此方法是我们的入口点，我们将在排队调用或客户端执行它时立即调用它。

### 4.2 proxySelectStart()和proxySelectEnd()

第一个方法在代理选择之前调用，同样在代理选择之后调用，包括按尝试顺序排列的代理列表。当然，如果没有配置代理，此列表可以为空。

### 4.3 dnsStart()和dnsEnd()

这些方法在DNS查找之前和DNS解析之后立即调用。

### 4.4 connectStart()和connectEnd()

这些方法在建立和关闭套接字连接之前被调用。

### 4.5 secureConnectStart()和secureConnectEnd()

如果我们的调用使用HTTPS，那么在connectStart和connectEnd之间我们将会有这些安全连接变化。

### 4.6 connectionAcquired()和connectionReleased()

获取或释放连接后调用。

### 4.7 requestHeadersStart()和requestHeadersEnd()

这些方法将在发送请求标头之前和之后立即调用。

### 4.8 requestBodyStart()和requestBodyEnd()

顾名思义，在发送请求主体之前调用。当然，这仅适用于包含主体的请求。

### 4.9 responseHeadersStart()和responseHeadersEnd()

当服务器首次返回响应头时以及在接收到响应头后立即调用这些方法。

### 4.10 responseBodyStart()和responseBodyEnd()

同样，在服务器第一次返回响应主体时以及在收到主体后立即调用。

**除了这些方法之外，我们还有另外三种方法可以用来捕获故障**：

### 4.11 callFailed()、responseFailed()和requestFailed()

如果我们的调用永久失败，则请求出现写入失败，或者响应出现读取失败。

## 5. 定义一个简单的事件监听器

让我们首先定义自己的事件监听器。为了使事情变得简单，**我们的事件监听器将在调用开始和结束时记录一些请求和响应标头信息**：

```java
public class SimpleLogEventsListener extends EventListener {

    private static final Logger LOGGER = LoggerFactory.getLogger(SimpleLogEventsListener.class);

    @Override
    public void callStart(Call call) {
        LOGGER.info("callStart at {}", LocalDateTime.now());
    }

    @Override
    public void requestHeadersEnd(Call call, Request request) {
        LOGGER.info("requestHeadersEnd at {} with headers {}", LocalDateTime.now(), request.headers());
    }

    @Override
    public void responseHeadersEnd(Call call, Response response) {
        LOGGER.info("responseHeadersEnd at {} with headers {}", LocalDateTime.now(), response.headers());
    }

    @Override
    public void callEnd(Call call) {
        LOGGER.info("callEnd at {}", LocalDateTime.now());
    }
}
```

如我们所见，**要创建监听器，我们需要做的就是从EventListener类扩展**，然后我们可以继续覆盖我们关心的事件的方法。

在我们的简单监听器中，我们记录调用开始和结束的时间以及到达的请求和响应标头。

### 5.1 连接起来

要真正使用这个监听器，我们需要做的就是在构建我们的OkHttpClient实例时调用eventListener方法，它就可以工作了：

```java
OkHttpClient client = new OkHttpClient.Builder() 
    .eventListener(new SimpleLogEventsListener())
    .build();
```

在下一节中，我们将了解如何测试新的监听器。

### 5.2 测试事件监听器

现在，我们已经定义了第一个事件监听器；让我们继续编写第一个集成测试：

```java
@Rule
public MockWebServer server = new MockWebServer();

@Test
public void givenSimpleEventLogger_whenRequestSent_thenCallsLogged() throws IOException {
    server.enqueue(new MockResponse().setBody("Hello Tuyucheng Readers!"));

    OkHttpClient client = new OkHttpClient.Builder()
            .eventListener(new SimpleLogEventsListener())
            .build();

    Request request = new Request.Builder()
            .url(server.url("/"))
            .build();

    try (Response response = client.newCall(request).execute()) {
        assertEquals("Response code should be: ", 200, response.code());
        assertEquals("Body should be: ", "Hello Tuyucheng Readers!", response.body().string());
    }
}
```

首先，我们使用OkHttpMockWebServer [JUnit Rule](https://www.baeldung.com/junit-4-rules)。

**这是一个轻量级、可编写脚本的Web服务器，用于测试HTTP客户端，我们将使用它来测试我们的事件监听器**。通过使用此Rule，我们将为每个集成测试创建一个干净的服务器实例。

考虑到这一点，现在让我们来看看测试的关键部分：

- 首先，我们设置一个Mock响应，其中包含正文中的简单消息
- 然后，我们构建OkHttpClient并配置SimpleLogEventsListener
- **最后，我们发送请求并使用断言检查收到的响应码和正文**

### 5.3 运行测试

当我们运行测试时，我们将看到记录的事件：

```text
callStart at 2021-05-04T17:51:33.024
...
requestHeadersEnd at 2021-05-04T17:51:33.046 with headers User-Agent: A Tuyucheng Reader
Host: localhost:51748
Connection: Keep-Alive
Accept-Encoding: gzip
...
responseHeadersEnd at 2021-05-04T17:51:33.053 with headers Content-Length: 23
callEnd at 2021-05-04T17:51:33.055
```

## 6. 综合起来

现在让我们想象一下，我们要基于简单的日志记录示例并记录调用链中每个步骤的经过时间：

```java
public class EventTimer extends EventListener {

    private long start;

    private void logTimedEvent(String name) {
        long now = System.nanoTime();
        if (name.equals("callStart")) {
            start = now;
        }
        long elapsedNanos = now - start;
        System.out.printf("%.3f %s%n", elapsedNanos / 1000000000d, name);
    }

    @Override
    public void callStart(Call call) {
        logTimedEvent("callStart");
    }

    // More event listener methods
}
```

这与我们的第一个示例非常相似，但这次我们捕获了每个事件从调用开始到经过的时间。**通常，这对于检测网络延迟可能非常有用**。

让我们看看如果在像我们自己的https://www.baeldung.com/这样的真实网站上运行这个程序会怎么样：

```text
0.000 callStart
0.012 proxySelectStart
0.012 proxySelectEnd
0.012 dnsStart
0.175 dnsEnd
0.183 connectStart
0.248 secureConnectStart
0.608 secureConnectEnd
0.608 connectEnd
0.609 connectionAcquired
0.612 requestHeadersStart
0.613 requestHeadersEnd
0.706 responseHeadersStart
0.707 responseHeadersEnd
0.765 responseBodyStart
0.765 responseBodyEnd
0.765 connectionReleased
0.765 callEnd

```

由于此调用通过HTTPS进行，我们还将看到secureConnectStart和secureConnectStart事件。

## 7. 监控失败调用

到目前为止，我们一直专注于成功的HTTP请求，但我们也可以捕获失败的事件：

```java
@Test (expected = SocketTimeoutException.class)
public void givenConnectionError_whenRequestSent_thenFailedCallsLogged() throws IOException {
    OkHttpClient client = new OkHttpClient.Builder()
            .eventListener(new EventTimer())
            .build();

    Request request = new Request.Builder()
            .url(server.url("/"))
            .build();

    client.newCall(request).execute();
}
```

在这个例子中，我们故意避免设置模拟Web服务器，这意味着，我们当然会看到以SocketTimeoutException形式出现的灾难性故障。

现在让我们看一下运行测试时的输出：

```text
0.000 callStart
...
10.008 responseFailed
10.009 connectionReleased
10.009 callFailed
```

**正如预期的那样，我们将看到调用开始，然后10秒后，发生连接超时，因此，我们看到记录的responseFailed和callFailed事件**。

## 8. 关于并发的简要说明

到目前为止，我们假设没有多个调用[同时](https://www.baeldung.com/cs/aba-concurrency)执行。**如果我们想要适应这种情况，那么我们需要在配置OkHttpClient时使用eventListenerFactory方法**。

我们可以使用工厂为每个HTTP调用创建一个新的EventListener实例，使用这种方法时，可以在监听器中保留特定于调用的状态。

## 9. 总结

在本文中，我们了解了如何使用OkHttp捕获事件。首先，我们首先解释什么是事件，并了解我们可以获得哪些类型的事件以及它们在处理HTTP调用时到达的顺序。

然后我们研究了如何定义一个简单的事件记录器来捕获HTTP调用的部分以及如何编写集成测试。