---
layout: post
title:  Apache HttpClient超时
category: apache
copyright: apache
excerpt: Apache HttpClient
---

## 1. 概述

本教程将展示如何**使用Apache HttpClient 5配置超时**。

如果你想深入了解并学习使用HttpClient可以做的其他有趣的事情，请直接阅读[主要的HttpClient教程](https://www.baeldung.com/httpclient-guide)。

## 2. 使用HttpClient 5.x API配置超时

新版API引入了新的超时配置方式，我们将使用ConnectionConfig配置连接超时和套接字超时：

```java
ConnectionConfig connConfig = ConnectionConfig.custom()
    .setConnectTimeout(timeout, TimeUnit.MILLISECONDS)
    .setSocketTimeout(timeout, TimeUnit.MILLISECONDS)
    .build();
```

下一步是创建一个连接管理器并设置我们上面创建的ConnectionConfig：

```java
BasicHttpClientConnectionManager cm = new BasicHttpClientConnectionManager();
cm.setConnectionConfig(connConfig);
```

有关配置连接管理器的详细示例，请关注文章[Apache HttpClient连接管理](https://www.baeldung.com/httpclient-connection-management)。

## 3. 使用HttpClient 4.3配置超时

如果我们使用的是HttpClient 4.3，可以使用流式的构建器API**在高级别设置超时**：

```java
int timeout = 5;
RequestConfig config = RequestConfig.custom()
    .setConnectTimeout(timeout * 1000)
    .setConnectionRequestTimeout(timeout * 1000)
    .setSocketTimeout(timeout * 1000).build();
CloseableHttpClient client = HttpClientBuilder.create().setDefaultRequestConfig(config).build();
```

通过这种方式，我们可以以类型安全且可读的方式配置所有3个超时。

## 4. 超时属性解释

现在，让我们解释一下这些不同类型的超时的含义：

- **连接超时**(http.connection.timeout)：与远程主机建立连接的时间
- **套接字超时**(http.socket.timeout)：建立连接后等待数据的时间；两个数据包之间的最大不活跃时间
- **连接管理器超时**(http.connection-manager.timeout)：等待连接管理器/池连接的时间

前两个参数连接超时和套接字超时是最重要的，然而，在高负载情况下，设置获取连接的超时时间绝对很重要，因此第3个参数也不容忽视。

## 5. 使用HttpClient

配置完成后，我们现在可以使用客户端执行HTTP请求：

```java
final HttpGet request = new HttpGet("http://www.github.com");

try (CloseableHttpClient client = HttpClientBuilder.create()
    .setDefaultRequestConfig(requestConfig)
    .setConnectionManager(cm)
    .build();

    CloseableHttpResponse response = (CloseableHttpResponse) client
        .execute(request, new CustomHttpClientResponseHandler())) {

    final int statusCode = response.getCode();
    assertThat(statusCode, equalTo(HttpStatus.SC_OK));
}
```

使用之前定义的客户端，**与主机的连接将在5秒后超时**。此外，如果连接已建立，但未收到数据，则超时时间也会额外增加5秒。

请注意，连接超时将导致抛出org.apache.hc.client5.http.ConnectTimeoutException，而套接字超时将导致java.net.SocketTimeoutException。

## 6. 硬超时

虽然在建立HTTP连接并且未接收数据时设置超时非常有用，但有时我们需要**为整个请求设置硬超时**。

例如，下载一个可能很大的文件就属于这种情况。在这种情况下，连接可能成功建立，数据也可能持续传输，但我们仍然需要确保该操作不超过特定的时间阈值。

HttpClient没有任何配置允许我们为请求设置总体超时；但是，它确实**为请求提供了中止功能**，因此我们可以利用该机制来实现简单的超时机制：

```java
HttpGet getMethod = new HttpGet("http://localhost:8082/httpclient-simple/api/bars/1");
getMethod.setConfig(requestConfig);

int hardTimeout = 5000; // milliseconds
TimerTask task = new TimerTask() {
    @Override
    public void run() {
        getMethod.abort();
    }
};
new Timer(true).schedule(task, hardTimeout);

try (CloseableHttpClient client = HttpClientBuilder.create()
    .setDefaultRequestConfig(requestConfig)
    .setConnectionManager(cm)
    .build();

    CloseableHttpResponse response = (CloseableHttpResponse) client
        .execute(getMethod, new CustomHttpClientResponseHandler())) {

    final int statusCode = response.getCode();
    System.out.println("HTTP Status of response: " + statusCode);
    assertThat(statusCode, equalTo(HttpStatus.SC_OK));
}
```

我们利用java.util.Timer和java.util.TimerTask来设置一个简单的延迟任务，该任务在5秒硬超时后中止HTTP GET请求。

## 7. 超时和DNS循环-需要注意的事项

一些较大的域名通常会使用DNS循环配置-**本质上是将同一个域名映射到多个IP地址**，这给此类域名的超时带来了新的挑战，因为HttpClient尝试连接到超时的域名的方式如下：

- HttpClient获取到该域的IP路由列表
- 它尝试第一个- 超时(与我们配置的超时时间相同)
- 它尝试第二个，但仍然超时
- 等等...

因此，正如你所见，**整个操作不会像我们预期的那样超时**。相反，它会在所有可能的路由都超时时才会超时。而且，这对于客户端来说是完全透明的(除非你将日志配置为DEBUG级别)。

这是一个你可以运行并重现此问题的简单示例：

```java
ConnectionConfig connConfig = ConnectionConfig.custom()
    .setConnectTimeout(timeout, TimeUnit.MILLISECONDS)
    .setSocketTimeout(timeout, TimeUnit.MILLISECONDS)
    .build();

RequestConfig requestConfig = RequestConfig.custom()
    .setConnectionRequestTimeout(Timeout.ofMilliseconds(3000L))
    .build();

BasicHttpClientConnectionManager cm = new BasicHttpClientConnectionManager();
cm.setConnectionConfig(connConfig);

CloseableHttpClient client = HttpClientBuilder.create()
    .setDefaultRequestConfig(requestConfig)
    .setConnectionManager(cm)
    .build();

HttpGet request = new HttpGet("http://www.google.com:81");

response = client.execute(request, new CustomHttpClientResponseHandler());
```

可以注意到具有DEBUG日志级别的重试逻辑：

```text
DEBUG o.a.h.i.c.HttpClientConnectionOperator - Connecting to www.google.com/173.194.34.212:81
DEBUG o.a.h.i.c.HttpClientConnectionOperator - 
 Connect to www.google.com/173.194.34.212:81 timed out. Connection will be retried using another IP address

DEBUG o.a.h.i.c.HttpClientConnectionOperator - Connecting to www.google.com/173.194.34.208:81
DEBUG o.a.h.i.c.HttpClientConnectionOperator - 
 Connect to www.google.com/173.194.34.208:81 timed out. Connection will be retried using another IP address

DEBUG o.a.h.i.c.HttpClientConnectionOperator - Connecting to www.google.com/173.194.34.209:81
DEBUG o.a.h.i.c.HttpClientConnectionOperator - 
 Connect to www.google.com/173.194.34.209:81 timed out. Connection will be retried using another IP address
//...
```

## 8. 总结

本教程讨论了如何配置HttpClient的各种超时类型，并演示了一种用于持续HTTP连接硬超时的简单机制。