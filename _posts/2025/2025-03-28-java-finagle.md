---
layout: post
title:  Finagle简介
category: libraries
copyright: libraries
excerpt: Finagle
---

## 1. 概述

在本教程中，我们将快速介绍Twitter的RPC库Finagle。

我们将使用它来构建一个简单的客户端和服务器。

## 2. 构建块

在深入实现之前，我们需要了解构建应用程序所需的基本概念。这些概念广为人知，但在Finagle的世界中含义略有不同。

### 2.1 Service

服务是由类表示的功能，它接收请求并返回包含操作最终结果或有关失败的信息的Future。

### 2.2 Filter

过滤器也是函数，它们接收请求和服务，对请求执行一些操作，将其传递给服务，对生成的Future执行一些操作，最后返回最终的Future。我们可以将它们视为[切面](https://www.baeldung.com/spring-aop)，因为它们可以实现围绕函数执行发生的逻辑并改变其输入和输出。

### 2.3 Future

Future表示异步操作的最终结果，它们可能处于以下三种状态之一：待处理、成功或失败。

## 3. 服务

首先，我们将实现一个简单的HTTP问候服务，它将从请求中获取name参数并响应，并添加惯常的“Hello”消息。

为此，我们需要创建一个类，它将扩展Finagle库中的抽象Service类，**并实现其apply方法**。

我们所做的看起来类似于实现一个[函数式接口](https://www.baeldung.com/java-8-functional-interfaces)。但有趣的是，我们实际上不能使用这个特定的功能，因为Finagle是用Scala编写的，而我们利用的是Java-Scala互操作性：

```java
public class GreetingService extends Service<Request, Response> {
    @Override
    public Future<Response> apply(Request request) {
        String greeting = "Hello " + request.getParam("name");
        Reader<Buf> reader = Reader.fromBuf(new Buf.ByteArray(greeting.getBytes(), 0, greeting.length()));
        return Future.value(Response.apply(request.version(), Status.Ok(), reader));
    }
}
```

## 4. 过滤器

接下来，我们将编写一个过滤器，将有关请求的一些数据记录到控制台。与Service类似，我们需要实现Filter的apply方法，该方法将接收请求并返回Future响应，但这次它还将接收Service作为第二个参数。

基本的[Filter](https://twitter.github.io/finagle/docs/com/twitter/finagle/Filter.html)类有4种类型参数，但很多时候我们不需要改变过滤器内部的请求和响应的类型。

为此，我们将使用将4个类型参数合并为2个的[SimpleFilter](https://twitter.github.io/finagle/docs/com/twitter/finagle/SimpleFilter.html)，我们将从请求中打印一些信息，然后简单地从提供的Service中调用apply方法：

```java
public class LogFilter extends SimpleFilter<Request, Response> {
    @Override
    public Future apply(Request request, Service<Request, Response> service) {
        logger.info("Request host:" + request.host().getOrElse(() -> ""));
        logger.info("Request params:");
        request.getParams().forEach(entry -> logger.info("\t" + entry.getKey() + " : " + entry.getValue()));
        return service.apply(request);
    }
}
```

## 5. 服务器

现在我们可以使用服务和过滤器来构建一个实际监听请求并处理请求的服务器。

我们将为该服务器提供一项服务，该服务包含通过andThen方法链接在一起的过滤器和服务：

```java
Service serverService = new LogFilter().andThen(new GreetingService()); 
Http.serve(":8080", serverService);
```

## 6. 客户端

最后，我们需要一个客户端向我们的服务器发送请求。

为此，我们将使用Finagle的[Http](https://twitter.github.io/finagle/docs/com/twitter/finagle/Http$.html)类中便捷的newService方法创建一个HTTP服务，它将直接负责发送请求。

此外，我们将使用之前实现的相同日志过滤器并将其与HTTP服务链接起来。然后，我们只需调用apply方法即可。

**最后一个操作是异步的，其最终结果存储在Future实例中**。我们可以等待这个Future成功或失败，但这将是一个阻塞操作，我们可能希望避免它。相反，我们可以实现一个在Future成功时触发的回调：

```java
Service<Request, Response> clientService = new LogFilter().andThen(Http.newService(":8080"));
Request request = Request.apply(Method.Get(), "/?name=John");
request.host("localhost");
Future<Response> response = clientService.apply(request);

Await.result(response
        .onSuccess(r -> {
            assertEquals("Hello John", r.getContentString());
            return BoxedUnit.UNIT;
        })
        .onFailure(r -> {
            throw new RuntimeException(r);
        })
);
```

请注意，我们返回的是BoxedUnit.UNIT。返回[Unit](https://www.scala-lang.org/api/current/scala/Unit.html)是Scala处理void方法的方式，因此我们在这里这样做是为了保持互操作性。

## 7. 总结

在本教程中，我们学习了如何使用Finagle构建一个简单的HTTP服务器和一个客户端，以及如何在它们之间建立通信并交换消息。