---
layout: post
title:  使用Apache HttpClient进行分段上传
category: apache
copyright: apache
excerpt: Apache HttpClient
---

## 1. 概述

在本教程中，我们将说明如何使用HttpClient 5执行分段上传操作。

如果你想深入了解使用HttpClient可以做的其他有趣的事情，请转到[主要的HttpClient教程](https://www.baeldung.com/httpclient-guide)。

## 2. 使用AddPart方法

让我们首先查看MultipartEntityBuilder对象，它将各个部分添加到Http实体，然后通过POST操作上传。

这是一种向表示表单的HttpEntity添加部分的通用方法。

### 2.1 上传包含两个文本部分和一个文件的表单

```java
final File file = new File(url.getPath());
final FileBody fileBody = new FileBody(file, ContentType.DEFAULT_BINARY);
final StringBody stringBody1 = new StringBody("This is message 1", ContentType.MULTIPART_FORM_DATA);
final StringBody stringBody2 = new StringBody("This is message 2", ContentType.MULTIPART_FORM_DATA);

final MultipartEntityBuilder builder = MultipartEntityBuilder.create();
builder.setMode(HttpMultipartMode.LEGACY);
builder.addPart("file", fileBody);
builder.addPart("text1", stringBody1);
builder.addPart("text2", stringBody2);
final HttpEntity entity = builder.build();

post.setEntity(entity);
try(CloseableHttpClient client = HttpClientBuilder.create()
    .build()) {

    client.execute(post, response -> { 
       //do something with response
    });
}
```

请注意，我们还通过指定服务器要使用的ContentType值来实例化File对象。

另外，请注意，addPart方法有两个参数，相当于表单的键值对。只有当服务器端确实需要并使用参数名时，这些参数才有意义，否则将被忽略。

## 3. 使用addBinaryBody和addTextBody方法

创建多部分实体的更直接方法是使用addBinaryBody和AddTextBody方法，这些方法适用于上传文本、文件、字符数组和InputStream对象，让我们通过简单示例来说明如何使用。

### 3.1 上传文本和文本文件部分

```java
final File file = new File(url.getPath());
final String message = "This is a multipart post";
final MultipartEntityBuilder builder = MultipartEntityBuilder.create();
builder.setMode(HttpMultipartMode.LEGACY);
builder.addBinaryBody("file", file, ContentType.DEFAULT_BINARY, TEXTFILENAME);
builder.addTextBody("text", message, ContentType.DEFAULT_BINARY);
final HttpEntity entity = builder.build();
post.setEntity(entity);

try(CloseableHttpClient client = HttpClientBuilder.create()
    .build()) {

    client.execute(post, response -> { 
       //do something with response
    });
}
```

请注意，这里不需要FileBody和StringBody对象。

同样重要的是，大多数服务器不检查文本正文的ContentType，因此addTextBody方法可能会省略ContentType值。

addBinaryBody API接收ContentType参数，但也可以仅通过二进制主体和包含文件的表单参数名称来创建实体。如上一节所述，如果未指定ContentType值，某些服务器将无法识别该文件。

接下来，我们将添加一个zip文件作为InputStream，同时将图像文件添加为File对象。

### 3.2 上传Zip文件、图片文件和文本部分

```java
final URL url = Thread.currentThread()
    .getContextClassLoader()
    .getResource("uploads/" + ZIPFILENAME);
final URL url2 = Thread.currentThread()
    .getContextClassLoader()
    .getResource("uploads/" + IMAGEFILENAME);
final InputStream inputStream = new FileInputStream(url.getPath());
final File file = new File(url2.getPath());
final String message = "This is a multipart post";
final MultipartEntityBuilder builder = MultipartEntityBuilder.create();
builder.setMode(HttpMultipartMode.LEGACY);
builder.addBinaryBody("file", file, ContentType.DEFAULT_BINARY, IMAGEFILENAME);
builder.addBinaryBody("upstream", inputStream, ContentType.create("application/zip"), ZIPFILENAME);
builder.addTextBody("text", message, ContentType.TEXT_PLAIN);
final HttpEntity entity = builder.build();
post.setEntity(entity);

try(CloseableHttpClient client = HttpClientBuilder.create()
    .build()) {

    client.execute(post, response -> { 
       //do something with response
    });
}
```

请注意，ContentType值可以动态创建，就像上面的zip文件示例一样。

最后，并非所有服务器都识别InputStream部分，我们在代码第一行实例化的服务器可以识别InputStream。

现在让我们看另一个示例，其中addBinaryBody直接使用字节数组。

### 3.3 上传字节数组和文本

```java
final String message = "This is a multipart post";
final byte[] bytes = "binary code".getBytes();
final MultipartEntityBuilder builder = MultipartEntityBuilder.create();
builder.setMode(HttpMultipartMode.LEGACY);
builder.addBinaryBody("file", bytes, ContentType.DEFAULT_BINARY, TEXTFILENAME);
builder.addTextBody("text", message, ContentType.TEXT_PLAIN);
final HttpEntity entity = builder.build();
post.setEntity(entity);

try(CloseableHttpClient client = HttpClientBuilder.create()
    .build()) {

    client.execute(post, response -> { 
       //do something with response
    });
}
```

注意ContentType，它现在指定二进制数据。

## 4. 总结

本文介绍了MultipartEntityBuilder作为一个灵活的对象，它提供了多种API选择来创建多部分表单。

示例还展示了如何使用HttpClient上传类似于表单实体的HttpEntity。