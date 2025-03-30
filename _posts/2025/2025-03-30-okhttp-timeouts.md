---
layout: post
title:  OkHttp超时快速指南
category: libraries
copyright: libraries
excerpt: OkHttp
---

## 1. 概述

在本快速教程中，我们将重点介绍可以为[OkHttp](https://square.github.io/okhttp/)客户端设置的不同类型的超时。

有关OkHttp库的更一般概述，请查看我们的[OkHttp入门指南](https://www.baeldung.com/guide-to-okhttp)。

## 2. 连接超时

**连接超时定义了我们的客户端应与目标主机建立连接的时间段**。

**默认情况下，对于OkHttpClient，此超时设置为10秒**。

但是，我们可以使用[OkHttpClient.Builder#connectTimeout](https://square.github.io/okhttp/3.x/okhttp/okhttp3/OkHttpClient.Builder.html#connectTimeout-long-java.util.concurrent.TimeUnit-)方法轻松更改其值，值为0表示根本没有超时。

现在让我们看看如何构建和使用具有自定义连接超时的OkHttpClient：

```java
@Test
public void whenConnectTimeoutExceeded_thenSocketTimeoutException() {
    OkHttpClient client = new OkHttpClient.Builder()
            .connectTimeout(10, TimeUnit.MILLISECONDS)
            .build();

    Request request = new Request.Builder()
            .url("http://203.0.113.1") // non routable address
            .build();

    Throwable thrown = catchThrowable(() -> client.newCall(request).execute());

    assertThat(thrown).isInstanceOf(SocketTimeoutException.class);
}
```

上面的例子表明，当连接尝试超过配置的超时时间时，客户端会抛出SocketTimeoutException。

## 3. 读取超时

从客户端和目标主机之间的连接成功建立那一刻起，就开始应用读取超时。

**它定义了等待服务器响应时两个数据包之间的最大不活动时间**。

可以使用[OkHttpClient.Builder#readTimeout](https://square.github.io/okhttp/3.x/okhttp/okhttp3/OkHttpClient.Builder.html#readTimeout-long-java.util.concurrent.TimeUnit-)更改默认超时时间10秒。与连接超时类似，0值表示无超时。

现在让我们看看如何在实践中配置自定义读取超时：

```java
@Test
public void whenReadTimeoutExceeded_thenSocketTimeoutException() {
    OkHttpClient client = new OkHttpClient.Builder()
            .readTimeout(10, TimeUnit.MILLISECONDS)
            .build();

    Request request = new Request.Builder()
            .url("https://httpbin.org/delay/2") // 2-second response time
            .build();

    Throwable thrown = catchThrowable(() -> client.newCall(request).execute());

    assertThat(thrown).isInstanceOf(SocketTimeoutException.class);
}
```

我们看到，服务器没有在定义的500毫秒超时时间内返回响应。因此，OkHttpClient抛出了SocketTimeoutException。

## 4. 写入超时

**写入超时定义了向服务器发送请求时两个数据包之间的最大不活动时间**。

类似地，对于连接和读取超时，**我们可以使用[OkHttpClient.Builder#writeTimeout](https://square.github.io/okhttp/3.x/okhttp/okhttp3/OkHttpClient.Builder.html#writeTimeout-long-java.util.concurrent.TimeUnit-)覆盖默认值10秒**。按照惯例，0值表示根本没有超时。

在以下示例中，我们设置了一个非常短的10毫秒写入超时，并将1MB的内容发布到服务器：

```java
@Test
public void whenWriteTimeoutExceeded_thenSocketTimeoutException() {
    OkHttpClient client = new OkHttpClient.Builder()
            .writeTimeout(10, TimeUnit.MILLISECONDS)
            .build();

    Request request = new Request.Builder()
            .url("https://httpbin.org/delay/2")
            .post(RequestBody.create(MediaType.parse("text/plain"), create1MBString()))
            .build();

    Throwable thrown = catchThrowable(() -> client.newCall(request).execute());

    assertThat(thrown).isInstanceOf(SocketTimeoutException.class);
}
```

如我们所见，由于负载过大，我们的客户端无法在定义的超时时间内向服务器发送请求主体。因此，OkHttpClient抛出了SocketTimeoutException。

## 5. 调用超时

调用超时与我们已经讨论过的连接、读取和写入超时略有不同。

**它定义了一个完整的HTTP调用的时间限制**，这包括解析DNS、连接、写入请求主体、服务器处理以及读取响应主体。

与其他超时不同，**它的默认值设置为0，这意味着没有超时**。但当然，我们可以使用[OkHttpClient.Builder#callTimeout](https://square.github.io/okhttp/3.x/okhttp/okhttp3/OkHttpClient.Builder.html#callTimeout-long-java.util.concurrent.TimeUnit-)方法配置自定义值。

我们来看一个实际使用的例子：

```java
@Test
public void whenCallTimeoutExceeded_thenInterruptedIOException() {
    OkHttpClient client = new OkHttpClient.Builder()
            .callTimeout(1, TimeUnit.SECONDS)
            .build();

    Request request = new Request.Builder()
            .url("https://httpbin.org/delay/2")
            .build();

    Throwable thrown = catchThrowable(() -> client.newCall(request).execute());

    assertThat(thrown).isInstanceOf(InterruptedIOException.class);
}
```

我们可以看到，调用超时了，OkHttpClient抛出了InterruptedIOException异常。

## 6. 每个请求的超时

**建议创建一个OkHttpClient实例并将其重用于我们应用程序中的所有HTTP调用**。

但有时我们知道某个请求比其他所有请求花费的时间都多，在这种情况下，我们只需要**为该特定调用延长给定的超时时间**。

在这种情况下，我们可以使用[OkHttpClient#newBuilder](https://square.github.io/okhttp/3.x/okhttp/okhttp3/OkHttpClient.html#newBuilder--)方法。这将构建一个共享相同设置的新客户端，然后，我们可以根据需要使用构建器方法来调整超时设置。

现在让我们看看在实践中如何做到这一点：

```java
@Test
public void whenPerRequestTimeoutExtended_thenResponseSuccess() throws IOException {
    OkHttpClient defaultClient = new OkHttpClient.Builder()
            .readTimeout(1, TimeUnit.SECONDS)
            .build();

    Request request = new Request.Builder()
            .url("https://httpbin.org/delay/2")
            .build();

    Throwable thrown = catchThrowable(() -> defaultClient.newCall(request).execute());

    assertThat(thrown).isInstanceOf(InterruptedIOException.class);

    OkHttpClient extendedTimeoutClient = defaultClient.newBuilder()
            .readTimeout(5, TimeUnit.SECONDS)
            .build();

    Response response = extendedTimeoutClient.newCall(request).execute();
    assertThat(response.code()).isEqualTo(200);
}
```

如我们所见，defaultClient由于超出读取超时而无法完成HTTP调用。

这就是我们创建extendedTimeoutClient、调整超时值并成功执行请求的原因。

## 7. 总结

在本文中，我们探讨了可以为OkHttpClient配置的不同超时。

我们还简要描述了在HTTP调用期间何时应用连接、读取和写入超时。

此外，我们还展示了仅针对单个请求更改某个超时值是多么容易。