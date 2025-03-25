---
layout: post
title:  使用Nats Java客户端发布和接收消息
category: libraries
copyright: libraries
excerpt: NATS
---

## 1. 概述

在本教程中，我们将使用[NATS的Java客户端](https://github.com/nats-io/nats.java?tab=readme-ov-file#nats---java-client)连接到[NATS服务器](https://docs.nats.io/)，以便我们可以发布和接收消息。

NATS提供三种主要的消息交换模式：

- [发布/订阅](https://www.enterpriseintegrationpatterns.com/patterns/messaging/PublishSubscribeChannel.html)语义将消息传递给某个主题的所有订阅者。
- 请求/回复消息传递将请求发送给主题的所有订阅者，并将响应路由回请求者。返回给请求者的第一个响应将被处理，我们可以轻松构建一个类似请求/回复的系统，其中所有响应都将被处理，此功能可能会在未来版本中添加。
- 订阅者在订阅主题时也可以加入消息队列组，发送到相关主题的消息只会传递给队列组中的一个订阅者，此语义可用于发布/订阅或请求/回复。

## 2. 设置

本节讨论运行示例所需的基本设置。

### 2.1 Maven依赖

首先，我们需要将[NATS库](https://mvnrepository.com/artifact/io.nats/jnats)添加到我们的pom.xml文件中：

```xml
<dependency>
    <groupId>io.nats</groupId>
    <artifactId>jnats</artifactId>
    <version>2.17.6</version>
</dependency>
```

Java客户端REAMDE文件的[Maven部分](https://github.com/nats-io/nats.java?tab=readme-ov-file#using-maven)提供了更详细的说明，Gradle用户可以在README文件的[Gradle部分找到说明](https://github.com/nats-io/nats.java?tab=readme-ov-file#using-gradle)。

### 2.2 Nats服务器

其次，我们需要一个Nats服务器来交换消息，[NATS文档网站](https://docs.nats.io/nats-server/installation)上有针对所有主要平台的说明。

我们假设在localhost:4222上有一个服务器正在运行。

## 3. 连接到Nats服务器

要交换消息，首先需要连接到服务器。客户端连接后，可以通过发布和订阅主题来交换消息。

### 3.1 使用或不使用自定义Options进行连接 

静态Nats类中的Nats.connect()方法创建一个Connection。

要创建连接，我们可以提供服务器的主机和端口等信息。还有其他可配置的连接选项，例如用户名和密码、如何处理安全连接以及影响连接行为方式的许多其他选项。如果我们不向Nats.connect()方法提供Options对象，它将使用所有默认值并尝试连接到localhost上的服务器，端口为4222：

```java
Connection natsConnection = Nats.connect();
```

为了演示如何使用自定义选项创建连接，让我们编写一个示例，使用以下内容构建自定义Options对象：

- 接收连接事件并打印出来的自定义ConnectionListener
- 接收错误事件并打印出来的自定义ErrorListener

此函数为特定URI创建一个Options对象并将其传递给Nats.connect，如果URI为null或为空，则Options将采用默认服务器位置：

```java
Options options = new Options.Builder()
    .server(uri)
    .connectionListener((connection, event) -> log.info("Connection Event: " + event))
    .errorListener(new CustomErrorListener())
    .build();
```

接下来要做的事情是使用该Options实例进行连接：

```java
Connection natsConnection = Nats.connect(options);
```

NATS连接持久，API将自动尝试重新连接丢失的连接。

作为连接选项的一部分，我们安装了一个ConnectionListener实现Lambda，用于在发生连接事件(如断开连接或重新连接)时通知我们。我们还安装了一个ErrorListener。ErrorListener接口包含多个方法，因此无法使用Lambda创建。示例代码有一个名为CustomErrorListener的实现。

让我们进行一个快速测试。首先，我们创建一个新的NatsClient，它将初始化一个连接。然后，我们添加60秒的睡眠以保持进程运行：

```java
Connection conn = createConnection("nats://localhost:4222");
Thread.sleep(60000);
```

当我们运行这个程序时，连接监听器将报告已打开的连接：

```text
Connection Event: nats: connection opened
```

然后，让我们停止并启动我们的Nats服务器：

```text
Exception Occurred: java.io.IOException: Read channel closed.
Connection Event: nats: connection disconnected
Exception Occurred: java.net.ConnectException: Connection refused: no further information
Connection Event: nats: connection disconnected
Exception Occurred: java.net.ConnectException: Connection refused: no further information
Connection Event: nats: connection disconnected
Connection Event: nats: connection reconnected
Connection Event: nats: subscriptions re-established
```

我们可以看到ConnectionListener和CustomErrorListener接收并处理异常，断开连接，重新连接等事件。

## 4. 交换信息

现在我们有了Connection，我们可以进行消息处理了。交换消息需要在主题上发布和订阅。

当我们向某个主题发布消息时，订阅该主题的程序将收到该消息的副本，订阅者必须在消息发布前做好准备。

NATS消息是字节数组的容器，除了预期的setData(byte[])和byte[] getData()方法外，还有用于设置和获取消息目标主题和回复主题的方法。

### 4.1 订阅消息

我们订阅的主题是String。

**NATS支持同步和异步订阅**。

### 4.2 异步订阅

异步订阅需要Dispatcher，Dispatcher包含将传入消息发送给异步处理程序的线程。

由于一个Connection可以有多个订阅，因此我们可以决定是否为每个订阅设置一个Dispatcher，或者在多个订阅之间共享一个Dispatcher。每个Dispatcher都有一个在其自己的线程上执行的运行循环。运行循环调用消息处理程序的onMessage(Message)方法并等待处理程序返回。

如果我们有多个负载较高的订阅，或者我们的消息处理程序需要很长时间才能处理每条消息，那么最好为每个订阅设置一个Dispatcher。如果偶尔有消息或处理时间很短，那么让它们共享一个Dispatcher也是可以的，这通常根据用例针对每个应用程序进行调整。

在此示例中，让我们创建一个Dispatcher并将其用于多个订阅。无论是否用于多个订阅，创建Dispatcher的过程都是相同的：

```java
Dispatcher dispatcher = natsConnection.createDispatcher();
Subscription subscription = dispatcher.subscribe(subject, msg -> log.info("Subscription received message " + msg));
Subscription qSubscription = dispatcher.subscribe(subject, queueGroup, msg -> log.info("Queue subscription received message " + msg));
```

订阅完成后，最好取消订阅。如果我们不取消订阅，服务器将继续向主题发送消息，直到连接关闭。要取消使用Dispatcher进行的订阅，我们调用unsubscribe(...)：

```java
dispatcher.unsubscribe(subscription);
```

### 4.3 同步订阅

同步订阅要求每个订阅者在其自己的线程中使用nextMessage方法轮询消息。在这里，我们创建一个同步订阅。请注意，不需要调度程序：

```java
Subscription subscription = natsConnection.subscribe("mySubject");
Message message = subscription.nextMessage(1000);
```

nextMessage(...)方法调用将等待指定的毫秒数或直到收到消息。如果在超时时间内没有可用的消息，则nextMessage返回null。

我们将使用同步订阅进行测试，以使测试用例保持简单。完成同步订阅后，我们需要直接从订阅中调用取消订阅：

```java
subscription.unsubscribe();
```

### 4.4 发布消息

发布消息是一种“发射后不管”的操作-我们不知道是否有人订阅了该消息。我们可以使用几种不同的API来发布消息。最简单的方法只需要一个主题字符串和消息字节：

```java
natsConnection.publish("mySubject", "Hi there!".getBytes());
```

对于其他一些发布组合，也有过载，比如传递消息而不是字节。

### 4.5 消息响应

有两种方法可以获取已发布消息的响应。我们可以使用reply-to进行常规发布，也可以使用内置的请求功能。如果我们想收到对任何给定主题的多个响应，则需要使用发布模式。如果我们只需要对任何给定主题的第一个响应，则可以使用请求模式。

### 4.6 发布以获得大量回应

消息中的reply-to字段是为发布者提供的，用于提供响应者将订阅的主题。我们可以在发布消息 时自己提供reply-to主题。然后，作为发布者的我们可以订阅该reply-to主题并等待我们想要的任意数量的响应：

```java
Subscription replySideSubscription1 = client.subscribeSync("publishSubject");
Subscription replySideSubscription2 = client.subscribeSync("publishSubject");
Subscription publishSideSubscription = client.subscribeSync("replyToSubject");
natsConnection.publish("publishSubject", "replyToSubject", "Please respond!".getBytes());
```

回复端订阅者将订阅发布主题。请注意，我们不必对回复主题进行硬编码，因为它随消息一起提供：

```java
Message m1 = replySideSubscription1.nextMessage(1000);
natsConnection.publish(m1.getReplyTo(), "Message Received By Subscription 1".getBytes());
Message m2 = replySideSubscription2.nextMessage(1000);
natsConnection.publish(m2.getReplyTo(), "Message Received By Subscription 2".getBytes());
```

发布端订阅可以得到所有回复，回复的顺序与服务器接收的顺序一致：

```java
Message m1 = publishSideSubscription.nextMessage(1000);
Message m2 = publishSideSubscription.nextMessage(1000);
```

### 4.7 使用请求(/回复)仅获取一个响应

当我们使用请求方法时，我们无法提供回复主题-客户端会为我们完成这项工作。它会等待第一个响应，这意味着即使多个订阅者回复，也只会返回第一个收到的响应。所有其他响应都会被忽略。

```java
Message response = natsConnection.request("requestSubject", "Please respond!".getBytes(), Duration.ofMillis(1000));
```

回复端订阅是相同的，但请求者(发布者)不需要设置订阅，只需等待响应即可。请求方法也有重载，使用CompletableFuture<Message\>代替内置超时。

### 4.8 通配符订阅

NATS服务器支持主题通配符。

通配符作用于由“.”字符分隔的主题标记。星号“*”匹配单个标记。大于号“>”是通配符，用于匹配主题的其余部分，这些部分可能不止一个标记。

例如：

- foo.*匹配foo.bar，但不匹配foo.bar.requests
- foo.>匹配foo.bar、foo.bar.requests、foo.bar.tuyucheng等等

让我们尝试几个测试：

```java
Subscription starSubscription = natsConnection.subscribe("segment.*");
natsConnection.publish("segment.another", convertStringToBytes("hello segment star"));

Message message = segmentStarSubscription.nextMessage(200);
assertNotNull("No message!", message);
assertEquals("hello segment star sub", convertMessageDataBytesToString(message));

natsConnection.publish("segment.one.two", "hello there");
message = segmentStarSubscription.nextMessage(200);
assertNull("Got message!", message);

Subscription segmentGreaterSubscription = natsConnection.subscribe("segment.>");

client.publishMessage("segment.one.two", "hello segment greater sub");

message = greaterSubscription.nextMessage(200);
assertNotNull("No message!", message);
assertEquals("hello segment greater sub", new String(message.getData()));
```

## 5. 消息队列

订阅者可以在订阅时指定队列组。当消息发布到该组时，NATS将把它投递给一个且只有一个订阅者。

队列组不持久化消息。如果没有可用的监听器，则消息将被丢弃。

### 5.1 发布到队列

将消息发布到队列组与正常发布没有什么不同-它只需要发布到相关主题：

```java
natsConnection.publish("mySubject", convertStringToBytes("data"));
```

### 5.2 订阅队列

订阅者以字符串形式指定队列组名称：

```java
Subscription subscription = natsConnection.subscribe("mySubject", "myQueue");
```

当然还有一个异步版本：

```java
List<Message> messages = new ArrayList<>();
Dispatcher dispatcher = connection.createDispatcher();
Subscription subscription = dispatcher.subscribe("mySubject", "myQueue", messages::add);

```

订阅在Nats服务器上创建队列，NATS服务器将消息路由到队列并选择消息接收者：

```java
Subscription queue1 = natsConnection.subscribe("mySubject", "myQueue");
Subscription queue2 = natsConnection.subscribe("mySubject", "myQueue");
```

只有一个订阅(queue1或queue2)会收到任何特定消息。如果我们订阅同一主题但不订阅队列，如下所示：

```java
Subscription subscription1 = natsConnection.subscribe("mySubject");
Subscription subscription2 = natsConnection.subscribe("mySubject");

```

那么subscription1和subscription2都会收到所有消息。如果同时订阅了四个订阅：queue1、queue2、subscription1和subscription2，那么queue1和queue2仍然会作为一个队列工作，并且subscription1和subscription2仍然会收到所有消息。

## 6. 总结

在这个简短的介绍中，我们连接到Nats服务器并发送发布/订阅消息和负载平衡队列消息。我们研究了Nats对通配符订阅的支持，我们还使用了请求/回复消息传递。