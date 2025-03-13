---
layout: post
title:  使用Java中的URLConnection查找Web文件的大小
category: java-net
copyright: java-net
excerpt: Java Network
---

## 1. 概述

[HTTP协议](https://www.baeldung.com/java-9-http-client)提供了有关所请求Web资源的全面信息，其标头字段之一Content-Length指定资源的大小(以字节为单位)，我们可以使用[URLConnection](https://www.baeldung.com/java-custom-url-connection)类提取此信息。

在下载之前了解网页文件的大小有助于估计下载资源所需的网络数据量。

在本教程中，我们将探讨如何使用URLConnection类的getContentLengthLong()和getHeaderField()方法获取Web文件的大小。

## 2. HTTP Content-Length

Content-Length属性在HTTP标头中指定Web文件的大小，由于HTTP是一种传输协议，因此它提供了传入响应的详细信息。**简而言之，Content-Length字段表示响应主体的大小(以字节为单位)**。

此外，可以指定Transfer-Encoding字段并将其设置为chunked，而不是Content-Length字段。在这种情况下，我们无法确定Web文件的大小，因为下载是分块进行的。

**值得注意的是，Content-Length标头并不总是准确的**，服务器可以将此字段设置为任意值，这可能并不代表真实的文件大小。例如，在Spring应用程序中使用[ResponseEntity](https://www.baeldung.com/spring-response-entity)时，开发人员可以将标头字段设置为任意值，这可能与响应主体的实际大小不符。

## 3. getContentLength()和getContentLengthLong()方法

Java中的URLConnection类提供了与URL资源建立连接以进行写入或读取操作的方法。

它提供了一个名为getContentLength()的方法，该方法以整数形式返回HTTP标头Content-Length字段。但是，**此方法无法表示大于Integer.MAX_VALUE的数字，这意味着它无法处理大于2GiB的文件大小**。

为了解决getContentLength()方法的限制，URLConnection类提供了getContentLengthLong()方法，该方法将内容长度返回为long值。**此方法是首选方法，因为它可以检索超过Integer.MAX_VALUE的大文件大小**。

值得注意的是，当HTTP标头中缺少Content-Length字段时，getContentLength()和getContentLengthLong()方法将返回-1。

## 4. 使用getContentLengthLong()方法

让我们看一个从“https://www.ingka.com/wp-content/uploads/2020/11/dummy.pdf”检索虚拟PDF文件大小的示例。

首先，让我们定义一个代表Web文件的URL的[URL](https://www.baeldung.com/java-url)实例：

```java
String fileUrl = "https://www.ingka.com/wp-content/uploads/2020/11/dummy.pdf";
URL url = new URL(fileUrl);
```

接下来，让我们创建一个URLConnection对象并打开到此URL的连接：

```java
URLConnection urlConnection = url.openConnection();
```

然后我们来获取一下网页文件的大小：

```java
long fileSize = urlConnection.getContentLengthLong();
if (fileSize != -1) {
    assertEquals(29789, fileSize);
} else {
    fail("Could not determine file size");
}
```

在上面的代码中，我们调用urlConnection对象上的getContentLengthLong()方法来获取估计的文件大小。然后，我们断言估计的文件大小等于预期大小。

另外，我们还处理了无法确定Web文件大小的情况。

## 5. 使用getHeaderField()方法

或者，**我们可以使用getHeaderField()方法来检索Web文件大小**，该方法返回它作为参数接收的名称字段值。

以下是使用getHeaderField()方法的示例：

```java
@Test
void givenUrl_whenGetFileSizeUsingURLConnectionAndGetHeaderField_thenCorrect() throws IOException {
    URL url = new URL(fileUrl);
    URLConnection urlConnection = url.openConnection();

    String headerField = urlConnection.getHeaderField("Content-Length");
    if (headerField != null && !headerField.isEmpty()) {
        long fileSize = Long.parseLong(headerField);
        assertEquals(29789, fileSize);
    } else {
        fail("Could not determine file size");
    }
}
```

在上面的代码中，我们在URLConnection对象上调用getHeaderField()方法并指定Content-Length标头字段。由于该方法返回一个字符串，我们将其值解析为long类型，并断言返回大小等于预期大小。

## 6. 总结

在本教程中，我们了解了如何使用URLConnection.getContentLengthLong()和URLConnection.getHeaderField()方法来检索已知Web文件的大小，我们利用HTTP标头的Content-Length字段来确定Web文件的大小。当无法确定文件大小时，它会返回-1值。