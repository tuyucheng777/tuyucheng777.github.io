---
layout: post
title:  在Java中运行时添加Camel路由
category: apache
copyright: apache
excerpt: Apache
---

## 1. 概述

[Apache Camel](https://www.baeldung.com/apache-camel-intro)是一个Java框架，可以轻松实现各种企业集成模式(EIP)，为企业集成提供解决方案。

集成模式中的常见任务之一是根据特定规则和条件在运行时确定消息路由，Apache Camel通过提供实现动态路由器EIP的方法简化了此过程。

在本教程中，我们将深入探讨如何在Apache Camel中实现动态路由的细节并通过一个示例进行讲解。

## 2. 了解动态路由器

有时，我们希望在运行时根据特定的规则和条件将消息发送到不同的路由，像路由单EIP这样的解决方案可以帮助解决这个问题，但由于需要反复试验，效率较低。

在路由单EIP中，一条消息包含要按定义顺序路由到的端点列表，这需要预先配置端点列表，并使用反复试验的方法通过每个端点发送消息。

动态路由EIP提供了更好的实现，可以在运行时添加路由，尤其是在有多个收件人或根本没有收件人的情况下。它提供了灵活性，无需预先配置严格的端点即可路由消息。

此外，它了解每个目的地以及将消息路由到特定目的地的规则。**此外，它还拥有一个控制通道，潜在目的地可以在启动时通过宣布其存在和参与规则来与其进行通信**。

此外，它还将所有可能目的地的规则存储在规则库中，消息到达后，它会检查规则库并满足收件人的请求。

[下图](https://camel.apache.org/components/4.8.x/eips/_images/eip/DynamicRouter.gif)展示了动态路由器EIP的内部结构：

![](/assets/images/2025/apache/javacameldynamicroute01.png)

**此外，一个常见的用例是动态服务发现，其中客户端应用程序可以通过发送包含服务名称的消息来访问服务**。

动态路由器EIP的核心目标是将路由决策推迟到运行时，而不是预先构建静态规则。

## 3. Maven依赖

让我们通过将[camel-core](https://mvnrepository.com/artifact/org.apache.camel/camel-core)和[camel-test-junit5](https://mvnrepository.com/artifact/org.apache.camel/camel-test-junit5)添加到pom.xml来引导一个简单的Apache Camel项目：

```xml
<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-core</artifactId>
    <version>4.3.0</version>
</dependency>
<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-test-junit5</artifactId>
    <version>4.3.0</version>
</dependency>
```

Camel Core提供动态路由器EIP以及其他路由功能，而Camel Test JUnit5则通过CamelSupport接口帮助更轻松地测试消息路由。

值得注意的是，我们还可以将Camel项目引导为[Spring Boot](https://www.baeldung.com/apache-camel-spring-boot)项目。

## 4. 使用动态路由器在运行时添加路由

动态路由器EIP确保我们为集成系统指定规则，以便在运行时正确匹配到特定路由。它会检查传入消息的主体并将其与路由匹配。

### 4.1 配置

首先，让我们创建一个名为DynamicRouteBean的类，并添加一个方法来定义规则和条件：

```java
class DynamicRouterBean {
    public String route(String body, @ExchangeProperties Map<String, Object> properties) {
        int invoked = (int) properties.getOrDefault("invoked", 0) + 1;

        properties.put("invoked", invoked);

        if (invoked == 1) {
            switch (body.toLowerCase()) {
                case "mock":
                    return "mock:dynamicRouter";
                case "direct":
                    return "mock:directDynamicRouter";
                case "seda":
                    return "mock:sedaDynamicRouter";
                case "file":
                    return "mock:fileDynamicRouter";
                default:
                    break;
            }
        }
        return null;
    }
}
```

在上面的代码中，我们根据传入的消息正文和当前调用次数动态确定合适的路由，route()方法检查消息正文是否与任何预定义的关键字规则匹配，并返回相应的路由。

此外，我们在[Map](https://www.baeldung.com/java-hashmap)对象上使用@ExchangeProperties注解，**此Map用作容器，用于存储和检索[交换](https://www.baeldung.com/apache-camel-intro#terminology-and-architecture)的当前状态并更新调用次数**。

此外，invoked变量表示route()方法被调用的次数。**如果消息符合预定义条件，并且是首次调用，则返回相应的路由。invoked == 1检查有助于在首次调用时执行动态路由逻辑，这简化了此特定情况下的代码**，并避免了不必要的重复执行。

**此外，为了防止动态路由器无限执行，route()方法在路由到相应端点后返回null，这确保了动态路由在根据消息识别出路由后结束**。

简单来说，每次交换都会调用route()方法，直到返回null。

最后，让我们在Camel路由构建器中配置动态路由器EIP：

```java
class DynamicRouterRoute extends RouteBuilder {
    @Override
    void configure() {
        from("direct:dynamicRouter")
                .dynamicRouter(method(DynamicRouterBean.class, "route"));
    }
}
```

在上面的代码中，我们创建了继承自RouteBuilder的DynamicRouterRoute类。接下来，我们重写configure方法，并通过调用dynamicRouter()方法添加动态路由Bean，将动态路由器调用连接到我们自定义的route()。

值得注意的是，我们可以在定义规则的方法上使用@DynamicRouter注解：

```java
class DynamicRouterBean {
    @DynamicRouter
    String route(String body, @ExchangeProperties Map<String, Object> properties) {
        // ...
    }
}
```

该注解消除了在Camel路由中明确配置dynamicRouter()方法的需要：

```java
// ...
@Override
void configure() {
    from("direct:dynamicRouter").bean(DynamicRouterBean.class, "route");
}
// ...
```

在上面的代码中，我们使用bean()方法指定包含路由逻辑的类。dynamicRouter()方法不再需要，因为route()方法已使用@DynamicRouter注解。

### 4.2 单元测试

让我们编写一个单元测试来断言某些条件是否成立，首先，确保我们的测试类扩展了CamelTestSupport：

```java
class DynamicRouterRouteUnitTest extends CamelTestSupport {
    @Override
    protected RoutesBuilder createRouteBuilder() {
        return new DynamicRouterRoute();
    }
}
```

这里提供一个DynamicRouterRoute实例，用作测试的路由构建器。

接下来，让我们看看一个名为mock的传入消息体：

```java
@Test
void whenMockEndpointExpectedMessageCountOneAndMockAsMessageBody_thenMessageSentToDynamicRouter() throws InterruptedException {
    MockEndpoint mockDynamicEndpoint = getMockEndpoint("mock:dynamicRouter");
    mockDynamicEndpoint.expectedMessageCount(1);

    template.send("direct:dynamicRouter", exchange -> exchange.getIn().setBody("mock"));
    MockEndpoint.assertIsSatisfied(context);
}
```

在这里，我们Mock动态端点，并将预期的消息路由设置为其值。接下来，我们将预期的消息数量设置为1。最后，我们设置一个传入消息路由，其中包含预期的正文消息，并断言MockEndpoint路由已满足要求。

另外，让我们Mock “mock：directDynamicRouter”消息路由：

```java
@Test
void whenMockEndpointExpectedMessageCountOneAndDirectAsMessageBody_thenMessageSentToDynamicRouter() throws InterruptedException {
    MockEndpoint mockDynamicEndpoint = context.getEndpoint("mock:directDynamicRouter", MockEndpoint.class);
    mockDynamicEndpoint.expectedMessageCount(1);

    template.send("direct:dynamicRouter", exchange -> exchange.getIn().setBody("direct"));

    MockEndpoint.assertIsSatisfied(context);
}
```

此测试验证当“direct”作为消息体发送时，它是否会动态路由到“mock:directDynamicRouter”端点。**此外，我们将预期消息计数设置为1，表示端点应接收的消息交换次数**。

## 5. 总结

在本文中，我们学习了如何使用动态路由器EIP在Apache Camel中运行时添加路由。与使用试错法向路由发送消息的路由单不同，动态路由器EIP提供了可靠的实现，可以根据特定规则和条件路由到端点。