---
layout: post
title:  在Java中将HTTP响应主体读取为字符串
category: apache
copyright: apache
excerpt: Apache HttpClient
---

## 1. 简介

在本教程中，我们将探索几个用于在Java中将HTTP响应主体读取为字符串的库。自第一个版本以来，Java就提供了[HttpURLConnection](https://www.baeldung.com/java-http-request) API，该API仅包含基本功能，并且以不太用户友好而闻名。

在JDK 11中，Java引入了全新改进的[HttpClient](https://www.baeldung.com/java-9-http-client) API来处理HTTP通信。我们将讨论这些库，并介绍一些替代方案，例如[Apache HttpClient](https://www.baeldung.com/httpclient-guide)和[Spring Rest Template](https://www.baeldung.com/rest-template)。

## 2. HttpClient

如前所述，[HttpClient](https://www.baeldung.com/java-9-http-client)已添加到Java 11中，它允许我们通过网络访问资源，但与HttpURLConnection不同，**HttpClient支持HTTP/1.1和HTTP/2**。此外，它还**提供了同步和异步请求类型**。

HttpClient提供了一个灵活且功能强大的现代API，该API由三个核心类组成：HttpClient、HttpRequest和HttpResponse。

HttpResponse描述了HttpRequest调用的结果，**HttpResponse不会直接创建，而是在请求体完全接收后才可用**。

要将响应主体读取为字符串，我们首先需要创建简单的客户端和请求对象：

```java
HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create(DUMMY_URL))
    .build();
```

然后我们将使用BodyHandlers并调用方法ofString()来返回响应：

```java
HttpResponse response = client.send(request, HttpResponse.BodyHandlers.ofString());
```

## 3. HttpURLConnection

[HttpURLConnection](https://www.baeldung.com/java-http-request)是一个轻量级的HTTP客户端，用于通过HTTP或HTTPS协议访问资源，它允许我们创建一个InputStream。获取到InputStream后，我们就可以像读取普通本地文件一样读取它。

在Java中，我们可以用来访问互联网的主要类是java.net.URL类和java.net.HttpURLConnection类。首先，我们使用URL类指向一个Web资源。然后，我们可以使用HttpURLConnection类来访问它。

要从URL获取响应主体作为String，我们应该首先使用URL创建一个HttpURLConnection：

```java
HttpURLConnection connection = (HttpURLConnection) new URL(DUMMY_URL).openConnection();
```

new URL(DUMMY_URL).openConnection()返回一个HttpURLConnection对象，该对象允许我们添加标头或检查响应码。

接下来，我们**从connection对象中获取InputStream**：

```java
InputStream inputStream = connection.getInputStream();
```

最后，我们需要将[InputStream转换为String](https://www.baeldung.com/convert-input-stream-to-string)。

## 4. Apache HttpClient

在本节中，我们将学习如何使用[Apache HttpClient](https://www.baeldung.com/httpclient-guide)将HTTP响应主体读取为字符串。

要使用这个库，我们需要将它的依赖添加到我们的Maven项目中：

```xml
<dependency>
    <groupId>org.apache.httpcomponents.client5</groupId>
    <artifactId>httpclient5</artifactId>
    <version>5.2</version>
</dependency>
```

我们可以**通过CloseableHttpClient类检索和发送数据**，要使用默认配置创建它的实例，我们可以使用HttpClients.createDefault()。

CloseableHttpClient提供了一个执行方法来发送和接收数据，该方法使用两个参数，第一个参数是HttpUriRequest类型，该类型有许多子类，包括HttpGet和HttpPost；第二个参数是[HttpClientResponseHandler](https://hc.apache.org/httpcomponents-core-5.2.x/current/httpcore5/apidocs/org/apache/hc/core5/http/io/HttpClientResponseHandler.html)类型，它从ClassicHttpResponse生成一个响应对象。

首先，我们**创建一个HttpGet对象**：

```java
HttpGet request = new HttpGet(DUMMY_URL);
```

其次，我们将**创建客户端**：

```java
CloseableHttpClient client = HttpClients.createDefault();
```

最后，我们将从execute方法的结果中**检索响应对象**：

```java
String response = client.execute(request, new BasicHttpClientResponseHandler());
logger.debug("Response -> {}", response);
```

这里我们使用了BasicHttpClientResponseHandler，它将响应主体作为字符串返回。

## 5. Spring RestTemplate

在本节中，我们将演示如何使用[Spring RestTemplate](https://www.baeldung.com/rest-template)将HTTP响应主体读取为字符串。需要注意的是，**RestTemplate现已弃用**，因此，我们应该考虑使用Spring WebClient，如下一节所述。

RestTemplate类是Spring提供的一个重要工具，**它提供了一个简单的模板**，用于通过底层HTTP客户端库(例如JDK HttpURLConnection、Apache HttpClient等)进行客户端HTTP操作。

RestTemplate提供了一些用于创建HTTP请求和处理响应的[有用方法](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html)。

我们可以通过首先向Maven项目添加一些依赖来使用这个库：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>${spring-boot.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <version>${spring-boot.version}</version>
    <scope>test</scope>
</dependency>
```

为了发出Web请求并以字符串形式返回响应主体，我们将**创建一个RestTemplate实例**：

```java
RestTemplate restTemplate = new RestTemplate();
```

然后，我们**通过调用getForObject()方法获取响应对象，并传入URL和所需的响应类型**，在本例中使用String.class：

```java
String response = restTemplate.getForObject(DUMMY_URL, String.class);
```

## 6. Spring WebClient

最后，我们将介绍如何**使用[Spring WebClient](https://www.baeldung.com/spring-5-webclient)，它是替代Spring RestTemplate的响应式、非阻塞解决方案**。

我们可以通过将[spring-boot-starter-webflux](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-webflux)依赖添加到Maven项目来使用这个库：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

创建客户端最简单方法是使用create方法：

```java
WebClient webClient = WebClient.create(DUMMY_URL);
```

执行HTTP Get请求最简单的方法是调用get和retrieve方法，**然后，我们将使用bodyToMono方法和String.class类型将请求体提取为单个String实例**：

```java
Mono<String> body = webClient.get().retrieve().bodyToMono(String.class);
```

最后，我们将**调用block方法来告诉WebFlux等待，直到整个body流被读取并复制到String结果中**：

```java
String s = body.block();
```

## 7. 总结

在本文中，我们学习了如何使用几个库将HTTP响应主体读取为String。