---
layout: post
title:  在Jersey中添加List作为查询参数
category: webmodules
copyright: webmodules
excerpt: Jersey
---

## 1. 概述

[Jersey](https://eclipse-ee4j.github.io/jersey/)是一个用于开发[RESTful Web服务](https://www.baeldung.com/jersey-rest-api-with-spring)的开源框架，它是[JAX-RS](https://www.baeldung.com/jax-rs-spec-and-implementations)的参考实现。

在本教程中，我们将探讨使用Jersey客户端发出请求时将List添加为查询参数的不同方法。

## 2. GET API在查询参数中接收列表

我们首先创建一个GET API，它接收[查询参数](https://www.baeldung.com/jersey-request-parameters)中的列表。

**我们可以使用@QueryParam注解从[URI](https://www.baeldung.com/cs/uniform-resource-identifiers)中的查询参数中提取值**，@QueryParam注解接收单个参数，即我们要提取的查询参数的名称。

要使用@QueryParam指定列表类型查询参数，我们将注解应用于方法参数，表明它从URL中的查询参数接收值列表：

```java
@Path("/")
public class JerseyListDemo {
    @GET
    public String getItems(@QueryParam("items") List<String> items) {
        return "Received items: " + items;
    }
}
```

现在，我们将使用不同的方法将列表作为查询参数传递。完成后，我们将验证响应以确保资源正确处理元素列表。

## 3. 使用queryParam()

Jersey中的queryParam()方法在构造[HTTP请求](https://www.baeldung.com/java-http-request)时将查询参数添加到URL，**queryParam()方法允许我们指定查询参数的名称和值**。

### 3.1 直接使用查询参数

在这种方法中，我们使用Jersey提供的方法直接添加查询参数。

在下面的例子中，我们有一个WebTarget作为target()，并且我们向请求URL添加一个具有多个值item1、item2的查询参数items：

```java
@Test
public void givenList_whenUsingQueryParam_thenPassParamsAsList() {
    Response response = target("/")
            .queryParam("items", "item1", "item2")
            .request
            .get();
    assertEquals(Response.Status.OK.getStatusCode(), response.getStatus());
    assertEquals("Received items: [item1, item2]", response.readEntity(String.class));
}
```

这会产生一个包含查询参数的URL，例如/?items=item1&items=item2。此处，items查询参数包含item1和item2作为其值。

### 3.2 使用逗号分隔的字符串

在这种方法中，我们[将列表转换为逗号分隔的字符串](https://www.baeldung.com/java-list-comma-separated-string)，然后将其作为查询参数添加到Jersey客户端中。它简化了URL构造过程，但需要服务器端逻辑将字符串解析为列表：

```java
@Test
public void givenList_whenUsingCommaSeparatedString_thenPassParamsAsList() {
    Response response = target("/")
            .queryParam("items", "item1,item2")
            .request()
            .get();
    assertEquals(Response.Status.OK.getStatusCode(), response.getStatus());
    assertEquals("Received items: [item1,item2]", response.readEntity(String.class));
}
```

这会产生一个带有查询参数的URL，如/?items=item1,item2。此处，items查询参数包含item1,item2作为其值。

## 4. 使用UriBuilder

UriBuilder方法是使用查询参数[构建URL](https://www.baeldung.com/java-url#creating-a-url)的有效方法，在此方法中，我们创建一个UriBuilder实例，指定基本URI，并循环添加查询参数：

```java
@Test
public void givenList_whenUsingUriBuilder_thenPassParamsAsList() {
    List<String> itemsList = Arrays.asList("item1","item2");
    UriBuilder builder = UriBuilder.fromUri("/");
    for (String item : itemsList) {
        builder.queryParam("items", item);
    }
    URI uri = builder.build();
    String expectedUri = "/?items=item1&items=item2";
    assertEquals(expectedUri, uri.toString());
}
```

[单元测试](https://www.baeldung.com/junit)确保UriBuilder正确地使用所需的查询参数组装URL，并验证其准确性。

## 5. 总结

在本文中，我们学习了在Jersey中将列表作为查询参数传递的不同方法。

**queryParam()方法很简单，适合用于简单情况。另一方面，UriBuilder非常适合用于生成具有多个查询参数的动态URL**。选择取决于应用程序的需求，并考虑列表复杂性和动态URL构建的必要性等因素。