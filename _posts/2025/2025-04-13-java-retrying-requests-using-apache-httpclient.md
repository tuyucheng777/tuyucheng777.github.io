---
layout: post
title:  使用Apache HttpClient重试请求
category: apache
copyright: apache
excerpt: Apache HttpClient
---

## 1. 简介

在本教程中，我们将研究如何在使用Apache HttpClient时重试HTTP请求，我们还将探讨该库重试的默认行为及其配置方法。

## 2. 默认重试策略

在进入默认行为之前，我们将创建一个带有HttpClient实例和请求计数器的测试类：

```java
public class ApacheHttpClientRetryUnitTest {

    private Integer requestCounter;
    private CloseableHttpClient httpClient;

    @BeforeEach
    void setUp() {
        requestCounter = 0;
    }

    @AfterEach
    void tearDown() throws IOException {
        if (httpClient != null) {
            httpClient.close();
        }
    }
}
```

让我们从默认行为开始-Apache HttpClient最多会重试3次所有以[IOException](https://www.baeldung.com/java-exceptions)结束的幂等请求，因此我们总共会收到4个请求。为了演示，我们在这里创建一个每次请求都会抛出IOException的HttpClient：

```java
private void createFailingHttpClient() {
    this.httpClient = HttpClientBuilder
            .create()
            .addInterceptorFirst((HttpRequestInterceptor) (request, context) -> requestCounter++)
            .addInterceptorLast((HttpResponseInterceptor) (response, context) -> { throw new IOException(); })
            .build();
}

@Test
public void givenDefaultConfiguration_whenReceviedIOException_thenRetriesPerformed() {
    createFailingHttpClient();
    assertThrows(IOException.class, () -> httpClient.execute(new HttpGet("https://httpstat.us/200")));
    assertThat(requestCounter).isEqualTo(4);
}
```

HttpClient认为某些IOException子类不可重试，具体来说，它们是：

- InterruptedIOException
- ConnectException
- UnknownHostException
- SSLException
- NoRouteToHostException

例如，如果我们无法解析目标主机的DNS名称，则不会重试该请求：

```java
public void createDefaultApacheHttpClient() {
    this.httpClient = HttpClientBuilder
            .create()
            .addInterceptorFirst((HttpRequestInterceptor) (httpRequest, httpContext) -> requestCounter++).build();
}

@Test
public void givenDefaultConfiguration_whenDomainNameNotResolved_thenNoRetryApplied() {
    createDefaultApacheHttpClient();
    HttpGet request = new HttpGet(URI.create("http://domain.that.does.not.exist:80/api/v1"));

    assertThrows(UnknownHostException.class, () -> httpClient.execute(request));
    assertThat(requestCounter).isEqualTo(1);
}
```

我们可以注意到，这些异常通常表示网络或TLS问题。因此，它们与HTTP请求处理失败无关。这意味着，如果服务器以5xx或4xx响应我们的请求，则不会应用重试逻辑：

```java
@Test
public void givenDefaultConfiguration_whenGotInternalServerError_thenNoRetryLogicApplied() throws IOException {
    createDefaultApacheHttpClient();
    HttpGet request = new HttpGet(URI.create("https://httpstat.us/500"));

    CloseableHttpResponse response = assertDoesNotThrow(() -> httpClient.execute(request));
    assertThat(response.getStatusLine().getStatusCode()).isEqualTo(500);
    assertThat(requestCounter).isEqualTo(1);
    response.close();
}
```

但大多数情况下，这并不是我们想要的，我们通常希望至少在5xx状态码时重试。因此，我们需要覆盖默认行为。

## 3. 幂等性

在自定义重试之前，我们需要稍微解释一下请求的幂等性。这一点很重要，因为Apache HTTP客户端认为所有HttpEntityEnclosingRequest实现都是非幂等的，此接口的常见实现是HttpPost、HttpPut和HttpPatch类。因此，默认情况下，我们的PATCH和PUT请求不会重试：

```java
@Test
public void givenDefaultConfiguration_whenHttpPatchRequest_thenRetryIsNotApplied() {
    createFailingHttpClient();
    HttpPatch request = new HttpPatch(URI.create("https://httpstat.us/500"));

    assertThrows(IOException.class, () -> httpClient.execute(request));
    assertThat(requestCounter).isEqualTo(1);
}

@Test
public void givenDefaultConfiguration_whenHttpPutRequest_thenRetryIsNotApplied() {
    createFailingHttpClient();
    HttpPut request = new HttpPut(URI.create("https://httpstat.us/500"));

    assertThrows(IOException.class, () -> httpClient.execute(request));
    assertThat(requestCounter).isEqualTo(1);
}
```

我们可以看到，即使我们收到了IOException异常，也没有进行任何重试。

## 4. 自定义RetryHandler

我们提到的默认行为是可以被覆盖的，首先，我们可以设置RetryHandler。为此，可以选择使用DefaultHttpRequestRetryHandler，这是一个方便的开箱即用的RetryHandler实现，顺便说一下，该库默认使用它。这个默认实现也实现了我们讨论过的默认行为。

通过使用DefaultHttpRequestRetryHandler，我们可以设置所需的重试次数以及HttpClient是否应该重试幂等请求：

```java
private void createHttpClientWithRetryHandler() {
    this.httpClient = HttpClientBuilder
            .create()
            .addInterceptorFirst((HttpRequestInterceptor) (httpRequest, httpContext) -> requestCounter++)
            .addInterceptorLast((HttpResponseInterceptor) (httpRequest, httpContext) -> { throw new IOException(); })
            .setRetryHandler(new DefaultHttpRequestRetryHandler(6, true))
            .build();
}

@Test
public void givenConfiguredRetryHandler_whenHttpPostRequest_thenRetriesPerformed() {
    createHttpClientWithRetryHandler();

    HttpPost request = new HttpPost(URI.create("https://httpstat.us/500"));

    assertThrows(IOException.class, () -> httpClient.execute(request));
    assertThat(requestCounter).isEqualTo(7);
}
```

如我们所见，我们将DefaultHttpRequestRetryHandler配置为进行6次重试，参见第一个构造函数参数。此外，我们还启用了幂等请求的重试，参见第二个构造函数布尔参数。因此，HttpCleint执行了7次POST请求-1次原始请求和6次重试。

此外，如果这种程度的定制还不够，我们可以创建自己的RetryHandler：

```java
private void createHttpClientWithCustomRetryHandler() {
    this.httpClient = HttpClientBuilder
            .create()
            .addInterceptorFirst((HttpRequestInterceptor) (httpRequest, httpContext) -> requestCounter++)
            .addInterceptorLast((HttpResponseInterceptor) (httpRequest, httpContext) -> { throw new IOException(); })
            .setRetryHandler((exception, executionCount, context) -> {
                if (executionCount <= 4 && Objects.equals("GET", ((HttpClientContext) context).getRequest().getRequestLine().getMethod())) {
                    return true;
                } else {
                    return false;
                }
            }).build();
}

@Test
public void givenCustomRetryHandler_whenUnknownHostException_thenRetryAnyway() {
    createHttpClientWithCustomRetryHandler();

    HttpGet request = new HttpGet(URI.create("https://domain.that.does.not.exist/200"));

    assertThrows(IOException.class, () -> httpClient.execute(request));
    assertThat(requestCounter).isEqualTo(5);
}
```

这里我们基本上说的是-无论发生什么异常，都要重试所有GET请求4次。所以在上面的例子中，我们重试了UnknownHostException。

## 5. 禁用重试逻辑

最后，有些情况下我们可能想要禁用重试，我们可以提供一个始终返回false的RetryHandler，或者使用disableAutomaticRetries()：

```java
private void createHttpClientWithRetriesDisabled() {
    this.httpClient = HttpClientBuilder
            .create()
            .addInterceptorFirst((HttpRequestInterceptor) (httpRequest, httpContext) -> requestCounter++)
            .addInterceptorLast((HttpResponseInterceptor) (httpRequest, httpContext) -> { throw new IOException(); })
            .disableAutomaticRetries()
            .build();
}

@Test
public void givenDisabledRetries_whenExecutedHttpRequestEndUpWithIOException_thenRetryIsNotApplied() {
    createHttpClientWithRetriesDisabled();
    HttpGet request = new HttpGet(URI.create("https://httpstat.us/200"));

    assertThrows(IOException.class, () -> httpClient.execute(request));
    assertThat(requestCounter).isEqualTo(1);
}
```

通过在HttpClientBuilder上调用disableAutomaticRetries()，我们禁用了HttpClient中的所有重试，这意味着不会有任何请求被丢弃。

## 6. 总结

在本教程中，我们讨论了Apache HttpClient中的默认重试行为，开箱即用的RetryHandler会在发生异常时重试幂等请求3次，但是，我们可以配置重试次数和非幂等请求的重试策略。此外，我们还可以提供自己的RetryHandler实现，以便进一步定制。最后，我们可以在HttpClient构造过程中调用HttpClientBuilder的方法来禁用重试。