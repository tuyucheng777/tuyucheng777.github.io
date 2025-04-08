---
layout: post
title:  向Jersey SSE客户端请求添加标头
category: webmodules
copyright: webmodules
excerpt: Jersey
---

## 1. 概述

在本教程中，我们将看到一种使用Jersey客户端API在[服务器发送事件](https://www.baeldung.com/spring-server-sent-events)(SSE)客户端请求中发送标头的简单方法。

我们还将介绍使用默认的Jersey传输连接器发送基本键/值标头、身份验证标头和限制标头的正确方法。

## 2. 直奔主题

我们在尝试使用SSE发送标头时可能都遇到过这种情况：

我们使用SseEventSource来接收SSE，但要构建SseEventSource，我们需要一个WebTarget实例，但它不提供添加标头的方法，客户端实例也帮不上忙。

**不过请记住，标头与SSE无关，而是与客户端请求本身有关**。

让我们看看用ClientRequestFilter可以做什么。

## 3. 依赖

要开始，我们需要Maven pom.xml文件中的[jersey-client](https://mvnrepository.com/artifact/org.glassfish.jersey.core/jersey-client)以及Jersey的[SSE依赖](https://mvnrepository.com/artifact/org.glassfish.jersey.media/jersey-media-sse)：

```xml
<dependency>
    <groupId>org.glassfish.jersey.core</groupId>
    <artifactId>jersey-client</artifactId>
    <version>3.1.1</version>
</dependency>
<dependency>
    <groupId>org.glassfish.jersey.media</groupId>
    <artifactId>jersey-media-sse</artifactId>
    <version>3.1.1</version>
</dependency>
```

## 4. 客户端请求过滤器

首先，我们将实现将标头添加到每个客户端请求的过滤器：

```java
public class AddHeaderOnRequestFilter implements ClientRequestFilter {

    public static final String FILTER_HEADER_VALUE = "filter-header-value";
    public static final String FILTER_HEADER_KEY = "x-filter-header";

    @Override
    public void filter(ClientRequestContext requestContext) throws IOException {
        requestContext.getHeaders().add(FILTER_HEADER_KEY, FILTER_HEADER_VALUE);
    }
}
```

此后，我们将注册它并使用它。

在我们的示例中，我们将使用https://sse.example.org作为虚拟端点，我们希望客户端从该端点消费事件。实际上，我们会将其更改为我们希望客户端消费的真实[SSE事件服务器端点](https://www.baeldung.com/java-ee-jax-rs-sse)。

```java
Client client = ClientBuilder.newBuilder()
    .register(AddHeaderOnRequestFilter.class)
    .build();

WebTarget webTarget = client.target("https://sse.example.org/");

SseEventSource sseEventSource = SseEventSource.target(webTarget).build();
sseEventSource.register((event) -> { /* Consume event here */ });
sseEventSource.open();
// do something here until ready to close
sseEventSource.close();
```

**现在，如果我们需要向SSE端点发送更复杂的标头(例如身份验证标头)，该怎么办**？

**让我们一起进入下一部分，了解有关Jersey Client API中的标头的更多信息**。

## 5. Jersey客户端API中的标头

**重要的是要知道，默认的Jersey传输连接器实现使用了JDK中的HttpURLConnection类，此类限制了某些标头的使用；为了避免该限制，我们可以设置系统属性**：

```java
System.setProperty("sun.net.http.allowRestrictedHeaders", "true");
```

我们可以在[Jersey文档](https://eclipse-ee4j.github.io/jersey.github.io/documentation/latest/client.html#d0e5144)中找到受限标题的列表。

### 5.1 简单通用标头

定义标头的最直接的方法是调用WebTarget#request来获取提供header方法的Invocation.Builder。

```java
public Response simpleHeader(String headerKey, String headerValue) {
    Client client = ClientBuilder.newClient();
    WebTarget webTarget = client.target("https://sse.example.org/");
    Invocation.Builder invocationBuilder = webTarget.request();
    invocationBuilder.header(headerKey, headerValue);
    return invocationBuilder.get();
}
```

实际上，我们可以很好地简化它以增加可读性：

```java
public Response simpleHeaderFluently(String headerKey, String headerValue) {
    Client client = ClientBuilder.newClient();

    return client.target("https://sse.example.org/")
        .request()
        .header(headerKey, headerValue)
        .get();
}
```

### 5.2 基本身份验证

实际上，**Jersey Client API提供了HttpAuthenticationFeature类，允许我们轻松发送身份验证标头**：

```java
public Response basicAuthenticationAtClientLevel(String username, String password) {
    HttpAuthenticationFeature feature = HttpAuthenticationFeature.basic(username, password);
    Client client = ClientBuilder.newBuilder().register(feature).build();

    return client.target("https://sse.example.org/")
        .request()
        .get();
}
```

由于我们在构建客户端时注册了该功能，因此它将应用于每个请求，API处理基本规范要求的用户名和密码的编码。

**顺便说一句，请注意，我们暗示HTTPS是我们的连接模式**。虽然这始终是一种有价值的安全措施，但在使用基本身份验证时，这是至关重要的，否则，密码将以纯文本形式公开。Jersey还支持[更复杂的安全配置](https://eclipse-ee4j.github.io/jersey.github.io/documentation/latest/user-guide.html#d0e5313)。

现在，我们还可以在请求时指定凭证：

```java
public Response basicAuthenticationAtRequestLevel(String username, String password) {
    HttpAuthenticationFeature feature = HttpAuthenticationFeature.basicBuilder().build();
    Client client = ClientBuilder.newBuilder().register(feature).build();

    return client.target("https://sse.example.org/")
        .request()
        .property(HTTP_AUTHENTICATION_BASIC_USERNAME, username)
        .property(HTTP_AUTHENTICATION_BASIC_PASSWORD, password)
        .get();
}
```

### 5.3 摘要式身份验证

Jersey的HttpAuthenticationFeature还支持摘要式身份验证：

```java
public Response digestAuthenticationAtClientLevel(String username, String password) {
    HttpAuthenticationFeature feature = HttpAuthenticationFeature.digest(username, password);
    Client client = ClientBuilder.newBuilder().register(feature).build();

    return client.target("https://sse.example.org/")
        .request()
        .get();
}
```

同样，我们可以在请求时覆盖：

```java
public Response digestAuthenticationAtRequestLevel(String username, String password) {
    HttpAuthenticationFeature feature = HttpAuthenticationFeature.digest();
    Client client = ClientBuilder.newBuilder().register(feature).build();

    return client.target("http://sse.example.org/")
        .request()
        .property(HTTP_AUTHENTICATION_DIGEST_USERNAME, username)
        .property(HTTP_AUTHENTICATION_DIGEST_PASSWORD, password)
        .get();
}
```

### 5.4 使用OAuth 2.0进行Bearer Token身份验证

**OAuth 2.0支持Bearer令牌的概念作为另一种身份验证机制**。

我们需要Jersey的[oauth2-client依赖](https://mvnrepository.com/artifact/org.glassfish.jersey.security/oauth2-client)来为我们提供与HttpAuthenticationFeature类似的OAuth2ClientSupportFeature：

```xml
<dependency>
    <groupId>org.glassfish.jersey.security</groupId>
    <artifactId>oauth2-client</artifactId>
    <version>3.1.1</version>
</dependency>
```

要添加Bearer Token，我们将遵循与之前类似的模式：

```java
public Response bearerAuthenticationWithOAuth2AtClientLevel(String token) {
    Feature feature = OAuth2ClientSupport.feature(token);
    Client client = ClientBuilder.newBuilder().register(feature).build();

    return client.target("https://sse.examples.org/")
        .request()
        .get();
}
```

或者，我们可以在请求级别进行覆盖，这在令牌由于轮换而发生变化时特别方便：

```java
public Response bearerAuthenticationWithOAuth2AtRequestLevel(String token, String otherToken) {
    Feature feature = OAuth2ClientSupport.feature(token);
    Client client = ClientBuilder.newBuilder().register(feature).build();

    return client.target("https://sse.example.org/")
        .request()
        .property(OAuth2ClientSupport.OAUTH2_PROPERTY_ACCESS_TOKEN, otherToken)
        .get();
}
```

### 5.5 使用OAuth 1.0进行Bearer Token身份验证

如果我们需要与使用OAuth 1.0的遗留代码集成，我们将需要Jersey的[oauth1-client依赖](https://mvnrepository.com/artifact/org.glassfish.jersey.security/oauth1-client)：

```xml
<dependency>
    <groupId>org.glassfish.jersey.security</groupId>
    <artifactId>oauth1-client</artifactId>
    <version>3.1.1</version>
</dependency>
```

与OAuth 2.0类似，我们有可以使用的OAuth1ClientSupport：

```java
public Response bearerAuthenticationWithOAuth1AtClientLevel(String token, String consumerKey) {
    ConsumerCredentials consumerCredential = 
      new ConsumerCredentials(consumerKey, "my-consumer-secret");
    AccessToken accessToken = new AccessToken(token, "my-access-token-secret");

    Feature feature = OAuth1ClientSupport
        .builder(consumerCredential)
        .feature()
        .accessToken(accessToken)
        .build();

    Client client = ClientBuilder.newBuilder().register(feature).build();

    return client.target("https://sse.example.org/")
        .request()
        .get();
}
```

通过OAuth1ClientSupport.OAUTH_PROPERTY_ACCESS_TOKEN属性再次启用请求级别。

## 6. 总结

总而言之，在本文中，我们介绍了如何使用过滤器在Jersey中向SSE客户端请求添加标头，我们还特别介绍了如何使用身份验证标头。