---
layout: post
title:  Apache HttpClient–遵循POST的重定向
category: apache
copyright: apache
excerpt: Apache HttpClient
---

## 1. 概述

本快速教程将展示如何配置Apache HttpClient以自动遵循POST请求的重定向。

如果你想深入了解使用HttpClient可以做的其他有趣的事情-请直接转到[主要的HttpClient教程](https://www.baeldung.com/httpclient-guide)。

## 2. HTTP POST重定向

### 2.1 对于HttpClient 5.x

默认情况下，GET和POST请求都会自动重定向，此功能与之前的版本(4.5.x)不同，我们将在下一节中介绍。

```java
@Test
void givenRedirectingPOST_whenUsingDefaultRedirectStrategy_thenRedirected() throws IOException {
    final HttpPost request = new HttpPost("http://t.co/I5YYd9tddw");

    try (CloseableHttpClient httpClient = HttpClientBuilder.create()
            .setRedirectStrategy(new DefaultRedirectStrategy())
            .build()) {
        httpClient.execute(request, response -> {
            assertThat(response.getCode(), equalTo(200));
            return response;
        });
    }
}
```

请注意，使用DefaultRedirectStrategy并通过POST进行重定向-导致200 OK状态码。

即使我们不使用新的DefaultRedirectStrategy()，也会遵循重定向：

```java
@Test
void givenRedirectingPOST_whenConsumingUrlWhichRedirectsWithPOST_thenRedirected() throws IOException {
    final HttpPost request = new HttpPost("http://t.co/I5YYd9tddw");

    try (CloseableHttpClient httpClient = HttpClientBuilder.create()
            .build()) {
        httpClient.execute(request, response -> {
            assertThat(response.getCode(), equalTo(200));
            return response;
        });
    }
}
```

### 2.2 对于HttpClient 4.5

在HttpClient 4.5中，默认情况下，只有导致重定向的GET请求才会自动执行。如果POST请求的响应为HTTP 301 Moved Permanently或302 Found，**则不会自动执行重定向**。

这是由[HTTP RFC 2616](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.3)指定的：

> 如果响应除GET或HEAD之外的请求而收到301状态码，则用户代理不得自动重定向请求，除非用户能够确认，因为这可能会改变发出请求的条件。

当然，在某些情况下我们需要改变这种行为并放宽严格的HTTP规范。

首先，让我们检查一下默认行为：

```java
@Test
public void givenPostRequest_whenConsumingUrlWhichRedirects_thenNotRedirected() throws ClientProtocolException, IOException {
    HttpClient instance = HttpClientBuilder.create().build();
    HttpResponse response = instance.execute(new HttpPost("http://t.co/I5YYd9tddw"));
    assertThat(response.getStatusLine().getStatusCode(), equalTo(301));
}
```

如你所见，**默认情况下不遵循重定向**，并且我们返回了301状态码。

现在让我们看看如何通过设置重定向策略来遵循重定向：

```java
@Test
public void givenRedirectingPOST_whenConsumingUrlWhichRedirectsWithPOST_thenRedirected() throws ClientProtocolException, IOException {
    HttpClient instance = HttpClientBuilder.create().setRedirectStrategy(new LaxRedirectStrategy()).build();
    HttpResponse response = instance.execute(new HttpPost("http://t.co/I5YYd9tddw"));
    assertThat(response.getStatusLine().getStatusCode(), equalTo(200));
}
```

使用LaxRedirectStrategy，HTTP限制会放宽，并且重定向也会通过POST进行-导致200 OK状态码。

## 3. 总结

本快速指南说明了如何配置任何版本的Apache HttpClient以遵循HTTP POST请求的重定向-放宽严格的HTTP标准。