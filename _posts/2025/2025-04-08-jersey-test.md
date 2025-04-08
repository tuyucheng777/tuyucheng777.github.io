---
layout: post
title:  探索Jersey测试框架
category: webmodules
copyright: webmodules
excerpt: Jersey
---

## 1. 概述

在本教程中，我们将介绍Jersey测试框架并了解如何使用它来快速编写集成测试。

正如我们在之前的文章中看到的，**[Jersey](https://jersey.github.io/)是一个用于开发RESTful Web服务的开源框架**。

## 2. 应用程序设置

Jersey测试框架是一个帮助我们验证服务器端组件是否正确实现的工具，正如我们稍后将看到的，**它提供了一种快速且轻松的集成测试编写方法**，并且可以很好地处理与我们的HTTP API的通信。

**同样，它几乎开箱即用，并且易于与我们基于Maven的项目集成**。该框架主要基于JUnit，但也可以与TestNG一起使用，这使得它几乎可以在所有环境中使用。

在下一节中，我们将看到要使用该框架需要向应用程序添加哪些依赖。

### 2.1 Maven依赖

首先，让我们将Jersey测试框架核心依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>org.glassfish.jersey.test-framework</groupId>
    <artifactId>jersey-test-framework-core</artifactId>
    <version>3.1.1</version>
    <scope>test</scope>
</dependency>
```

与往常一样，可以从[Maven Central](https://mvnrepository.com/artifact/org.glassfish.jersey.test-framework/jersey-test-framework-core)获取最新版本。

几乎所有的Jersey测试都使用事实上的Grizzly测试容器工厂，我们还需要添加：

```xml
<dependency>
    <groupId>org.glassfish.jersey.test-framework.providers</groupId>
    <artifactId>jersey-test-framework-provider-grizzly2</artifactId>
    <version>3.1.1</version>
    <scope>test</scope>
</dependency>
```

可以在[Maven Central](https://mvnrepository.com/artifact/org.glassfish.jersey.test-framework.providers/jersey-test-framework-provider-grizzly2)中找到最新版本。

## 3. 入门

在下一部分中，我们将介绍编写简单测试所需的基本步骤。

我们首先在服务器上测试简单的问候资源：

```java
@Path("/greetings")
public class Greetings {

    @GET
    @Path("/hi")
    public String getHiGreeting() {
        return "hi";
    }
}
```

### 3.1 配置测试

现在让我们定义测试类：

```java
public class GreetingsResourceIntegrationTest extends JerseyTest {

    @Override
    protected Application configure() {
        return new ResourceConfig(Greetings.class);
    }
    // ...
}
```

**从上面的例子中我们可以看出，要使用Jersey测试框架开发测试，我们的测试需要子类化JerseyTest**。

接下来，我们重写configure方法，该方法返回用于测试的自定义资源配置，并且仅包含Greetings资源。当然，这是我们想要测试的资源。

### 3.2 编写第一个测试

让我们首先测试一下Greetings API中的简单GET请求：

```java
@Test
public void givenGetHiGreeting_whenCorrectRequest_thenResponseIsOkAndContainsHi() {
    Response response = target("/greetings/hi").request()
        .get();

    assertEquals("Http Response should be 200: ", Status.OK.getStatusCode(), response.getStatus());
    assertEquals("Http Content-Type should be: ", MediaType.TEXT_HTML, response.getHeaderString(HttpHeaders.CONTENT_TYPE));

    String content = response.readEntity(String.class);
    assertEquals("Content of response is: ", "hi", content);
}
```

请注意，我们对HTTP响应具有完全访问权限-**因此我们可以执行诸如检查状态码之类的操作以确保操作确实成功，或者处理响应的实际主体**。

让我们更详细地解释一下上面例子中我们所做的事情：

1. 向'/greetings/hi'发送HTTP GET请求
2. 检查HTTP状态码和Content-Type响应标头
3. 测试响应内容是否包含字符串“hi”

## 4. 测试GET来检索资源

现在，我们已经了解了创建测试的基本步骤，让我们来测试一下在[Jersey MVC支持文章](https://www.baeldung.com/jersey-mvc)中介绍的简单Fruit API。

### 4.1 获取纯JSON

在下面的例子中，我们将响应主体作为标准JSON字符串：

```java
@Test
public void givenFruitExists_whenSearching_thenResponseContainsFruit() {
    final String json = target("fruit/search/strawberry").request()
        .get(String.class);
    assertThat(json, containsString("{\"name\":\"strawberry\",\"weight\":20}"));
}
```

### 4.2 获取实体而不是JSON

我们还可以将响应直接映射到资源实体类-例如：

```java
@Test
public void givenFruitExists_whenSearching_thenResponseContainsFruitEntity() {
    final Fruit entity = target("fruit/search/strawberry").request()
            .get(Fruit.class);

    assertEquals("Fruit name: ", "strawberry", entity.getName());
    assertEquals("Fruit weight: ", 20, entity.getWeight());
}
```

这次，我们在get方法中指定响应实体将转换为的Java类型-Fruit对象。

## 5. 测试POST来创建资源

为了在我们的API中创建新资源，我们将充分利用POST请求。在下一节中，我们将了解如何测试API的这一部分。

### 5.1 发布纯JSON

让我们首先发布一个简单的JSON字符串来测试新水果资源的创建：

```java
@Test
public void givenCreateFruit_whenJsonIsCorrect_thenResponseCodeIsCreated() {
    Response response = target("fruit/created").request()
            .post(Entity.json("{\"name\":\"strawberry\",\"weight\":20}"));

    assertEquals("Http Response should be 201 ", Status.CREATED.getStatusCode(), response.getStatus());
    assertThat(response.readEntity(String.class), containsString("Fruit saved : Fruit [name: strawberry colour: null]"));
}
```

在上面的例子中，我们使用了接收Entity对象参数的post方法，**我们使用便捷的json方法从相应的JSON字符串创建一个实体**。

### 5.2 发布实体而不是JSON

**正如我们已经看到的，通过获取请求，我们也可以直接发布资源实体类**-例如：

```java
@Test
public void givenCreateFruit_whenFruitIsInvalid_thenResponseCodeIsBadRequest() {
    Fruit fruit = new Fruit("Blueberry", "purple");
    fruit.setWeight(1);

    Response response = target("fruit/create").request(MediaType.APPLICATION_JSON_TYPE)
        .post(Entity.entity(fruit, MediaType.APPLICATION_JSON_TYPE));

    assertEquals("Http Response should be 400 ", 400, response.getStatus());
    assertThat(response.readEntity(String.class), containsString("Fruit weight must be 10 or greater"));
}
```

这次我们使用entity方法来发布我们的Fruit实体，并将媒体类型指定为JSON。

### 5.3 使用POST提交表单

在最后一个POST示例中，我们将看到如何通过POST请求测试表单提交：

```java
@Test
public void givenCreateFruit_whenFormContainsNullParam_thenResponseCodeIsBadRequest() {
    Form form = new Form();
    form.param("name", "apple");
    form.param("colour", null);
    
    Response response = target("fruit/create").request(MediaType.APPLICATION_FORM_URLENCODED)
        .post(Entity.form(form));

    assertEquals("Http Response should be 400 ", 400, response.getStatus());
    assertThat(response.readEntity(String.class), containsString("Fruit colour must not be null"));
 }
```

类似地，我们使用Entity类，但这次将包含许多参数的表单传递给我们的POST请求。

## 6. 测试其他HTTP动词

有时我们需要测试其他HTTP端点，例如PUT和DELETE；当然，**使用Jersey测试框架完全可以实现这一点**。

让我们看一个简单的PUT示例：

```java
@Test
public void givenUpdateFruit_whenFormContainsBadSerialParam_thenResponseCodeIsBadRequest() {
    Form form = new Form();
    form.param("serial", "2345-2345");

    Response response = target("fruit/update").request(MediaType.APPLICATION_FORM_URLENCODED)
        .put(Entity.form(form));

    assertEquals("Http Response should be 400 ", 400, response.getStatus());
    assertThat(response.readEntity(String.class), containsString("Fruit serial number is not valid"));
}
```

一旦我们调用了request方法，我们就可以调用当前请求对象上的任何HTTP方法。

## 7. 附加功能

**Jersey测试框架包含许多额外的配置属性，可以帮助调试和测试**。

在下一个示例中，我们将看到如何以编程方式启用具有给定名称的功能：

```java
public class FruitResourceIntegrationTest extends JerseyTest {

    @Override
    protected Application configure() {
        enable(TestProperties.LOG_TRAFFIC);
        enable(TestProperties.DUMP_ENTITY);
        // ...
```

当我们创建和配置正在测试的Jersey应用程序时，我们还可以启用其他属性。在本例中，我们启用了两个日志记录属性-LOG_TRAFFIC和DUMP_ENTITY，**它们将在测试运行期间提供有用的其他日志记录和调试信息**。

## 8. 支持的容器

正如我们已经提到的，使用Jersey测试框架编写测试时使用的实际容器是Grizzly；但是，还支持许多其他容器：

- 内存容器
- Oracle JDK中的HttpServer
- 简单容器(org.simpleframework.http)
- Jetty容器(org.eclipse.jetty)

有关如何配置这些容器的更多信息，请参阅[此处](https://eclipse-ee4j.github.io/jersey.github.io/documentation/latest/test-framework.html#d0e17501)的文档。

## 9. 总结

在本教程中，我们探索了Jersey测试框架。首先，我们介绍了如何配置Jersey测试框架，然后了解了如何为一个非常简单的API编写测试。

之后，我们介绍了如何为各种GET和POST API端点编写测试。最后，我们了解了Jersey测试框架支持的一些附加功能和容器。