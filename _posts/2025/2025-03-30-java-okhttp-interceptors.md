---
layout: post
title:  在OkHTTP中添加拦截器
category: libraries
copyright: libraries
excerpt: OkHTTP
---

## 1. 概述

通常，当我们在Web应用程序中管理HTTP请求和响应周期时，我们需要一种方法来利用此链。通常，这是在我们完成请求之前或在我们的Servlet代码完成后添加一些自定义行为。

[OkHttp](https://square.github.io/okhttp/)是适用于Android和Java应用程序的高效HTTP和HTTP/2客户端，在之前的教程中，我们了解了如何使用OkHttp的[基础知识](https://www.baeldung.com/guide-to-okhttp)。

在本教程中，**我们将学习如何拦截HTTP请求和响应对象**。

## 2. 拦截器

顾名思义，拦截器是可插拔的Java组件，我们可以使用它在请求发送到我们的应用程序代码之前拦截和处理请求。

同样，它们为我们提供了一种强大的机制，允许我们在容器将响应发送回客户端之前处理服务器响应。

**当我们想要改变HTTP请求中的某些内容时，这特别有用，例如添加新的控制标头、更改请求主体或只是生成日志来帮助我们调试**。

使用拦截器的另一个好处是，它们允许我们将常用功能封装在一个地方。假设我们想将一些逻辑全局应用于所有请求和响应对象，例如错误处理。

将这种逻辑放入拦截器至少有几个优点：

- 我们只需要在一个地方维护这段代码，而不是所有的端点
- 每个请求都以相同的方式处理错误

最后，我们还可以监视、重写和重试拦截器的调用。

## 3. 常见用法

当使用拦截器时，其他一些常见任务可能是一个明显的选择，包括：

- 记录请求参数和其他有用信息
- 在请求中添加身份验证和授权标头
- 格式化请求和响应主体
- **压缩发送给客户端的响应数据**
- 通过添加一些cookies或额外的标头信息来更改我们的响应标头

我们将在后续章节中看到一些实际的例子。

## 4. 依赖

当然，我们需要在pom.xml中添加标准[okhttp依赖](https://mvnrepository.com/artifact/com.squareup.okhttp3/okhttp)：

```xml
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>5.0.0-alpha.12</version>
</dependency>
```

我们还需要另一个专门用于测试的依赖。让我们添加OkHttp [mockwebserver](https://mvnrepository.com/artifact/com.squareup.okhttp3/mockwebserver)：

```xml
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>mockwebserver</artifactId>
    <version>5.0.0-alpha.12</version>
    <scope>test</scope>
</dependency>
```

现在我们已经配置了所有必要的依赖，可以继续编写我们的第一个拦截器。

## 5. 定义一个简单的日志拦截器

让我们首先定义自己的拦截器，为了简单起见，我们的拦截器将记录请求标头和请求URL：

```java
public class SimpleLoggingInterceptor implements Interceptor {

    private static final Logger LOGGER = LoggerFactory.getLogger(SimpleLoggingInterceptor.class);

    @Override
    public Response intercept(Chain chain) throws IOException {
        Request request = chain.request();

        LOGGER.info("Intercepted headers: {} from URL: {}", request.headers(), request.url());

        return chain.proceed(request);
    }
}
```

我们可以看到，**要创建拦截器，我们只需要继承Interceptor接口，该接口有一个强制方法intercept(Chain chain)，然后我们可以继续用自己的实现覆盖此方法**。

首先，我们通过调用chain.request()获取传入的请求，然后打印出标头和请求URL。

**值得注意的是，每个拦截器实现的关键部分是对chain.proceed(request)的调用**。

这个看起来很简单的方法就是我们发出信号，表示我们想要访问我们的应用程序代码，并产生响应来满足请求。

### 5.1 连接起来

要真正使用这个拦截器，我们需要做的就是在构建我们的OkHttpClient实例时调用addInterceptor方法，它就可以工作了：

```java
OkHttpClient client = new OkHttpClient.Builder() 
    .addInterceptor(new SimpleLoggingInterceptor())
    .build();
```

我们可以继续为所需的任意数量的拦截器调用addInterceptor方法，只需记住它们将按照添加的顺序被调用。

### 5.2 测试拦截器

现在，我们已经定义了第一个拦截器；让我们继续编写第一个集成测试：

```java
@Rule
public MockWebServer server = new MockWebServer();

@Test
public void givenSimpleLoggingInterceptor_whenRequestSent_thenHeadersLogged() throws IOException {
    server.enqueue(new MockResponse().setBody("Hello Tuyucheng Readers!"));

    OkHttpClient client = new OkHttpClient.Builder()
            .addInterceptor(new SimpleLoggingInterceptor())
            .build();

    Request request = new Request.Builder()
            .url(server.url("/greeting"))
            .header("User-Agent", "A Tuyucheng Reader")
            .build();

    try (Response response = client.newCall(request).execute()) {
        assertEquals("Response code should be: ", 200, response.code());
        assertEquals("Body should be: ", "Hello Tuyucheng Readers!", response.body().string());
    }
}
```

首先，我们使用OkHttpMockWebServer [JUnit Rule](https://www.baeldung.com/junit-4-rules)。

**这是一个轻量级、可编写脚本的Web服务器，用于测试HTTP客户端，我们将使用它来测试我们的拦截器**。通过使用此Rule，我们将为每个集成测试创建一个干净的服务器实例。

考虑到这一点，现在让我们来看看测试的关键部分：

- 首先，我们设置一个Mock响应，其中包含正文中的简单消息
- 然后，我们构建OkHttpClient并配置SimpleLoggingInterceptor
- 接下来，我们设置要发送的请求，其中包含一个User-Agent标头
- **最后一步是发送请求并验证收到的响应码和正文是否符合预期**

### 5.3 运行测试

最后，当我们运行测试时，我们将看到记录的HTTPUser-Agent标头：

```text
16:07:02.644 [main] INFO  c.t.t.o.i.SimpleLoggingInterceptor - Intercepted headers: User-Agent: A Tuyucheng Reader
 from URL: http://localhost:54769/greeting
```

### 5.4 使用内置的HttpLoggingInterceptor

虽然我们的日志拦截器很好地演示了如何定义拦截器，但值得一提的是，OkHttp有一个内置的记录器，我们可以利用它。

为了使用这个记录器，我们需要一个额外的[Maven依赖](https://mvnrepository.com/artifact/com.squareup.okhttp3/logging-interceptor)：

```xml
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>logging-interceptor</artifactId>
    <version>5.0.0-alpha.12</version>
</dependency>
```

然后我们可以继续实例化我们的记录器并定义我们感兴趣的日志记录级别：

```java
HttpLoggingInterceptor logger = new HttpLoggingInterceptor();
logger.setLevel(HttpLoggingInterceptor.Level.HEADERS);
```

在这个例子中，我们只对读取标头感兴趣。

## 6. 添加自定义响应标头

我们了解了创建拦截器的基本知识，现在让我们看看另一个典型的用例，即修改其中一个HTTP响应标头。

**如果我们想要添加自己的专有应用程序HTTP标头或重写从我们的服务器返回的其中一个标头，这将很有用**：

```java
public class CacheControlResponeInterceptor implements Interceptor {

    @Override
    public Response intercept(Chain chain) throws IOException {
        Response response = chain.proceed(chain.request());
        return response.newBuilder()
                .header("Cache-Control", "no-store")
                .build();
    }
}
```

和之前一样，我们调用chain.proceed方法，但这次事先不使用请求对象。当响应返回时，我们使用它来创建一个新的响应，并将Cache-Control标头设置为no-store。

实际上，我们不太可能每次都告诉浏览器从服务器提取数据，但我们可以使用这种方法在响应中设置任何标头。

## 7. 使用拦截器处理错误

如前所述，我们还可以使用拦截器来封装一些我们想要全局应用于所有请求和响应对象的逻辑，例如错误处理。

假设当响应不是HTTP 200时，我们想要返回带有状态和消息的轻量级JSON响应。

考虑到这一点，我们首先定义一个简单的Bean来保存错误消息和状态码：

```java
public class ErrorMessage {

    private final int status;
    private final String detail;

    public ErrorMessage(int status, String detail) {
        this.status = status;
        this.detail = detail;
    }

    // Getters and setters
}
```

接下来，我们将创建拦截器：

```java
public class ErrorResponseInterceptor implements Interceptor {

    public static final MediaType APPLICATION_JSON = MediaType.get("application/json; charset=utf-8");

    @Override
    public Response intercept(Chain chain) throws IOException {
        Response response = chain.proceed(chain.request());

        if (!response.isSuccessful()) {
            Gson gson = new Gson();
            String body = gson.toJson(
                    new ErrorMessage(response.code(), "The response from the server was not OK"));
            ResponseBody responseBody = ResponseBody.create(body, APPLICATION_JSON);

            ResponseBody originalBody = response.body();
            if (originalBody != null) {
                originalBody.close();
            }

            return response.newBuilder().body(responseBody).build();
        }
        return response;
    }
}
```

很简单，我们的拦截器检查响应是否成功，如果不成功，则创建一个包含响应码和简单消息的JSON响应。请注意，在这种情况下，我们必须记住关闭原始响应的主体，以释放与其相关的任何资源。

```json
{
    "status": 500,
    "detail": "The response from the server was not OK"
}
```

## 8. 网络拦截器

到目前为止，我们介绍的拦截器都是OkHttp所指的应用程序拦截器。但是，OkHttp还支持另一种类型的拦截器，即网络拦截器。

我们可以按照前面解释的完全相同的方式定义网络拦截器，**但是，我们需要在创建HTTP客户端实例时调用addNetworkInterceptor方法**：

```java
OkHttpClient client = new OkHttpClient.Builder()
    .addNetworkInterceptor(new SimpleLoggingInterceptor())
    .build();
```

应用程序和网络拦截器之间的一些重要区别包括：

- **应用程序拦截器始终被调用一次，即使HTTP响应来自缓存**
- 网络拦截器挂接到网络层，是放置重试逻辑的理想位置
- 同样，当我们的逻辑不依赖于响应的实际内容时，我们应该考虑使用网络拦截器
- 使用网络拦截器可以让我们访问承载请求的连接，包括用于连接到Web服务器的IP地址和TLS配置等信息
- 应用程序拦截器无需担心重定向和重试等中间响应
- 相反，如果我们设置了重定向，网络拦截器可能会被多次调用

我们可以看到，两种选择都有各自的优点。因此，选择哪种选择实际上取决于我们自己的具体用例。

不过，大多数情况下，应用程序拦截器就能很好地完成这项工作。

## 9. 总结

在本文中，我们了解了如何使用OkHttp创建拦截器。首先，我们解释什么是拦截器以及如何定义一个简单的日志拦截器来检查我们的HTTP请求标头。

然后，我们了解了如何在响应对象中设置标头和不同的主体。最后，我们快速了解了应用程序和网络拦截器之间的一些差异。