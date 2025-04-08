---
layout: post
title:  在JAX-RS中设置响应正文
category: webmodules
copyright: webmodules
excerpt: Jersey
---

## 1. 概述

为了简化使用Java开发REST Web服务及其客户端，我们设计了一种标准且可移植的JAX-RS API实现，称为Jersey。

Jersey是一个用于开发REST Web服务的开源框架，它提供对JAX-RS API的支持并作为JAX-RS的参考实现。

在本教程中，我们将研究如何设置具有不同媒体类型的Jersey响应主体。

## 2. Maven依赖

首先，我们需要在pom.xml文件中包含以下依赖：

```xml
<dependency>
    <groupId>org.glassfish.jersey.bundles</groupId>
    <artifactId>jaxrs-ri</artifactId>
    <version>3.1.1</version>
</dependency>
<dependency>
    <groupId>org.glassfish.jersey.core</groupId>
    <artifactId>jersey-server</artifactId>
    <version>3.1.1</version>
</dependency>
```

最新版本的JAX-RS可以在[jaxrs-ri](https://mvnrepository.com/artifact/org.glassfish.jersey.bundles/jaxrs-ri)找到，Jersey服务器可以在[jersey-server](https://mvnrepository.com/artifact/org.glassfish.jersey.core/jersey-server)找到。

## 3. Jersey中的响应

当然，有多种使用Jersey构建响应的方法，我们将在下面研究如何构建它们。

这里的所有示例都是HTTP GET请求，我们将使用curl命令来测试资源。

### 3.1 文本响应

此处显示的端点是如何将纯文本作为Jersey响应返回的简单示例：

```java
@GET
@Path("/ok")
public Response getOkResponse() {

    String message = "This is a text response";

    return Response
            .status(Response.Status.OK)
            .entity(message)
            .build();
}
```

我们可以使用curl执行HTTP GET来验证响应：

```shell
curl -XGET http://localhost:8080/jersey/response/ok
```

该端点将发回如下响应：

```text
This is a text response
```

当未指定媒体类型时，Jersey将默认为text/plain。

### 3.2 错误响应

Jersey也可以将错误作为响应发送：

```java
@GET
@Path("/not_ok")
public Response getNOkTextResponse() {

    String message = "There was an internal server error";

    return Response
            .status(Response.Status.INTERNAL_SERVER_ERROR)
            .entity(message)
            .build();
}
```

为了验证响应，我们可以使用curl发出HTTP GET请求：

```shell
curl -XGET http://localhost:8080/jersey/response/not_ok
```

错误消息将在响应中显示：

```text
There was an internal server error
```

### 3.3 纯文本响应

我们还可以返回简单的纯文本响应：

```java
@GET
@Path("/text_plain")
public Response getTextResponseTypeDefined() {

    String message = "This is a plain text response";

    return Response
            .status(Response.Status.OK)
            .entity(message)
            .type(MediaType.TEXT_PLAIN)
            .build();
}
```

再次，我们可以使用curl执行HTTP GET来验证响应：

```shell
curl -XGET http://localhost:8080/jersey/response/text_plain
```

响应如下：

```text
This is a plain text response
```

也可以通过@Produces注解而不是使用Response中的type()方法来实现相同的结果：

```java
@GET
@Path("/text_plain_annotation")
@Produces({ MediaType.TEXT_PLAIN })
public Response getTextResponseTypeAnnotated() {

    String message = "This is a plain text response via annotation";

    return Response
            .status(Response.Status.OK)
            .entity(message)
            .build();
}
```

我们可以使用curl进行响应验证：

```shell
curl -XGET http://localhost:8080/jersey/response/text_plain_annotation
```

以下是响应：

```text
This is a plain text response via annotation
```

### 3.4 使用POJO的JSON响应

**简单的POJO也可用于构建Jersey响应**。

我们有一个非常简单的Person POJO，如下所示，我们将使用它来构建响应：

```java
public class Person {
    String name;
    String address;

    // standard constructor
    // standard getters and setters
}
```

Person POJO现在可以用来**返回JSON作为响应主体**：

```java
@GET
@Path("/pojo")
public Response getPojoResponse() {

    Person person = new Person("Abhinayak", "Nepal");

    return Response
            .status(Response.Status.OK)
            .entity(person)
            .build();
}
```

可以通过以下curl命令验证此GET端点的响应：

```shell
curl -XGET http://localhost:8080/jersey/response/pojo
```

Person POJO将被转换为JSON并作为响应发回：

```json
{"address":"Nepal","name":"Abhinayak"}
```

### 3.5 使用简单字符串的JSON响应

我们可以使用预格式化的字符串来创建响应，而且可以简单地完成。

以下端点是如何将以字符串形式表示的JSON作为JSON发送回Jersey响应的示例：

```java
@GET
@Path("/json")
public Response getJsonResponse() {

    String message = "{\"hello\": \"This is a JSON response\"}";

    return Response
            .status(Response.Status.OK)
            .entity(message)
            .type(MediaType.APPLICATION_JSON)
            .build();
}
```

可以通过使用curl执行HTTP GET来验证响应：

```shell
curl -XGET http://localhost:8080/jersey/response/json
```

调用此资源将返回一个JSON：

```json
{"hello":"This is a JSON response"}
```

**同样的模式也适用于其他常见的媒体类型，如XML或HTML**，我们只需要使用MediaType.TEXT_XML或MediaType.TEXT_HTML通知Jersey它是一个XML或HTML，Jersey将处理其余的事情。


## 4. 总结

在这篇简短的文章中，我们为各种媒体类型构建了Jersey(JAX-RS)响应。