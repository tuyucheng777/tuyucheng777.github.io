---
layout: post
title:  使用Java中的async-http-client实现异步HTTP
category: libraries
copyright: libraries
excerpt: AsyncHttpClient
---

## 1. 概述

[AsyncHttpClien](https://github.com/AsyncHttpClient/async-http-client)t(AHC)是一个建立在[Netty](https://www.baeldung.com/netty)之上的库，目的是轻松执行HTTP请求并异步处理响应。

在本文中，我们将介绍如何配置和使用HTTP客户端，如何使用AHC执行请求并处理响应。

## 2. 设置

最新版本的库可以在[Maven仓库](https://mvnrepository.com/artifact/org.asynchttpclient/async-http-client)中找到，我们应该小心使用groupId为org.asynchttpclient的依赖，而不是groupId为com.ning的依赖：

```xml
<dependency>
    <groupId>org.asynchttpclient</groupId>
    <artifactId>async-http-client</artifactId>
    <version>2.2.0</version>
</dependency>
```

## 3. HTTP客户端配置

获取HTTP客户端最直接的方法是使用Dsl类，静态asyncHttpClient()方法返回一个AsyncHttpClient对象：

```java
AsyncHttpClient client = Dsl.asyncHttpClient();
```

如果我们需要自定义HTTP客户端配置，我们可以使用构建器DefaultAsyncHttpClientConfig.Builder来构建AsyncHttpClient对象：

```java
DefaultAsyncHttpClientConfig.Builder clientBuilder = Dsl.config()
```

这提供了配置超时、代理服务器、HTTP证书等的可能性：

```java
DefaultAsyncHttpClientConfig.Builder clientBuilder = Dsl.config()
    .setConnectTimeout(500)
    .setProxyServer(new ProxyServer(...));
AsyncHttpClient client = Dsl.asyncHttpClient(clientBuilder);
```

一旦我们配置并获取了HTTP客户端的实例，我们就可以在整个应用程序中重用它。我们不需要为每个请求创建一个实例，因为它在内部会创建新的线程和连接池，这将导致性能问题。

另外，值得注意的是，**一旦我们完成使用客户端，我们应该调用close()方法以防止任何内存泄漏或挂起资源**。

## 4. 创建HTTP请求

我们可以通过两种方法使用AHC定义HTTP请求：

- 绑定
- 未绑定

就性能而言，这两种请求类型没有太大区别，它们只是代表了我们可以用来定义请求的两个独立API。**绑定请求与创建它的HTTP客户端绑定，并且默认情况下，如果没有特别指定，将使用该特定客户端的配置**。

例如，创建绑定请求时，会从HTTP客户端配置中读取disableUrlEncoding标志，而对于未绑定请求，默认情况下将其设置为false。这很有用，因为可以使用作为VM参数传递的系统属性来更改客户端配置，而无需重新编译整个应用程序：

```shell
java -jar -Dorg.asynchttpclient.disableUrlEncodingForBoundRequests=true
```

可以在ahc-default.properties文件中找到完整的属性列表。

### 4.1 绑定请求

要创建绑定请求，我们使用AsyncHttpClient类中以前缀“prepare”开头的辅助方法。此外，我们还可以使用接收已创建的Request对象的prepareRequest()方法。

例如，prepareGet()方法将创建一个HTTP GET请求：

```java
BoundRequestBuilder getRequest = client.prepareGet("http://www.tuyucheng.com");
```

### 4.2 非绑定请求

可以使用RequestBuilder类创建未绑定的请求：

```java
Request getRequest = new RequestBuilder(HttpConstants.Methods.GET)
    .setUrl("http://www.tuyucheng.com")
    .build();
```

或者通过使用Dsl帮助类，它实际上使用RequestBuilder来配置请求的HTTP方法和URL：

```java
Request getRequest = Dsl.get("http://www.tuyucheng.com").build()
```

## 5. 执行HTTP请求

该库的名称提示了我们如何执行请求，AHC支持同步和异步请求。

执行请求取决于其类型。使用绑定请求时，**我们使用BoundRequestBuilder类中的execute()方法；使用非绑定请求时，我们将使用AsyncHttpClient接口中 executeRequest()方法的实现之一来执行它**。

### 5.1 同步

该库设计为异步的，但在需要时，我们可以通过阻塞Future对象来模拟同步调用。execute()和executeRequest()方法都返回一个ListenableFuture<Response\>对象，此类扩展了Java Future接口，从而继承了get()方法，该方法可用于阻塞当前线程，直到HTTP请求完成并返回响应：

```java
Future<Response> responseFuture = boundGetRequest.execute();
responseFuture.get();
```

```java
Future<Response> responseFuture = client.executeRequest(unboundRequest);
responseFuture.get();
```

在尝试调试代码的某些部分时，使用同步调用很有用，但不建议在生产环境中使用，因为异步执行会带来更好的性能和吞吐量。

### 5.2 异步

当我们谈论异步执行时，我们也会谈论用于处理结果的监听器。AHC库提供了3种类型的监听器，可用于异步HTTP调用：

- AsyncHandler
- AsyncCompletionHandler
- ListenableFuture监听器

AsyncHandler监听器提供了在HTTP调用完成之前控制和处理HTTP调用的可能性，使用它可以处理与HTTP调用相关的一系列事件：

```java
request.execute(new AsyncHandler<Object>() {
    @Override
    public State onStatusReceived(HttpResponseStatus responseStatus) throws Exception {
        return null;
    }

    @Override
    public State onHeadersReceived(HttpHeaders headers) throws Exception {
        return null;
    }

    @Override
    public State onBodyPartReceived(HttpResponseBodyPart bodyPart) throws Exception {
        return null;
    }

    @Override
    public void onThrowable(Throwable t) {

    }

    @Override
    public Object onCompleted() throws Exception {
        return null;
    }
});
```

State枚举让我们可以控制HTTP请求的处理，通过返回State.ABORT，我们可以在特定时刻停止处理，通过使用State.CONTINUE，我们可以完成处理。

值得一提的是，**AsyncHandler不是线程安全的，在执行并发请求时不应重复使用**。

AsyncCompletionHandler继承了AsyncHandler接口的所有方法，并添加了onCompleted(Response)辅助方法来处理调用完成。所有其他监听器方法都被重写以返回State.CONTINUE，从而使代码更具可读性：

```java
request.execute(new AsyncCompletionHandler<Object>() {
    @Override
    public Object onCompleted(Response response) throws Exception {
        return response;
    }
});
```

ListenableFuture接口让我们可以添加在HTTP调用完成时运行的监听器。

另外，让我们通过使用另一个线程池来执行来自监听器的代码：

```java
ListenableFuture<Response> listenableFuture = client
    .executeRequest(unboundRequest);
listenableFuture.addListener(() -> {
    Response response = listenableFuture.get();
    LOG.debug(response.getStatusCode());
}, Executors.newCachedThreadPool());
```

此外，通过添加监听器的选项，ListenableFuture接口让我们可以将Future响应转换为CompletableFuture。

## 7. 总结

AHC是一个非常强大的库，具有许多有趣的功能。它提供了一种非常简单的方法来配置HTTP客户端，并具有执行同步和异步请求的能力。