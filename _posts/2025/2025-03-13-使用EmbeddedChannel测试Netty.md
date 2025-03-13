---
layout: post
title:  使用EmbeddedChannel测试Netty
category: netty
copyright: netty
excerpt: Netty
---

## 1. 简介

在本文中，我们将了解如何使用EmbeddedChannel来测试入站和出站通道处理程序的功能。

[Netty](https://www.baeldung.com/netty)是一款功能非常丰富的框架，可用于编写高性能异步应用程序。如果没有合适的工具，对此类应用程序进行单元测试可能会非常棘手。

值得庆幸的是，框架为我们提供了**EmbeddedChannel类-这使得ChannelHandlers的测试更加方便**。

## 2. 设置

EmbeddedChannel是Netty框架的一部分，因此唯一需要的依赖就是Netty本身的依赖。

可以在[Maven Central](https://mvnrepository.com/artifact/io.netty/netty-all)上找到依赖项：

```xml
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.24.Final</version>
</dependency>
```

## 3. EmbeddedChannel概述

**EmbeddedChannel类只是AbstractChannel的另一种实现**-它不需要真正的网络连接即可传输数据。

这很有用，因为我们可以通过在入站通道上写入数据来模拟传入消息，还可以检查出站通道上生成的响应。这样，我们可以单独测试每个ChannelHandler或整个通道管道。

为了测试一个或多个ChannelHandler，我们首先必须使用它的一个构造函数创建一个EmbeddedChannel实例。

**初始化EmbeddedChannel的最常见方式是将ChannelHandler列表传递给其构造函数**：

```java
EmbeddedChannel channel = new EmbeddedChannel(
    new HttpMessageHandler(), new CalculatorOperationHandler());
```

如果我们想要更好地控制处理程序插入管道的顺序，我们可以使用默认构造函数创建一个EmbeddedChannel并直接添加处理程序：

```java
channel.pipeline()
    .addFirst(new HttpMessageHandler())
    .addLast(new CalculatorOperationHandler());
```

另外，**当我们创建一个EmbeddedChannel时，它将具有由DefaultChannelConfig类提供的默认配置**。

当我们想要使用自定义配置时，比如降低默认的连接超时值，我们可以使用config()方法访问ChannelConfig对象：

```java
DefaultChannelConfig channelConfig = (DefaultChannelConfig) channel
    .config();
channelConfig.setConnectTimeoutMillis(500);
```

EmbeddedChannel包含一些方法，我们可以使用它们来读取和写入数据到ChannelPipeline，最常用的方法是：

- readInbound()
- readOutbound()
- writeInbound(Object... msgs)
- writeOutbound(Object... msgs)

**读取方法检索并删除入站/出站队列中的第一个元素**，当我们需要访问整个消息队列而不删除任何元素时，我们可以使用outboundMessages()方法：

```java
Object lastOutboundMessage = channel.readOutbound();
Queue<Object> allOutboundMessages = channel.outboundMessages();
```

**当消息成功添加到Channel的入站/出站管道时，写入方法返回true**：

```java
channel.writeInbound(httpRequest)
```

我们的想法是，我们在入站管道上写入消息，以便输出ChannelHandler将处理它们，并且我们期望结果可以从出站管道读取。

## 4. 测试ChannelHandler

让我们看一个简单的例子，我们要测试一个由两个ChannelHandler组成的管道，它们接收一个HTTP请求并期望一个包含计算结果的HTTP响应：

```java
EmbeddedChannel channel = new EmbeddedChannel(
    new HttpMessageHandler(), new CalculatorOperationHandler());
```

第一个HttpMessageHandler会从HTTP请求中提取数据，并将其传递给管道中的第二个ChannelHandler，即CalculatorOperationHandler对数据进行处理。

现在，让我们编写HTTP请求并查看入站管道是否处理它：

```java
FullHttpRequest httpRequest = new DefaultFullHttpRequest(
    HttpVersion.HTTP_1_1, HttpMethod.GET, "/calculate?a=10&b=5");
httpRequest.headers().add("Operator", "Add");

assertThat(channel.writeInbound(httpRequest)).isTrue();
long inboundChannelResponse = channel.readInbound();
assertThat(inboundChannelResponse).isEqualTo(15);
```

我们可以看到我们已经使用writeInbound()方法在入站管道上发送了HTTP请求，并使用readInbound()读取了结果；inboundChannelResponse是我们发送的数据经过入站管道中的所有ChannelHandler处理后产生的消息。

现在，让我们检查Netty服务器是否响应了正确的HTTP响应消息。为此，我们将检查出站管道上是否存在消息：

```java
assertThat(channel.outboundMessages().size()).isEqualTo(1);
```

在本例中，出站消息是HTTP响应，因此让我们检查内容是否正确。我们通过读取出站管道中的最后一条消息来执行此操作：

```java
FullHttpResponse httpResponse = channel.readOutbound();
String httpResponseContent = httpResponse.content()
    .toString(Charset.defaultCharset());
assertThat(httpResponseContent).isEqualTo("15");
```

## 5. 测试异常处理

另一个常见的测试场景是异常处理。

我们可以通过实现exceptionCaught()方法来处理ChannelInboundHandler中的异常，但是有些情况下我们不想处理异常，而是将其传递给管道中的下一个ChannelHandler。

我们可以使用EmbeddedChannel类中的checkException()方法来检查管道上是否收到了任何Throwable对象并重新抛出它。

这样我们就可以捕获异常并检查ChannelHandler是否应该抛出它：

```java
assertThatThrownBy(() -> {
    channel.pipeline().fireChannelRead(wrongHttpRequest);
    channel.checkException();
}).isInstanceOf(UnsupportedOperationException.class)
  .hasMessage("HTTP method not supported");
```

从上面的例子中我们可以看到，我们发送了一个HTTP请求，我们期望该请求会触发Exception。通过使用checkException()方法，我们可以重新抛出管道中存在的最后一个异常，这样我们就可以断言需要从中得到什么。

## 6. 总结

EmbeddedChannel是Netty框架提供的一项很棒的功能，可以帮助我们测试ChannelHandler管道的正确性。它可用于单独测试每个ChannelHandler，更重要的是测试整个管道。