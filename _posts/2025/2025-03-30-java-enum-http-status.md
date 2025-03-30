---
layout: post
title:  包含所有HTTP状态码的Java枚举
category: libraries
copyright: libraries
excerpt: OkHttp、RestTemplate、Apache HttpComponent
---

## 1. 简介

[枚举](https://www.baeldung.com/a-guide-to-java-enums)提供了一种在Java编程语言中定义一组命名常量的强大方法，它们对于表示一组固定的相关值(例如[HTTP状态码](https://www.baeldung.com/cs/http-status-codes))非常有用。众所周知，互联网上的所有Web服务器都会发出HTTP状态码作为标准响应代码。

**在本教程中，我们将深入研究如何创建包含所有HTTP状态码的Java枚举**。

## 2. 理解HTTP状态码

HTTP状态码在网络通信中发挥着至关重要的作用，它可以告知客户端其请求的结果。**此外，服务器会发出这些3位数的代码，这些代码分为5类，每类在HTTP协议中都发挥着特定的作用**。

## 3. 使用枚举作为HTTP状态码的好处

在Java中枚举HTTP状态码有几个优点，包括：

- 类型安全：使用枚举确保类型安全，使我们的代码更具可读性和可维护性
- 分组常量：枚举将相关常量组合在一起，提供一种清晰、结构化的方式来处理固定值集
- 避免使用硬编码值：将HTTP状态码定义为枚举有助于防止硬编码字符串或整数导致的错误
- 增强清晰度和可维护性：这种方法通过增强清晰度、减少错误和提高代码可维护性来促进软件开发的最佳实践

## 4. 基本方法

为了有效地管理Java应用程序中的HTTP状态码，我们可以定义一个枚举来封装所有标准HTTP状态码及其描述。

这种方法使我们能够利用枚举的优点，例如类型安全性和代码清晰度。让我们从定义HttpStatus枚举开始：

```java
public enum HttpStatus {
    CONTINUE(100, "Continue"),
    SWITCHING_PROTOCOLS(101, "Switching Protocols"),
    OK(200, "OK"),
    CREATED(201, "Created"),
    ACCEPTED(202, "Accepted"),
    MULTIPLE_CHOICES(300, "Multiple Choices"),
    MOVED_PERMANENTLY(301, "Moved Permanently"),
    FOUND(302, "Found"),
    BAD_REQUEST(400, "Bad Request"),
    UNAUTHORIZED(401, "Unauthorized"),
    FORBIDDEN(403, "Forbidden"),
    NOT_FOUND(404, "Not Found"),
    INTERNAL_SERVER_ERROR(500, "Internal Server Error"),
    NOT_IMPLEMENTED(501, "Not Implemented"),
    BAD_GATEWAY(502, "Bad Gateway"),
    UNKNOWN(-1, "Unknown Status");

    private final int code;
    private final String description;

    HttpStatus(int code, String description) {
        this.code = code;
        this.description = description;
    }

    public static HttpStatus getStatusFromCode(int code) {
        for (HttpStatus status : HttpStatus.values()) {
            if (status.getCode() == code) {
                return status;
            }
        }
        return UNKNOWN;
    }

    public int getCode() {
        return code;
    }

    public String getDescription() {
        return description;
    }
}
```

此枚举中的每个常量都与一个整数代码和一个字符串描述相关联。此外，构造函数初始化这些值，我们提供Getter方法来检索它们。

让我们创建单元测试来确保我们的HttpStatus枚举类正常工作：

```java
@Test
public void givenStatusCode_whenGetCode_thenCorrectCode() {
    assertEquals(100, HttpStatus.CONTINUE.getCode());
    assertEquals(200, HttpStatus.OK.getCode());
    assertEquals(300, HttpStatus.MULTIPLE_CHOICES.getCode());
    assertEquals(400, HttpStatus.BAD_REQUEST.getCode());
    assertEquals(500, HttpStatus.INTERNAL_SERVER_ERROR.getCode());
}

@Test
public void givenStatusCode_whenGetDescription_thenCorrectDescription() {
    assertEquals("Continue", HttpStatus.CONTINUE.getDescription());
    assertEquals("OK", HttpStatus.OK.getDescription());
    assertEquals("Multiple Choices", HttpStatus.MULTIPLE_CHOICES.getDescription());
    assertEquals("Bad Request", HttpStatus.BAD_REQUEST.getDescription());
    assertEquals("Internal Server Error", HttpStatus.INTERNAL_SERVER_ERROR.getDescription());
}
```

在这里，我们验证getCode()和getDescription()方法是否为各种HTTP状态码返回正确的值。第一个测试方法检查getCode()方法是否为每个枚举常量返回正确的整数码。同样，第二个测试方法确保getDescription()方法返回适当的字符串描述。

## 5. 使用Apache HttpComponents

Apache [HttpComponents](https://www.baeldung.com/httpclient-guide)是一个流行的HTTP通信库，要将其与Maven一起使用，我们在pom.xml中包含以下依赖：

```xml
<dependency>
    <groupId>org.apache.httpcomponents.client5</groupId>
    <artifactId>httpclient5</artifactId>
    <version>5.3.1</version>
</dependency>
```

我们可以在[Maven Central](https://mvnrepository.com/artifact/org.apache.httpcomponents.client5/httpclient5/)上找到有关此依赖的更多详细信息。

我们可以使用HttpStatus枚举来处理HTTP响应：

```java
@Test
public void givenHttpRequest_whenUsingApacheHttpComponents_thenCorrectStatusDescription() throws IOException {
    CloseableHttpClient httpClient = HttpClients.createDefault();
    HttpGet request = new HttpGet("http://example.com");
    try (CloseableHttpResponse response = httpClient.execute(request)) {
        String reasonPhrase = response.getStatusLine().getReasonPhrase();
        assertEquals("OK", reasonPhrase);
    }
}
```

在这里，我们首先使用createDefault()方法创建一个CloseableHttpClient实例。**此外，此客户端负责发出HTTP请求**。然后，我们使用new HttpGet("http://example.com")构造对http://example.com的HTTP GET请求。通过使用execute()方法执行请求，我们收到一个CloseableHttpResponse对象。

**从此响应中，我们使用response.getStatusLine().getStatusCode()提取状态码**，然后我们使用HttpStatusUtil.getStatusDescription()检索与状态码相关的状态描述。

最后，我们使用assertEquals()来确保描述与预期值匹配，从而验证我们的状态码处理是准确的。

## 6. 使用RestTemplate框架

Spring框架的[RestTemplate](https://www.baeldung.com/rest-template)也可以从我们的HttpStatus枚举中受益，用于处理HTTP响应。让我们首先在pom.xml中包含以下依赖：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>6.1.11</version>
</dependency>
```

我们可以在[Maven Central](https://mvnrepository.com/artifact/org.springframework/spring-web)上找到有关此依赖的更多详细信息。

让我们探索如何通过简单的实现来利用这种方法：

```java
@Test
public void givenHttpRequest_whenUsingSpringRestTemplate_thenCorrectStatusDescription() {
    RestTemplate restTemplate = new RestTemplate();
    ResponseEntity<String> response = restTemplate.getForEntity("http://example.com", String.class);
    int statusCode = response.getStatusCode().value();
    String statusDescription = HttpStatus.getStatusFromCode(statusCode).getDescription();
    assertEquals("OK", statusDescription);
}
```

这里我们创建一个RestTemplate实例来执行HTTP GET请求。**获取ResponseEntity对象后，我们使用response.getStatusCode().value()提取状态码**。然后我们将此状态码传递给HttpStatus.getStatusFromCode.getDescription()以检索相应的状态描述。

## 7. 使用OkHttp库 

[OkHttp](https://www.baeldung.com/guide-to-okhttp)是Java中另一个广泛使用的HTTP客户端库，让我们通过在pom.xml中添加以下依赖将该库合并到Maven项目中：

```xml
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>4.12.0</version>
</dependency>
```

我们可以在[Maven Central](https://mvnrepository.com/artifact/com.squareup.okhttp3/okhttp/)上找到有关此依赖的更多详细信息。

现在，让我们将HttpStatus枚举与OkHttp集成来处理响应：

```java
@Test
public void givenHttpRequest_whenUsingOkHttp_thenCorrectStatusDescription() throws IOException {
    OkHttpClient client = new OkHttpClient();
    Request request = new Request.Builder()
            .url("http://example.com")
            .build();
    try (Response response = client.newCall(request).execute()) {
        int statusCode = response.code();
        String statusDescription = HttpStatus.getStatusFromCode(statusCode).getDescription();
        assertEquals("OK", statusDescription);
    }
}
```

在这个测试中，我们初始化一个OkHttpClient实例并使用Request.Builder()创建HTTP GET请求。**然后我们使用client.newCall(request).execute()方法执行请求并获取Response对象**，我们使用response.code()方法提取状态码并将其传递给HttpStatus.getStatusFromCode.getDescription()方法以获取状态描述。

## 8. 总结

在本文中，我们讨论了使用Java枚举来表示HTTP状态码，增强代码的可读性、可维护性和类型安全性。

无论我们选择简单的枚举定义还是将其与各种Java库(如Apache HttpComponents、Spring RestTemplate或OkHttp)一起使用，枚举都足够强大，可以处理Java中固定的相关常量集。