---
layout: post
title:  解码OkHttp JSON响应
category: libraries
copyright: libraries
excerpt: JavaParser
---

## 1. 简介

在本教程中，我们将探讨使用[OkHttp](https://www.baeldung.com/guide-to-okhttp)解码JSON响应的几种技术。

## 2. OkHttp响应

OkHttp是适用于Java和Android的HTTP客户端，具有透明处理GZIP、响应缓存和网络问题恢复等功能。

尽管有这些很棒的功能，OkHttp并没有内置JSON、XML和其他内容类型的编码器/解码器。但是，我们可以借助XML/JSON绑定库来实现这些功能，或者我们可以使用[Feign](https://www.baeldung.com/intro-to-feign)或[Retrofit](https://www.baeldung.com/retrofit)等高级库。

要实现JSON解码器，我们需要从服务调用的结果中提取JSON。为此，我们可以通过Response对象的body()方法访问主体，ResponseBody类有几种提取此数据的选项：

- **byteStream()**：将主体的原始字节作为InputStream公开；我们可以将其用于所有格式，但通常用于二进制文件和文件。
- **charStream()**：当我们有文本响应时，charStream()将其InputStream包装在Reader中，并根据响应的内容类型处理编码，如果响应标头中未设置字符集，则处理“UTF-8”；但是，当使用charStream()时，我们无法更改Reader的编码。
- **string()**：以String形式返回整个响应主体；与charStream()一样管理编码，但如果我们需要不同的编码，我们可以改用source().readString(charset)。

在本文中，我们将使用string()，因为我们的响应很小，并且我们不存在内存或性能问题。当性能和内存很重要时，byteStream()和charStream()方法是生产系统中更好的选择。

首先，让我们将[okhttp](https://mvnrepository.com/artifact/com.squareup.okhttp3/okhttp)添加到pom.xml文件中：

```xml
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId> 
    <version>5.0.0-alpha.12</version> 
</dependency>
```

然后，我们对SimpleEntity进行建模来测试我们的解码器：

```java
public class SimpleEntity {
    protected String name;

    public SimpleEntity(String name) {
        this.name = name;
    }
    
    // no-arg constructor, getters, and setters
}
```

现在，我们将开始测试：

```java
SimpleEntity sampleResponse = new SimpleEntity("Tuyucheng");

OkHttpClient client = // build an instance;
MockWebServer server = // build an instance;
Request request = new Request.Builder().url(server.url("...")).build();
```

## 3. 使用Jackson解码ResponseBody

[Jackson](https://www.baeldung.com/jackson)是最流行的JSON对象绑定库之一。

让我们将[jackson-databind](https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind)添加到pom.xml中：

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.17.2</version>
</dependency>
```

Jackson的ObjectMapper允许我们将JSON转换为对象。因此，我们可以使用ObjectMapper.readValue()解码响应：

```java
ObjectMapper objectMapper = new ObjectMapper(); 
ResponseBody responseBody = client.newCall(request).execute().body(); 
SimpleEntity entity = objectMapper.readValue(responseBody.string(), SimpleEntity.class);

Assert.assertNotNull(entity);
Assert.assertEquals(sampleResponse.getName(), entity.getName());
```

## 4. 使用Gson解码ResponseBody

[Gson](https://www.baeldung.com/gson-deserialization-guide)是另一个用于将JSON映射到对象的有用库。

让我们将[gson](https://mvnrepository.com/artifact/com.google.code.gson/gson)添加到pom.xml文件中：

```xml
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.10.1</version>
</dependency>
```

让我们看看如何使用Gson.fromJson()来解码响应主体：

```java
Gson gson = new Gson(); 
ResponseBody responseBody = client.newCall(request).execute().body();
SimpleEntity entity = gson.fromJson(responseBody.string(), SimpleEntity.class);

Assert.assertNotNull(entity);
Assert.assertEquals(sampleResponse.getName(), entity.getName());

```

## 5. 总结

在本文中，我们探讨了使用Jackson和Gson解码OkHttp的JSON响应的几种方法。