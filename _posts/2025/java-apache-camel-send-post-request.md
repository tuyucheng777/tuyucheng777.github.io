---
layout: post
title:  如何在Camel中发送Post请求
category: apache
copyright: apache
excerpt: Apache Camel
---

## 1. 概述

[Apache Camel](https://www.baeldung.com/apache-camel-intro)是一个强大的开源集成框架，它提供了一套成熟的组件来与各种协议和系统进行交互，包括[HTTP](https://www.baeldung.com/java-9-http-client)。

在本教程中，我们将探索Apache Camel HTTP组件并演示如何向[JSONPlaceholder](https://jsonplaceholder.typicode.com/)(一个用于测试和原型设计的免费虚假API)发起POST请求。

## 2. Apache Camel HTTP组件

Apache Camel HTTP组件提供与外部Web服务器通信的功能，它支持各种HTTP方法，包括GET、POST、PUT、DELETE等。

默认情况下，**HTTP组件使用端口80(用于HTTP)和端口443(用于HTTPS)**；以下是HTTP组件URI的常规语法：

```text
http://hostname[:port][/resourceUri][?options]
```

该组件必须以“http”或“https”模式开头，后跟主机名、可选端口、资源路径和查询参数。

我们可以使用[URI](https://www.baeldung.com/java-url-vs-uri)中的httpMethod选项设置HTTP方法：

```text
https://jsonplaceholder.typicode.com/posts?httpMethod=POST
```

另外，我们可以在消息头中设置HTTP方法：

```java
setHeader(Exchange.HTTP_METHOD, constant("POST"))
```

设置HTTP方法对于成功发起请求至关重要。

## 3. 项目设置

首先，让我们将[camel-core](https://mvnrepository.com/artifact/org.apache.camel/camel-core)和[camel-test-jnit5](https://mvnrepository.com/artifact/org.apache.camel/camel-test-junit5)依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-core</artifactId>
    <version>4.6.0</version>
</dependency>
<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-test-junit5</artifactId>
    <version>4.6.0</version>
</dependency>
```

camel-core依赖提供了系统集成的核心类，**其中一个重要的类是用于创建路由的RouteBuilder**；camel-test-junit5提供了使用[JUnit 5](https://www.baeldung.com/junit-5)测试Camel路由的支持。

接下来，让我们将[camel-jackson](https://mvnrepository.com/artifact/org.apache.camel/camel-jackson)和[camel-http](https://mvnrepository.com/artifact/org.apache.camel/camel-http)依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-jackson</artifactId>
    <version>4.6.0</version>
</dependency>
<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-http</artifactId>
    <version>4.6.0</version>
</dependency>
```

camel-http依赖为HTTP组件提供了与外部服务器通信的支持；此外，我们还添加了camel-jackson依赖，以便使用Jackson进行[JSON](https://www.baeldung.com/jackson-object-mapper-tutorial)序列化和反序列化。

然后，让我们为“https://jsonplaceholder.typicode.com/post”的POST请求创建一个示例[JSON](https://www.baeldung.com/java-json)有效负载：

```json
{
    "userId": 1,
    "title": "Java 21",
    "body": "Virtual Thread"
}
```

这里，有效负载包含userId、title和body，我们期望在成功创建新帖子后，端点返回HTTP状态码201。

## 4. 发送Post请求

首先，让我们创建一个名为PostRequestRoute的类，它扩展了RouteBuilder类：

```java
public class PostRequestRoute extends RouteBuilder { 
}
```

RouteBuilder类允许我们重写configure()方法来创建路由。

### 4.1 使用JSON字符串发送Post请求

让我们定义一个向我们的虚拟服务器发送POST请求的路由：

```java
from("direct:post").process(exchange -> exchange.getIn()
    .setBody("{\"title\":\"Java 21\",\"body\":\"Virtual Thread\",\"userId\":\"1\"}"))
    .setHeader(Exchange.CONTENT_TYPE, constant("application/json"))
    .to("https://jsonplaceholder.typicode.com/posts?httpMethod=POST")
    .to("mock:post");
```

在这里，我们定义一个路由并将有效负载设置为JSON字符串，**setBody()方法接收JSON字符串作为参数**；此外，我们使用httpMethod选项将HTTP方法设置为POST。

然后，我们将请求发送到JSONPlacehoder API；最后，我们将响应转发到Mock端点。

### 4.2 使用POJO类发送Post请求

但是，定义JSON字符串可能容易出错，为了更类型安全，我们定义一个名为Post的[POJO](https://www.baeldung.com/java-pojo-class)类：

```java
public class Post {
    private int userId;
    private String title;
    private String body;

    // standard constructor, getters, setters
}
```

接下来，让我们修改我们的路由以使用POJO类：

```java
from("direct:start").process(exchange -> exchange.getIn()
    .setBody(new Post(1, "Java 21", "Virtual Thread"))).marshal().json(JsonLibrary.Jackson)
    .setHeader(Exchange.HTTP_METHOD, constant("POST"))
    .setHeader(Exchange.CONTENT_TYPE, constant("application/json"))
    .to("https://jsonplaceholder.typicode.com/posts")
    .process(exchange -> log.info("The HTTP response code is: {}", exchange.getIn().getHeader(Exchange.HTTP_RESPONSE_CODE)))
    .process(exchange -> log.info("The response body is: {}", exchange.getIn().getBody(String.class)))
    .to("mock:result");
```

在这里，我们从名为start的直接端点开始。然后，创建一个Post实例并将其设置为请求主体。此外，**我们使用[Jackson](https://www.baeldung.com/jackson-json-view-annotation)将POJO封装为JSON**。

接下来，我们将请求发送到伪造的API，并[记录](https://www.baeldung.com/java-apache-camel-logging)响应码和正文。最后，我们将响应转发到Mock端点进行测试。

## 5. 测试路由

让我们编写一个测试来验证我们的路由行为，首先，让我们创建一个扩展CamelTestSupport类的测试类：

```java
class PostRequestRouteUnitTest extends CamelTestSupport {
}
```

然后，让我们创建一个Mock端点和生产者模板：

```java
@EndpointInject("mock:result")
protected MockEndpoint resultEndpoint;

@Produce("direct:start")
protected ProducerTemplate template;
```

接下来，让我们重写createRouteBuilder()方法以使用PostRequestRoute：

```java
@Override
protected RouteBuilder createRouteBuilder() {
    return new PostRequestRoute();
}
```

最后我们来写一个测试方法：

```java
@Test
void whenMakingAPostRequestToDummyServer_thenAscertainTheMockEndpointReceiveOneMessage() throws Exception {
    resultEndpoint.expectedMessageCount(1);
    resultEndpoint.message(0).header(Exchange.HTTP_RESPONSE_CODE)
        .isEqualTo(201);
    resultEndpoint.message(0).body()
        .isNotNull();

    template.sendBody(new Post(1, "Java 21", "Virtual Thread"));

    resultEndpoint.assertIsSatisfied();
}
```

在上面的代码中，我们定义了对Mock端点的期望，并使用template.sendBody()方法发送请求。最后，我们确认Mock端点的期望已得到满足。

## 6. 总结

在本文中，我们将学习如何使用Apache Camel向外部服务器发出POST请求。首先，我们定义一个使用JSON字符串和POJO发送POST请求的路由。

此外，我们还了解了如何使用HTTP组件与外部API通信；最后，我们编写了一个单元测试来验证路由的行为。