---
layout: post
title:  使用Apache HttpClient扩展缩短的URL
category: apache
copyright: apache
excerpt: Apache HttpClient
---

## 1. 概述

在本文中，我们将展示如何使用HttpClient扩展URL。

一个简单的例子是，原始URL已被缩短一次–通过bit.ly等服务。

更复杂的例子是，当URL被不同的服务缩短多次时，需要多次传递才能获得原始的完整URL。

## 2. 扩展URL一次

我们从简单的开始，扩展仅通过缩短URL服务传递过一次的URL。

我们首先需要的是一个**不会自动遵循重定向的HTTP客户端**：

```java
CloseableHttpClient client = HttpClientBuilder.create().disableRedirectHandling().build();
```

这是必要的，因为我们需要手动拦截重定向响应并从中提取信息。

我们首先向缩短的URL发送请求-我们收到的响应将是301 Moved Permanently。

然后，我们需要提取指向下一个URL的Location标头，在本例中是最终URL：

```java
private String expandSingleLevel(final String url) throws IOException {
    try {
        HttpHead request = new HttpHead(url);
        String expandedUrl = httpClient.execute(request, response -> {
            final int statusCode = response.getCode();
            if (statusCode != 301 && statusCode != 302) {
                return url;
            }
            final Header[] headers = response.getHeaders(HttpHeaders.LOCATION);
            Preconditions.checkState(headers.length == 1);

            return headers[0].getValue();
        });
        return expandedUrl;
    } catch (final IllegalArgumentException uriEx) {
        return url;
    }
}
```

最后，使用“未缩短”的URL进行简单的实时测试：

```java
@Test
public final void givenShortenedOnce_whenUrlIsExpanded_thenCorrectResult() throws IOException {
    final String expectedResult = "https://www.tuyucheng.com/rest-versioning";
    final String actualResult = expandSingleLevel("http://bit.ly/3LScTri");
    assertThat(actualResult, equalTo(expectedResult));
}
```

## 3. 处理多个URL级别

短网址的问题在于，它们可能会被不同的服务多次缩短，扩展这样的网址需要多次才能回到原始网址。

我们将应用先前定义的expandSingleLevel原始操作来简单地遍历所有中间URL并到达最终目标：

```java
public String expand(String urlArg) throws IOException {
    String originalUrl = urlArg;
    String newUrl = expandSingleLevel(originalUrl);
    while (!originalUrl.equals(newUrl)) {
        originalUrl = newUrl;
        newUrl = expandSingleLevel(originalUrl);
    }
    return newUrl;
}
```

现在，有了扩展多级URL的新机制，让我们定义一个测试并使其运行起来：

```java
@Test
public final void givenShortenedMultiple_whenUrlIsExpanded_thenCorrectResult() throws IOException {
    final String expectedResult = "https://www.tuyucheng.com/rest-versioning";
    final String actualResult = expand("http://t.co/e4rDDbnzmk");
    assertThat(actualResult, equalTo(expectedResult));
}
```

这次，短网址“http://t.co/e4rDDbnzmk”实际上被缩短了两次，一次通过bit.ly，第二次通过t.co服务，被正确扩展为原始网址。

## 4. 检测重定向循环

最后，有些URL无法展开，因为它们形成了重定向循环。这类问题本来可以被HttpClient检测到，但自从我们关闭了重定向的自动跟踪功能后，它就无法再检测到了。

URL扩展机制的最后一步是检测重定向循环，并在发生此类循环时快速失败。

为了使其有效，我们需要从之前定义的expandSingleLevel方法中获取一些额外的信息-主要是，我们还需要返回响应的状态码以及URL。

由于Java不支持多个返回值，我们将把信息包装在org.apache.commons.lang3.tuple.Pair对象中，该方法的新签名现在是：

```java
public Pair<Integer, String> expandSingleLevelSafe(String url) throws IOException {}
```

最后，让我们将重定向循环检测纳入主扩展机制中：

```java
public String expandSafe(String urlArg) throws IOException {
    String originalUrl = urlArg;
    String newUrl = expandSingleLevelSafe(originalUrl).getRight();
    List<String> alreadyVisited = Lists.newArrayList(originalUrl, newUrl);
    while (!originalUrl.equals(newUrl)) {
        originalUrl = newUrl;
        Pair<Integer, String> statusAndUrl = expandSingleLevelSafe(originalUrl);
        newUrl = statusAndUrl.getRight();
        boolean isRedirect = statusAndUrl.getLeft() == 301 || statusAndUrl.getLeft() == 302;
        if (isRedirect && alreadyVisited.contains(newUrl)) {
            throw new IllegalStateException("Likely a redirect loop");
        }
        alreadyVisited.add(newUrl);
    }
    return newUrl;
}
```

就是这样，expandSafe机制能够通过任意数量的URL缩短服务来扩展URL，同时正确地在重定向循环中快速失败。

## 5. 总结

本教程讨论了如何使用Apache HttpClient在Java中扩展短URL。

我们从一个简单的用例开始，其中URL仅缩短一次，然后实现一个更通用的机制，能够处理多级重定向并在此过程中检测重定向循环。