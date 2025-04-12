---
layout: post
title:  Apache HttpClient与CloseableHttpClient
category: apache
copyright: apache
excerpt: Apache
---

## 1. 概述

[Apache HttpClient](https://www.baeldung.com/httpclient-guide)是一个流行的Java库，提供高效且功能丰富的软件包，用于实现最新HTTP标准的客户端。该库专为扩展而设计，**同时为基本HTTP方法提供强大的支持**。

在本教程中，我们将了解Apache HttpClient API的设计，我们将解释HttpClient和CloseableHttpClient之间的区别；此外，我们将学习如何使用HttpClients或HttpClientBuilder创建CloseableHttpClient实例。

最后，我们将推荐在自定义代码中使用上述哪些API。此外，我们还将研究哪些API类实现了Closeable接口，因此需要我们关闭它们的实例以释放资源。

## 2. API设计

让我们首先看看API是如何设计的，重点关注它的高级类和接口。在下面的类图中，我们将展示执行经典HTTP请求和处理HTTP响应所需的部分API：

![](/assets/images/2025/apache/apachehttpclientvscloseablehttpclient01.png)

此外，Apache HttpClient API还支持[异步](https://www.baeldung.com/httpasyncclient-tutorial)HTTP请求/响应交换，以及使用[RxJava](https://www.baeldung.com/rx-java)的响应式消息交换。

## 3. HttpClient与CloseableHttpClient

**HttpClient是一个高级接口，代表了HTTP请求执行的基本约定**，它对请求执行过程没有任何限制。此外，它还将状态管理、身份验证和重定向等具体细节留给了各个客户端实现。

我们可以将任何客户端实现强制转换为HttpClient接口，因此，我们可以使用它通过默认客户端实现来执行基本的HTTP请求：

```java
HttpClient httpClient = HttpClients.createDefault();
HttpGet httpGet = new HttpGet(serviceOneUrl);
httpClient.execute(httpGet, response -> {
      assertThat(response.getCode()).isEqualTo(HttpStatus.SC_OK);
      return response;
});
```

注意这里execute方法的第二个参数是一个HttpClientResponseHandler函数接口。

但是，上面的代码会导致[SonarQube](https://www.baeldung.com/sonar-qube)出现[阻塞问题](https://rules.sonarsource.com/java/RSPEC-2095)，原因是默认客户端实现返回了一个Closeable HttpClient实例，而该实例需要关闭。

CloseableHttpClient是一个抽象类，它实现了HttpClient接口的基类。然而，它也实现了Closeable接口；因此，我们应该在使用后关闭它的所有实例。我们可以通过使用[try-with-resources](https://www.baeldung.com/java-try-with-resources)语句或在finally子句中调用close方法来关闭它们：

```java
try (CloseableHttpClient httpClient = HttpClients.createDefault()) {
     HttpGet httpGet = new HttpGet(serviceOneUrl);
     httpClient.execute(httpGet, response -> {
        assertThat(response.getCode()).isEqualTo(HttpStatus.SC_OK);
        return response;
     });
}
```

因此，在我们的自定义代码中，我们应该使用CloseableHttpClient类，而不是HttpClient接口。

## 4. HttpClients与HttpClientBuilder

在上面的例子中，我们使用了HttpClients类中的一个静态方法来获取默认的客户端实现。HttpClients是一个实用程序类，**包含用于创建CloseableHttpClient实例的工厂方法**：

```java
CloseableHttpClient httpClient = HttpClients.createDefault();
```

我们可以使用HttpClientBuilder类实现同样的功能，**HttpClientBuilder是[构建器设计模式](https://www.baeldung.com/creational-design-patterns#builder)的一个实现**，用于创建CloseableHttpClient实例：

```java
CloseableHttpClient httpClient = HttpClientBuilder.create().build();
```

HttpClients内部使用HttpClientBuilder创建客户端实现实例，因此，我们应该在自定义代码中优先使用HttpClients。由于它是一个更高级别的类，其内部实现可能会随着新版本的发布而发生变化。

## 5. 资源管理

一旦CloseableHttpClient实例超出范围，我们就需要关闭它，原因是为了关闭相关的连接管理器。

### 5.1 自动资源释放(HttpClient 4.x)

在当前的Apache HttpClient 5版本中，客户端资源在HTTP通信后通过使用我们之前看到的HttpClientResponseHandler自动释放。

在当前版本之前，提供了CloseableHttpResponse以便向后兼容HttpClient 4.x。

CloseableHttpResponse是实现了ClassicHttpResponse接口的类，然而，ClassicHttpResponse也继承了HttpResponse、HttpEntityContainer和Closeable接口。

**底层HTTP连接由响应对象持有，以便响应内容能够直接从网络套接字流式传输**。因此，在自定义代码中，我们应该使用CloseableHttpResponse类而不是HttpResponse接口。我们还需要确保在使用响应后调用close方法：

```java
try (CloseableHttpClient httpClient = HttpClientBuilder.create().build()) {
    HttpGet httpGet = new HttpGet(serviceUrl);
    try (CloseableHttpResponse response = httpClient.execute(httpGet)) {
        HttpEntity entity = response.getEntity();
        EntityUtils.consume(entity);
    }
}
```

需要注意的是，当响应内容未完全消费时，底层连接无法安全地重新使用。在这种情况下，连接管理器将关闭并丢弃该连接。

### 5.2 重用客户端

关闭CloseableHttpClient实例并为每个请求创建一个新的实例可能是一项昂贵的操作，相反，**我们可以重用一个CloseableHttpClient实例来发送多个请求**：

```java
try (CloseableHttpClient httpClient = HttpClientBuilder.create().build()) {
      HttpGet httpGetOne = new HttpGet(serviceOneUrl);
      httpClient.execute(httpGetOne, responseOne -> {
          HttpEntity entityOne = responseOne.getEntity();
          EntityUtils.consume(entityOne);
          assertThat(responseOne.getCode()).isEqualTo(HttpStatus.SC_OK);
          return responseOne;
      });

      HttpGet httpGetTwo = new HttpGet(serviceTwoUrl);
      httpClient.execute(httpGetTwo, responseTwo -> {
           HttpEntity entityTwo = httpGetTwo.getEntity();
           EntityUtils.consume(entityTwo);
           assertThat(responseTwo.getCode()).isEqualTo(HttpStatus.SC_OK);
           return responseTwo;
      });
}
```

因此，我们避免关闭内部关联的连接管理器并创建一个新的连接管理器。

## 6. 总结

在本文中，我们探讨了Apache HttpClient的经典HTTP API，它是Java的一个流行的客户端HTTP库。

我们了解了HttpClient和CloseableHttpClient之间的区别。此外，我们建议在自定义代码中使用CloseableHttpClient。接下来，我们了解了如何使用HttpClients或HttpClientBuilder创建CloseableHttpClient实例。

最后，我们研究了CloseableHttpClient和CloseableHttpResponse类，它们都实现了Closeable接口。