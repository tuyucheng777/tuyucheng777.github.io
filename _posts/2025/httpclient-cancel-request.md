---
layout: post
title:  Apache HttpClient–取消请求
category: apache
copyright: apache
excerpt: Apache HttpClient
---

## 1. 概述

本快速教程介绍如何**使用Apache HttpClient取消HTTP请求**。

这对于可能长时间运行的请求或大型下载文件特别有用，否则会不必要地消耗带宽和连接。

## 2. 中止GET请求

要中止正在进行的请求，客户端可以使用：

```java
request.abort();
```

这将确保客户端不必使用整个请求主体来释放连接：

```java
@Test
void whenRequestIsCanceled_thenCorrect() throws IOException {
    HttpGet request = new HttpGet(SAMPLE_URL);
    try (CloseableHttpClient httpClient = HttpClients.createDefault()) {
        httpClient.execute(request, response -> {
            HttpEntity entity = response.getEntity();

            System.out.println("----------------------------------------");
            System.out.println(response.getCode());
            if (entity != null) {
                System.out.println("Response content length: " + entity.getContentLength());
            }
            System.out.println("----------------------------------------");
            if (entity != null) {
                // Closes this stream and releases any system resources
                entity.close();
            }
            // Do not feel like reading the response body
            // Call abort on the request object
            request.abort();

            assertThat(response.getCode()).isEqualTo(HttpStatus.SC_OK);
            return response;
        });
    }
}
```

## 3. 中止GET请求(使用HttpClient 4.5)

```java
@Test
public final void whenRequestIsCanceled_thenCorrect() throws IOException {
    instance = HttpClients.custom().build();
    final HttpGet request = new HttpGet(SAMPLE_URL);
    response = instance.execute(request);

    try {
        final HttpEntity entity = response.getEntity();

        System.out.println("----------------------------------------");
        System.out.println(response.getStatusLine());
        if (entity != null) {
            System.out.println("Response content length: " + entity.getContentLength());
        }
        System.out.println("----------------------------------------");

        // Do not feel like reading the response body
        // Call abort on the request object
        request.abort();
    } finally {
        response.close();
    }
}
```

## 4. 总结

本文演示了如何使用HTTP客户端中止正在进行的请求，停止长时间运行的请求的另一个方法是确保它们[超时](https://www.baeldung.com/httpclient-timeout)。