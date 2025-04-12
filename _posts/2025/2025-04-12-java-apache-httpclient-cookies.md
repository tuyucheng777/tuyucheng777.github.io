---
layout: post
title:  如何从Apache HttpClient响应中获取Cookie
category: apache
copyright: apache
excerpt: Apache HttpClient
---

## 1. 概述

在这个简短的教程中，我们将了解如何从Apache HttpClient响应中获取Cookie。 

首先，我们将展示如何通过HttpClient请求发送自定义Cookie。然后，我们将了解如何从响应中获取它。

请注意，此处提供的代码示例基于HttpClient 5.2.x及更高版本，因此它们不适用于旧版本的API。

## 2. 在请求中发送Cookie

值得注意的是，在从客户端响应中获取自定义Cookie之前，我们需要创建它并在请求中发送它：

```java
BasicCookieStore cookieStore = new BasicCookieStore();
BasicClientCookie cookie = new BasicClientCookie("custom_cookie", "test_value");
cookie.setDomain("tuyucheng.com");
cookie.setAttribute("domain", "true");
cookie.setPath("/");
cookieStore.addCookie(cookie);

HttpClientContext context = HttpClientContext.create();
context.setAttribute(HttpClientContext.COOKIE_STORE, createCustomCookieStore());

try (CloseableHttpClient client = HttpClientBuilder.create()
    .build()) {
    client.execute(request, context, new BasicHttpClientResponseHandler());
}
```

首先，我们创建一个基本的Cookie存储，以及一个名为custom_cookie、值为test_value的基本[Cookie](https://www.baeldung.com/httpclient-4-cookies)。然后，我们创建一个HttpClientContext实例来保存该Cookie存储区；最后，我们将创建的上下文作为参数传递给execute()方法。

```text
2023-09-13 20:56:59,628 [DEBUG] org.apache.hc.client5.http.headers - http-outgoing-0 >> Cookie: custom_cookie=test_value
```

在日志消息中，我们可以看到custom_cookie在请求中发送。

## 3. 访问Cookie

现在我们已经在请求中发送了自定义Cookie，让我们看看如何从响应中读取它：

```java
try (CloseableHttpClient client = HttpClientBuilder.create()
    .build()) {
    client.execute(request, context, new BasicHttpClientResponseHandler());
    CookieStore cookieStore = context.getCookieStore();
    Cookie customCookie = cookieStore.getCookies()
        .stream()
        .peek(cookie -> log.info("cookie name:{}", cookie.getName()))
        .filter(cookie -> "custom_cookie".equals(cookie.getName()))
        .findFirst()
        .orElseThrow(IllegalStateException::new);

    assertEquals("test_value", customCookie.getValue());
}
```

要从响应中获取自定义Cookie，**首先必须从context获取Cookie存储区**。然后，使用getCookies方法获取Cookie列表。之后，我们可以使用[Java Stream](https://www.baeldung.com/java-streams)对其进行迭代并搜索自定义Cookie。此外，我们记录存储区中的所有Cookie名称，这样我们就可以看到自定义Cookie已存在于请求中：

```text
[INFO] cn.tuyucheng.taketoday.httpclient.cookies.HttpClientGettingCookieValueUnitTest - cookie name:_gh_sess 
[INFO] cn.tuyucheng.taketoday.httpclient.cookies.HttpClientGettingCookieValueUnitTest - cookie name:_octo 
[INFO] cn.tuyucheng.taketoday.httpclient.cookies.HttpClientGettingCookieValueUnitTest - cookie name:custom_cookie  

```

## 4. 总结

在本文中，我们学习了如何从Apache HttpClient响应中获取Cookie。