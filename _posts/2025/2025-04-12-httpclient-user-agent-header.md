---
layout: post
title:  Apache HttpClient中的自定义User-Agent
category: apache
copyright: apache
excerpt: Apache HttpClient
---

## 1. 概述

本快速教程将展示**如何使用Apache HttpClient发送自定义User-Agent标头**。

## 2. 在HttpClient上设置User-Agent

我们可以在配置客户端本身时设置User-Agent：

```java
HttpClients.custom().setUserAgent("Mozilla/5.0 Firefox/26.0").build();
```

完整的示例如下：

```java
@Test
void whenClientUsesCustomUserAgent_thenCorrect() throws IOException {
    CloseableHttpClient client = HttpClients.custom()
        .setUserAgent("Mozilla/5.0 Firefox/26.0")
        .build();
    final HttpGet request = new HttpGet(SAMPLE_URL);

    String response = client.execute(request, new BasicHttpClientResponseHandler());
    logger.info("Response -> {}", response);
}
```

## 3. 在单个请求中设置User-Agent

还可以在单个请求上设置自定义User-Agent标头，从而为我们的客户端增加更多灵活性：

```java
@Test
void whenRequestHasCustomUserAgent_thenCorrect() throws IOException {
    CloseableHttpClient client = HttpClients.createDefault();
    final HttpGet request = new HttpGet(SAMPLE_URL);
    request.setHeader(HttpHeaders.USER_AGENT, "Mozilla/5.0 Firefox/26.0");

    String response = client.execute(request, new BasicHttpClientResponseHandler());
    logger.info("Response -> {}", response);
}
```

## 4. 总结

本文说明了如何使用HttpClient发送带有自定义User-Agent标头的请求-例如，模拟特定浏览器的行为。