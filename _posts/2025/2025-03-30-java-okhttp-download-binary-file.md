---
layout: post
title:  使用OkHttp下载二进制文件
category: libraries
copyright: libraries
excerpt: OkHttp
---

## 1. 概述

本教程将给出一个实际示例，说明如何**使用[OkHttp](https://www.baeldung.com/guide-to-okhttp)下载二进制文件**。

## 2. Maven依赖

我们首先添加基础库[okhttp](https://mvnrepository.com/artifact/com.squareup.okhttp3/okhttp)依赖：

```xml
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>5.0.0-alpha.12</version>
</dependency>
```

然后，**如果我们想为使用OkHttp库实现的模块编写集成测试，我们可以使用[mockwebserver](https://mvnrepository.com/artifact/com.squareup.okhttp3/mockwebserver)库，该库具有Mock服务器及其响应的工具**：

```xml
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>mockwebserver</artifactId>
    <version>5.0.0-alpha.12</version>
    <scope>test</scope>
</dependency>
```

## 3. 请求二进制文件

我们首先实现一个类，该类接收下载文件的URL作为参数，并为该URL创建并执行HTTP请求。

为了使该类可测试，我们将在构造函数中**注入OkHttpClient和writer**：

```java
public class BinaryFileDownloader implements AutoCloseable {

    private final OkHttpClient client;
    private final BinaryFileWriter writer;

    public BinaryFileDownloader(OkHttpClient client, BinaryFileWriter writer) {
        this.client = client;
        this.writer = writer;
    }
}
```

接下来，我们将实现从URL下载文件的方法：

```java
public long download(String url) throws IOException {
    Request request = new Request.Builder().url(url).build();
    Response response = client.newCall(request).execute();
    ResponseBody responseBody = response.body();
    if (responseBody == null) {
        throw new IllegalStateException("Response doesn't contain a file");
    }
    double length = Double.parseDouble(Objects.requireNonNull(response.header(CONTENT_LENGTH, "1")));
    return writer.write(responseBody.byteStream(), length);
}
```

下载文件的过程分为四个步骤：使用URL创建请求、执行请求并接收响应、获取响应的主体，如果为空则失败、将响应主体的字节写入文件。

## 4. 将响应写入本地文件

为了将从响应中接收到的字节写入本地文件，我们将实现一个BinaryFileWriter类，**该类以InputStream和OutputStream作为输入，并将内容从[InputStream](https://www.baeldung.com/convert-input-stream-to-a-file)复制到OutputStream**。

OutputStream将被注入到构造函数中，以便该类可测试：

```java
public class BinaryFileWriter implements AutoCloseable {

    private final OutputStream outputStream;

    public BinaryFileWriter(OutputStream outputStream) {
        this.outputStream = outputStream;
    }
}
```

现在我们将实现将内容从InputStream复制到OutputStream的方法，该方法首先用BufferedInputStream包装InputStream，以便我们可以一次读取更多字节。然后我们准备一个数据缓冲区，在其中临时存储来自InputStream的字节。

最后，我们将缓冲的数据写入OutputStream。只要InputStream有数据要读取，我们就会这样做：

```java
public long write(InputStream inputStream) throws IOException {
    try (BufferedInputStream input = new BufferedInputStream(inputStream)) {
        byte[] dataBuffer = new byte[CHUNK_SIZE];
        int readBytes;
        long totalBytes = 0;
        while ((readBytes = input.read(dataBuffer)) != -1) {
            totalBytes += readBytes;
            outputStream.write(dataBuffer, 0, readBytes);
        }
        return totalBytes;
    }
}
```

## 5. 获取文件下载进度

在某些情况下，我们可能想告诉用户文件下载的进度。

我们首先需要创建一个[函数接口](https://www.baeldung.com/java-8-functional-interfaces)：

```java
public interface ProgressCallback {
    void onProgress(double progress);
}
```

然后，我们将在BinaryFileWriter类中使用它，这将在每一步中为我们提供下载器迄今为止写入的总字节数。

首先，我们**将ProgressCallback作为字段添加到writer类**。然后，我们将更新write方法以接收响应的长度作为参数，这将帮助我们计算进度。

然后，我们将使用根据迄今为止写入的totalBytes和length计算出的进度来调用onProgress方法：

```java
public class BinaryFileWriter implements AutoCloseable {
    private final ProgressCallback progressCallback;
    public long write(InputStream inputStream, double length) {
        // ...
        progressCallback.onProgress(totalBytes / length * 100.0);
    }
}
```

最后，我们将更新BinaryFileDownloader类，以使用总响应长度调用write方法。我们将从Content-Length标头中获取响应长度，然后将其传递给write方法：

```java
public class BinaryFileDownloader {
    public long download(String url) {
        double length = getResponseLength(response);
        return write(responseBody, length);
    }
    private double getResponseLength(Response response) {
        return Double.parseDouble(Objects.requireNonNull(response.header(CONTENT_LENGTH, "1")));
    }
}
```

## 6. 总结

在本文中，我们使用OkHttp库实现了从URL下载二进制文件的简单而实用的示例。