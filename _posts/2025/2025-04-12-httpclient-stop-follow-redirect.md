---
layout: post
title:  Apache HttpClient–不遵循重定向
category: apache
copyright: apache
excerpt: Apache HttpClient
---

## 1. 概述

在本文中，我们将学习如何配置Apache HttpClient以停止遵循重定向。

默认情况下，遵循HTTP规范，HttpClient将自动遵循重定向。

对于某些用例来说，这可能完全没问题，但肯定存在一些不希望出现的用例-我们现在将研究如何更改默认行为并停止遵循重定向。

如果你想深入了解使用HttpClient可以做的其他有趣的事情，请转到[主要的HttpClient教程](https://www.baeldung.com/httpclient-guide)。

## 2. 不遵循重定向

### 2.1 配置-HttpClient 5

```java
@Test
void givenRedirectsAreDisabled_whenConsumingUrlWhichRedirects_thenNotRedirected() throws IOException {
    final HttpGet request = new HttpGet("http://t.co/I5YYd9tddw");

    try (CloseableHttpClient httpClient = HttpClients.custom()
            .disableRedirectHandling()
            .build()) {
        httpClient.execute(request, response -> {
            assertThat(response.getCode(), equalTo(301));
            return response;
        });
    }
}
```

### 2.1 配置-HttpClient 4.5

```java
@Test
void givenRedirectsAreDisabled_whenConsumingUrlWhichRedirects_thenNotRedirected() throws IOException {
    CloseableHttpClient instance = HttpClients.custom().disableRedirectHandling().build();

    final HttpGet httpGet = new HttpGet("http://t.co/I5YYd9tddw");
    CloseableHttpResponse response = instance.execute(httpGet);

    assertThat(response.getStatusLine().getStatusCode(), equalTo(301));
}
```

## 3. 总结

本快速教程介绍了如何配置Apache HttpClient以防止其自动遵循HTTP重定向。