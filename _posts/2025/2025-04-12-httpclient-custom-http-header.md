---
layout: post
title:  使用Apache HttpClient自定义HTTP标头
category: apache
copyright: apache
excerpt: Apache HttpClient
---

## 1. 概述

在本教程中，我们将研究如何使用HttpClient设置自定义标头。

如果你想深入了解使用HttpClient可以做的其他有趣的事情-请直接转到[主要的HttpClient教程](https://www.baeldung.com/httpclient-guide)。

## 2. 设置请求头 

我们可以通过对请求进行简单的setHeader调用来设置任何自定义标头：

```java
@Test
void whenRequestHasCustomContentType_thenCorrect() throws IOException {
    final HttpGet request = new HttpGet(SAMPLE_URL);
    request.setHeader(HttpHeaders.CONTENT_TYPE, "application/json");

    try (CloseableHttpClient client = HttpClients.createDefault()) {
        String response = client.execute(request, new BasicHttpClientResponseHandler());
        //do something with response
    }
}
```

可以看到，我们将请求中的Content-Type直接设置为自定义值JSON。

## 3. 使用HttpClient 4.5的RequestBuilder设置请求头

在HttpClient 4.5中，我们可以使用RequestBuilder来设置请求头。要设置请求头，**我们需要在构建器中使用setHeader方法**：

```java
HttpClient client = HttpClients.custom().build();
HttpUriRequest request = RequestBuilder.get()
    .setUri(SAMPLE_URL)
    .setHeader(HttpHeaders.CONTENT_TYPE, "application/json")
    .build();
client.execute(request);
```

## 4. 在客户端设置默认标头

我们不需要在每个请求上都设置标头，而是可以**将其配置为客户端本身的默认标头**：

```java
@Test
void givenConfigOnClient_whenRequestHasCustomContentType_thenCorrect() throws IOException {
    final HttpGet request = new HttpGet(SAMPLE_URL);
    final Header header = new BasicHeader(HttpHeaders.CONTENT_TYPE, "application/json");
    final List<Header> headers = Lists.newArrayList(header);
    
    try (CloseableHttpClient client = HttpClients.custom()
        .setDefaultHeaders(headers)
        .build()) {
        
        String response = client.execute(request, new BasicHttpClientResponseHandler());
       //do something with response
    }
}
```

当所有请求的标头都需要相同时(例如自定义应用程序标头)，这非常有用。

## 5. 总结

本文说明了如何向通过Apache HttpClient发送的一个或所有请求添加HTTP标头。