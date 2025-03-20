---
layout: post
title:  Quarkus WebSockets Next
category: quarkus
copyright: quarkus
excerpt: Quarkus
---

## 1. 简介

在本文中，我们将研究[Quarkus](https://www.baeldung.com/quarkus-io)框架的[quarkus-websockets-next](https://quarkus.io/guides/websockets-next-tutorial)扩展，此扩展是一个新的实验性扩展，用于在我们的应用程序中支持[WebSockets](https://www.baeldung.com/rest-vs-websockets)。

**Quarkus WebSockets Next是一个新的扩展，旨在取代旧的Quarkus WebSockets扩展。与旧扩展相比，它将更易于使用且更高效**。

然而，与Quarkus不同，它不支持[Jakarta WebSockets API](https://jakarta.ee/specifications/websocket/)，而是提供了一个简化且更现代的API来处理WebSockets。它使用自己的带注解的类和方法，在功能上提供了更大的灵活性，同时还提供了JSON支持等内置功能。

与此同时，Quarkus WebSockets Next仍然建立在标准Quarkus核心之上。这意味着我们可以获得我们期望的所有预期性能和可扩展性，同时还可以受益于Quarkus为我们提供的开发体验。

## 2. 依赖

**如果我们要开始一个全新的项目，我们可以使用Maven创建已安装websockets-next扩展的结构**：

```shell
$ mvn io.quarkus.platform:quarkus-maven-plugin:3.16.4:create \
    -DprojectGroupId=cn.tuyucheng.taketoday.quarkus \
    -DprojectArtifactId=quarkus-websockets-next \
    -Dextensions='websockets-next'
```

请注意，由于该扩展仍处于实验阶段，因此我们需要使用[io.quarkus.platform:quarkus-maven-plugin](https://mvnrepository.com/artifact/io.quarkus.platform/quarkus-maven-plugin)。

或者，如果我们正在使用现有项目，我们可以简单地将适当的依赖添加到pom.xml文件中：

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-websockets-next</artifactId>
</dependency>
```

## 3. 服务器端点

一旦我们的应用程序准备好并且安装了websockets-next扩展，我们就可以开始使用WebSockets了。

**通过创建用@WebSocket标注的新类，在Quarkus中创建服务器端点**：

```java
@WebSocket(path = "/echo")
public class EchoWebsocket {
    // WebSocket code here.
}
```

这将创建一个监听所提供路径的端点，与Quarkus一样，我们可以根据需要在其中使用路径参数以及固定路径。

### 3.1 消息回调

为了使我们的WebSocket端点有用，我们需要能够处理消息。

WebSocket连接支持两种类型的消息-文本和二进制，**我们可以在服务器端点使用带有@OnTextMessage或@OnBinaryMessage注解的方法处理这些消息**：

```java
@OnTextMessage
public String onMessage(String message) {
    return message;
}
```

在这种情况下，我们将消息负载作为方法参数接收，并将方法的返回值发送到客户端。因此，此示例就是我们所需的回显服务器-我们将每条收到的消息原封不动地发回。

如果有必要，我们还可以接收二进制有效负载而不是文本有效负载，这是通过使用@OnBinaryMessage注解来实现的：

```java
@OnBinaryMessage
public Buffer onMessage(Buffer message) {
    return message;
}
```

在这种情况下，我们接收并返回一个io.vertx.core.buffer.Buffer实例，它将包含收到的消息的原始字节。

### 3.2 方法参数和返回值

除了我们的处理程序接收传入消息的原始有效负载之外，Quarkus还能够将许多其他内容传递到我们的回调消息中。

**所有消息处理程序都必须有一个参数，表示传入消息的有效负载，此参数的确切类型决定了我们如何访问它**。

正如我们之前看到的，如果我们使用Buffer或byte[\]，我们将获得传入消息的精确字节。如果我们使用String，这些字节将首先被解码为字符串。

不过，我们也可以使用更丰富的对象。如果我们使用JsonObject或JsonArray，则传入消息将被视为JSON并根据需要进行解码。或者，如果我们使用Quarkus不支持的任何其他类型，Quarkus将尝试将传入消息反序列化为该类型：

```java
@OnTextMessage
public Message onTextMessage(Message message) {
    return message;
}

record Message(String message) {}
```

**我们也可以使用所有这些相同的类型作为返回值，在这种情况下，Quarkus将在将消息发送回客户端时完全按照预期对消息进行序列化**。此外，消息处理程序可以有一个void返回，以指示不会将任何内容作为响应发送回去。

除此之外，还有一些其他的方法参数我们可以接收。

如我们所见，消息有效负载将提供一个未标注的String参数。但是，我们也可以使用@PathParam标注的String参数来接收传入URL中此参数的值：

```java
@WebSocket(path = "/chat/:user")
public class ChatWebsocket {

    @OnTextMessage(broadcast = true)
    public String onTextMessage(String message, @PathParam("user") String user) {
        return user + ": " + message;
    }
}
```

我们还可以接收WebSocketConnection类型的参数，它将代表客户端和服务器之间的精确连接：

```java
@OnTextMessage
public Map<String, String> onTextMessage(String message, WebSocketConnection connection) {
    return Map.of(
        "message", message,
        "connection", connection.toString()
    );
}
```

使用它可以让我们访问客户端和服务器之间的网络连接的详细信息-包括连接的唯一ID、建立连接的时间以及用于连接的URL的路径参数。

我们还可以通过向其发送消息甚至强行关闭它，来使用它来更直接地与连接进行交互：

```java
@OnTextMessage
public void onTextMessage(String message, WebSocketConnection connection) {
    if ("close".equals(message)) {
        connection.sendTextAndAwait("Goodbye");
        connection.closeAndAwait();
    }
}
```

### 3.3 OnOpen和OnClose回调

**除了用于接收消息的处理程序之外，我们还可以注册当首次打开新连接时的处理程序-使用@OnOpen，以及当关闭连接时的处理程序-使用@OnClose**：

```java
@OnOpen
public void onOpen() {
    LOG.info("Connection opened");
}

@OnClose
public void onClose() {
    LOG.info("Connection closed");
}
```

这些回调处理程序不能接收任何消息负载，但可以接收如前所述的任何其他方法参数。

此外，@OnOpen处理程序可以有一个返回值，该返回值将被序列化并发送给客户端。这对于在连接时立即发送消息非常有用，而无需等待客户端先发送某些内容。执行此操作遵循与消息处理程序的返回值相同的规则：

```java
@OnOpen
public String onOpen(WebSocketConnection connection) {
    return "Hello, " + connection.id();
}
```

### 3.4 访问连接

我们已经看到，我们可以将当前连接注入到回调处理程序中。但是，这并不是我们访问连接详细信息的唯一方法。

**Quarkus允许我们使用@Inject将WebSocketConnection对象作为[CDI会话作用域Bean](https://www.baeldung.com/java-ee-cdi#field-injection)注入**，这可以注入到我们系统中的任何其他Bean中，我们可以从那里访问当前连接：

```java
@ApplicationScoped
public class CdiConnectionService {
    @Inject
    WebSocketConnection connection;
}
```

但是，这仅在从WebSocket处理程序上下文中调用时才有效。如果我们尝试从任何其他上下文(包括常规HTTP调用)访问它，则将抛出jakarta.enterprise.context.ContextNotActiveException。

**我们还可以通过注入OpenConnections类型的对象来访问所有当前打开的WebSocket连接**：

```java
@Inject
OpenConnections connections;
```

然后我们不仅可以使用它来查询所有当前打开的连接，还可以向它们发送消息：

```java
public void sendToAll(String message) {
    connections.forEach(connection -> connection.sendTextAndAwait(message));
}
```

与注入单个WebSocketConnection不同，它可以在任何上下文中正常工作，这允许我们在需要时从任何其他上下文访问WebSocket连接。

### 3.5 错误处理

在某些情况下，当我们处理WebSocket回调时，可能会出错。**Quarkus允许我们通过编写异常处理程序方法来处理这些方法抛出的任何异常，这些是使用@OnError标注的任何方法**：

```java
@OnError
public String onError(RuntimeException e) {
    return e.toString();
}
```

这些回调处理程序遵循与其他回调处理程序相同的规则，涉及它们可以接收的参数及其返回值。此外，它们必须有一个表示要处理的异常的参数。

我们可以根据需要编写任意数量的错误处理程序，只要它们都适用于不同的异常类即可。如果有重叠的异常处理程序(换句话说，如果一个异常处理程序是另一个异常处理程序的子类)，则将调用最具体的异常处理程序：

```java
@OnError
public String onIoException(IOException e) {
    // Handles IOException and all subclasses.
}

@OnError
public String onException(Exception e) {
    // Handles Exception and all subclasses except for IOException.
}
```

## 4. 客户端API

Quarkus除了允许我们编写服务器端点之外，还允许我们编写可以与其他服务器通信的WebSocket客户端。

### 4.1 基本连接器

**编写WebSocket客户端的最基本方法是使用BasicWebSocketConnector，这允许我们可以打开连接并发送和接收原始消息**。

首先，我们需要在代码中注入一个BasicWebSocketConnector：

```java
@Inject
BasicWebSocketConnector connector;
```

然后我们可以使用它来连接到远程服务：

```java
WebSocketClientConnection connection = connector
    .baseUri(serverUrl)
    .executionModel(BasicWebSocketConnector.ExecutionModel.NON_BLOCKING)
    .onTextMessage((c, m) -> {
        // Handle incoming messages.
    })
    .connectAndAwait();
```

**作为此连接的一部分，我们注册了一个Lambda来处理从服务器收到的任何传入消息**。这是必要的，因为WebSockets具有异步、全双工的特性，我们不能仅仅将其视为具有请求和响应对的标准HTTP连接。

一旦我们打开了连接，我们就可以用它向服务器发送消息：

```java
connection.sendTextAndAwait("Hello, World!");
```

我们既可以在回调处理程序内部执行此操作，也可以在其外部执行此操作。但是，请记住，**连接不是线程安全的**，因此我们需要确保永远不会同时通过多个线程对其进行写入。

除了onTextMessage回调之外，我们还可以注册其他生命周期事件的回调，包括onOpen()和onClose()。

### 4.2 富客户端Bean

基本连接器足以应付简单的连接，但有时我们需要更灵活的功能。**Quarkus还允许我们编写更丰富的客户端，就像我们编写服务器端点一样**。

为此，我们需要编写一个用@WebSocketClient标注的新类：

```kotlin
@WebSocketClient(path = "/json")
class RichWebsocketClient {
    // Client code here.
}
```

在这个类中，我们用与服务器端点相同的方式编写注解方法，使用诸如@OnTextMessage，@OnOpen等注解：

```java
@OnTextMessage
void onMessage(String message, WebSocketClientConnection connection) {
    // Process message here
}
```

**这遵循与服务器端点关于方法参数和返回值的所有相同的规则**，只是如果我们想要访问连接详细信息，我们使用WebSocketClientConnection而不是WebSocketConnection。

一旦我们编写了客户端类，我们就可以通过注入的WebSocketConnector<T\>实例来使用它：

```java
@Inject
WebSocketConnector<RichWebsocketClient> connector;
```

使用这个，我们以与以前类似的方式创建连接，只是我们不需要提供任何回调，因为我们的客户端实例将为我们处理所有这些：

```java
WebSocketClientConnection connection = connector
    .baseUri(serverUrl)
    .connectAndAwait();
```

此时，我们有一个WebSocketClientConnection，可以像以前一样使用它。

如果我们需要访问我们的客户端实例，我们可以通过为适当的类注入一个Instance<T\>来实现：

```java
@Inject
Instance<RichWebsocketClient> clients;
```

但是，我们需要记住，这个客户端实例只有在正确的上下文中才可用，而且因为它都是异步的，所以我们需要确保某些事件已经发生。

## 5. 总结

在本文中，我们简要介绍了Quarkus中使用websockets-next扩展的WebSockets，我们了解了如何使用它编写服务器和客户端组件。在这里，我们只讨论了这个库的基础知识，但它能够处理许多更高级的场景。