---
layout: post
title:  在Spring WebClient中一次设置多个标头
category: springreactive
copyright: springreactive
excerpt: Spring WebClient
---

## 1. 简介

在本教程中，我们将了解如何在[Spring WebClient](https://www.baeldung.com/spring-5-webclient)中一次设置多个标头。

WebClient是[Spring WebFlux](https://www.baeldung.com/spring-webflux)中的一个类，简单来说，它支持同步和异步HTTP请求。我们将首先了解WebClient如何处理标头，然后通过代码示例探索一次性设置多个标头的不同方法。

## 2. WebClient如何处理标头

一般来说，HTTP请求中的标头充当元数据，它们携带诸如身份验证详细信息、内容类型、版本等信息。

**在WebClient中，[HttpHeaders](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/HttpHeaders.html)类负责管理标头，它是Spring框架专门设计用于表示请求和响应标头的类。它实现了MultiValueMap<String, String\>接口，允许单个标头键拥有多个值**。

这为需要多个值的标头(例如[Accept](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Accept))提供了灵活性。

## 3. 在WebClient中设置多个标头

有几种方法可以向请求添加标头，**根据用例，我们可以为单个请求设置标头，为整个WebClient实例定义全局标头，或者动态修改它们**。

让我们通过代码示例来探索这些方法。

### 3.1 为单个请求设置标头

当标头特定于单个请求并且因端点而异时，直接的方法是在请求上直接设置它们。

在以下示例中，我们创建一个简单的测试，实例化WebClient，向请求添加两个标头，并断言该请求已通过这些标头发送，我们还使用了Okhttp3库中的[MockWebServer](https://www.baeldung.com/spring-mocking-webclient)来模拟服务器响应并验证WebClient的行为：

```java
@Test
public void givenRequestWithHeaders_whenSendingRequest_thenAssertHeadersAreSent() throws Exception {
    mockWebServer.enqueue(new MockResponse().setResponseCode(HttpStatus.OK.value()));

    WebClient client = WebClient.builder()
            .baseUrl(mockWebServer.url("/").toString())
            .build();

    ResponseEntity<Void> response = client.get()
            .headers(headers -> {
                headers.put("X-Request-Id", Collections.singletonList(RANDOM_UUID));
                headers.put("Custom-Header", Collections.singletonList("CustomValue"));
            })
            .retrieve()
            .toBodilessEntity()
            .block();

    assertNotNull(response);
    assertEquals(HttpStatusCode.valueOf(HttpStatus.OK.value()), response.getStatusCode());

    RecordedRequest recordedRequest = mockWebServer.takeRequest();
    assertEquals(RANDOM_UUID, recordedRequest.getHeader("X-Request-Id"));
    assertEquals("CustomValue", recordedRequest.getHeader("Custom-Header"));
}
```

此示例使用WebClient类中的headers(Consumer<HttpHeaders\> headersConsumer)方法。

如前所述，WebClient依赖于HttpHeaders，在此上下文中，HttpHeaders使用Consumer[函数接口](https://www.baeldung.com/java-8-functional-interfaces)进行配置，**此设置允许我们通过传递对HttpHeaders实例进行操作的Lambda来修改请求标头**。

### 3.2 全局设置默认标头

在其他情况下，我们可能需要定义全局标头，这些标头在全局级别配置，并会自动添加到使用此客户端实例发出的每个请求中，这种配置有助于我们保持一致性并减少重复。

**我们始终可以用请求特定的标头覆盖全局标头，因为它们仅充当基准**。与以前的方法唯一的区别是，我们在构建WebClient时添加它们：

```java
WebClient client = WebClient.builder()
    .baseUrl(mockWebServer.url("/").toString())
    .defaultHeaders(headers -> {
        headers.put("X-Request-Id", Collections.singletonList(RANDOM_UUID));
        headers.put("Custom-Header", Collections.singletonList("CustomValue"));
    })
    .build();
```

### 3.3 使用ExchangeFilterFunction动态修改标头

在某些情况下，我们可能希望在运行时动态设置或修改标头，对于这种情况，我们可以使用ExchangeFilterFunction类：

```java
ExchangeFilterFunction dynamicHeadersFilter = (request, next) -> next.exchange(ClientRequest.from(request)
    .headers(headers -> {
        headers.put("X-Request-Id", Collections.singletonList(RANDOM_UUID));
        headers.put("Custom-Header", Collections.singletonList("CustomValue"));
    })
    .build());
```

构建ExchangeFilterFunction实例后，我们在实例化期间将其注册到WebClient：

```java
WebClient client = WebClient.builder()
    .baseUrl(mockWebServer.url("/").toString())
    .filter(dynamicHeadersFilter)
    .build();
```

**还值得注意的是，我们可以为单个WebClient堆叠多个过滤函数实例**。

## 4. 总结

在本文中，我们探讨了Spring WebClient如何处理标头，并了解了几种设置多个标头的方法。无论是针对单个请求、全局所有请求，还是在运行时动态设置，WebClient都提供了一种简单灵活的方法来一致、清晰地管理标头。