---
layout: post
title:  在Java中查找URL的重定向URL
category: java-net
copyright: java-net
excerpt: Java Network
---

## 1. 简介

了解[URL](https://www.baeldung.com/java-url)的重定向方式对于Web开发和网络编程任务(例如处理[HTTP](https://www.baeldung.com/java-http-request)重定向、验证URL[重定向](https://www.baeldung.com/spring-redirect-and-forward)或提取最终目标URL)至关重要，在Java中，我们可以使用HttpURLConnection或HttpClient库来实现此功能。

**在本教程中，我们将探讨在Java中查找给定URL的重定向URL的不同方法**。

## 2. 使用HttpURLConnection

Java提供了[HttpURLConnection](https://www.baeldung.com/httpurlconnection-post)类，它允许我们发出HTTP请求并处理响应。此外，我们还可以利用HttpURLConnection查找给定URL的重定向URL，具体操作如下：

```java
String canonicalUrl = "http://www.taketoday.com/";
String expectedRedirectedUrl = "https://www.taketoday.com/";

@Test
public void givenOriginalUrl_whenFindRedirectUrlUsingHttpURLConnection_thenCorrectRedirectedUrlReturned() throws IOException {
    URL url = new URL(canonicalUrl);
    HttpURLConnection connection = (HttpURLConnection) url.openConnection();
    connection.setInstanceFollowRedirects(true);
    int status = connection.getResponseCode();
    String redirectedUrl = null;
    if (status == HttpURLConnection.HTTP_MOVED_PERM || status == HttpURLConnection.HTTP_MOVED_TEMP) {
        redirectedUrl = connection.getHeaderField("Location");
    }
    connection.disconnect();
    assertEquals(expectedRedirectedUrl, redirectedUrl);
}
```

在这里，我们定义原始URL字符串(canonicalUrl)和重定向URL(expectedRedirectedUrl)，然后，我们创建一个HttpURLConnection对象并打开到原始URL的连接。

之后，我们将instanceFollowRedirects属性设置为true以启用自动重定向。**收到响应后，我们检查状态代码以确定是否发生重定向，如果发生重定向，我们从“Location”标头中检索重定向的URL**。

最后，我们断开连接并断言检索到的重定向URL与预期的URL匹配。

## 3. 使用Apache HttpClient

或者，我们可以使用Apache [HttpClient](https://www.baeldung.com/httpclient-guide)，这是一个更通用的Java HTTP客户端库。Apache HttpClient提供了更高级的功能，并为处理HTTP请求和响应提供了更好的支持。以下是我们如何使用Apache HttpClient找到重定向的URL：

```java
@Test
public void givenOriginalUrl_whenFindRedirectUrlUsingHttpClient_thenCorrectRedirectedUrlReturned() throws IOException {
    RequestConfig config = RequestConfig.custom()
        .setRedirectsEnabled(false)
        .build();
    try (CloseableHttpClient httpClient = HttpClients.custom().setDefaultRequestConfig(config).build()) {
        HttpGet httpGet = new HttpGet(canonicalUrl);
        try (CloseableHttpResponse response = httpClient.execute(httpGet)) {
            int statusCode = response.getStatusLine().getStatusCode();
            String redirectedUrl = null;
            if (statusCode == HttpURLConnection.HTTP_MOVED_PERM || statusCode == HttpURLConnection.HTTP_MOVED_TEMP) {
                org.apache.http.Header[] headers = response.getHeaders("Location");
                if (headers.length > 0) {
                    redirectedUrl = headers[0].getValue();
                }
            }
            assertEquals(expectedRedirectedUrl, redirectedUrl);
        }
    }
}
```

在这个测试中，我们首先配置请求以停用自动重定向。随后，我们实例化CloseableHttpClient并向原始URL发送HttpGet请求。

获得响应后，我们会分析状态代码和标头以确定是否发生了重定向。如果发生重定向，我们会从Location标头中提取重定向的URL。

## 4. 总结

在本文中，我们探讨了如何使用HttpURLConnection和Apache HttpClient查找给定URL的重定向URL。