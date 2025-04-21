---
layout: post
title:  JeroMQ简介
category: messaging
copyright: messaging
excerpt: JeroMQ
---

## 1. 简介

**在本文中，我们将介绍[JeroMQ](https://github.com/zeromq/jeromq)，它是[ZeroMQ](https://zeromq.org/)的纯Java实现**。我们将了解它是什么，以及它能在我们的应用程序中为我们做些什么。

## 2. 什么是ZeroMQ？

**ZeroMQ是一个消息传递基础设施，无需搭建任何实际的基础设施服务**。我们不需要像[ActiveMQ](https://www.baeldung.com/spring-remoting-jms#starting-an-apache-activemq-broker)或[Kafka](https://www.baeldung.com/spring-kafka#installation-and-setup)等其他实现那样单独部署消息代理，相反，我们应用程序中的ZeroMQ依赖能够为我们完成所有这些工作。

那么，我们能用它做什么呢？我们可以实现通常想要的所有标准消息传递模式：

- 请求/响应
- 发布/订阅
- 同步与异步
- 以及其他

### 2.1 套接字

**ZeroMQ使用套接字的概念，套接字的概念与我们在底层网络编程中使用的[套接字](https://www.baeldung.com/a-guide-to-java-sockets)非常相似**。

所有套接字都有一个类型，我们将在本文中介绍其中一些类型。它们要么监听来自其他套接字的连接，要么打开与其他套接字的连接。一旦一对套接字连接成功，我们就可以开始在它们之间发送消息了。需要注意的是，只有特定的套接字组合才能一起使用，具体取决于我们想要实现的具体目标。

JeroMQ还支持套接字之间的几种不同的传输机制，例如，常见的包括：

- tcp://<host\>:<port\>：使用TCP/IP网络在套接字之间发送消息，这允许套接字位于不同的进程和不同的主机上，但也会带来一些网络相关的可靠性问题。
- ipc://<endpoint\>：使用系统相关的机制在套接字之间发送消息，这允许套接字属于不同的进程，但它们必须位于同一主机上，并且系统可能对哪些进程可以通信存在其他限制。
- inproc://<name\>：允许同一进程中的套接字之间进行通信，具体来说，它们必须位于同一个JeroMQ上下文中。

具体选择哪种传输方式取决于我们的需求，根据具体的传输方式和套接字类型，我们还可以用它来与其他ZeroMQ实现(包括其他语言的实现)进行通信。

## 3. 入门

JeroMQ是ZeroMQ的纯Java实现，让我们快速了解如何在我们的应用程序中使用它。

### 3.1 依赖

**我们需要做的第一件事是添加依赖**：

```xml
<dependency>
    <groupId>org.zeromq</groupId>
    <artifactId>jeromq</artifactId>
    <version>0.5.3</version>
</dependency>
```

可以在[Maven Central](https://mvnrepository.com/artifact/org.zeromq/jeromq)中找到最新版本。

### 3.2 JeroMQ上下文

**在使用JeroMQ进行任何操作之前，我们需要设置一个上下文，它是ZContext类的一个实例，负责管理所有内容**。

创建上下文没有什么特别之处，我们只需使用new ZContext()即可，我们还必须确保正确关闭它-使用close()方法，这确保我们正确释放所有网络资源。

我们使用的实例的生命周期必须至少与我们所做的任何事情一样长，因此我们需要确保它是在应用程序开始时创建的，并且在结束之前不会关闭。

如果我们编写一个标准的Java应用程序，可以简单地使用[try-with-resources模式](https://www.baeldung.com/java-try-with-resources)。如果我们使用类似Spring的框架，那么我们可以将其设置为一个配置了[destroy方法的Bean](https://www.baeldung.com/spring-shutdown-callbacks#3-declaring-a-bean-destroy-method)。此外，我们还可以根据所使用的框架的需要，使用其他模式。

### 3.3 创建套接字

**一旦有了上下文，我们就可以用它来创建套接字，这些套接字将成为我们所有消息传递的基础**。

我们使用ZContext.createSocket()方法创建一个套接字，并提供我们想要使用的套接字类型。创建完成后，我们通常需要调用ZMQ.Socket.bind()来监听连接，或者调用ZMQ.Socket.connect()来打开与另一个套接字的连接。

至此，我们已经准备好使用套接字了，使用send()等方法发送消息，使用recv()等方法接收消息。

完成后，我们可以关闭套接字以断开连接，我们可以通过显式调用Socket.close()来实现，或者关闭ZContext来实现，这样，所有从中创建的套接字都会自动关闭。

注意，套接字不是线程安全的，我们可以在线程之间传递它们，但重要的是，同一时间只能有一个线程访问它们。

## 4. 请求/响应消息

**让我们从一个简单的请求/响应设置开始**，我们首先需要一个服务器，这是监听传入连接的部分：

```java
try (ZContext context = new ZContext()) {
    ZMQ.Socket socket = context.createSocket(SocketType.REP);
    socket.bind("tcp://*:5555");

    byte[] reply = socket.recv();
    // Do something here.

    String response = "world";
    socket.send(response.getBytes(ZMQ.CHARSET), 0);
}
```

这里我们创建了一个REP(Reply的缩写)类型的新套接字，我们可以指示它开始监听给定的地址，然后进入一个循环，在这个循环中，我们从套接字接收下一条消息，对其进行处理，然后发送响应。

接下来，我们需要一个客户端。客户端负责与服务器建立连接，它也是发送初始请求的一方-我们的服务器只能回复它收到的请求：

```java
try (ZContext context = new ZContext()) {
    ZMQ.Socket socket = context.createSocket(SocketType.REQ);
    socket.connect("tcp://localhost:5555");

    String request = "Hello";
    socket.send(request.getBytes(ZMQ.CHARSET), 0);

    byte[] reply = socket.recv();
}
```

和之前一样，我们创建一个新的套接字，只是这次它的类型是REQ(请求的缩写)，然后，我们指示它在发送消息和接收响应之前连接到某个地方的另一个套接字。

REQ端和REP端的主要区别在于它们何时允许发送消息，REQ端可以随时发送消息，而REP端只能在收到消息后才发送消息-因此称为请求(Request)和响应(Response)。

### 4.1 多客户端

我们已经了解了如何让单个客户端向单个服务器发送消息，但是，如果我们想要多个客户端呢？

好消息是，**JeroMQ允许任意数量的客户端连接到同一个服务器地址，并且它会为我们处理所有的网络需求**。

但是，这到底是怎么实现的呢？我们的服务器没有任何内容表明应该将响应发送给哪个客户端。这是因为我们不需要它，JeroMQ会帮我们追踪这一切，当服务器调用send()时，消息会被发送到我们上次收到消息的客户端，这使得我们的代码不需要关心这些。

缺点是我们的处理必须完全单线程，由于这种工作方式，我们需要完成一条消息的所有处理，并在收到下一条消息之前发送回复。在某些情况下，这没问题，但通常这会成为一个很大的瓶颈。

### 4.2 异步处理

**那么，如果我们希望异步处理传入的请求并无序发送响应呢**？使用REQ/REP设置很难做到这一点，因为每个响应都直接发送到最后收到的请求。

相反，我们可以使用不同类型的套接字来实现这一点-ROUTER。它的工作原理与REP非常相似，只是我们需要负责指明消息的接收者是谁。

让我们看一下服务器组件：

```java
try (ZContext context = new ZContext()) {
    ZMQ.Socket broker = context.createSocket(SocketType.ROUTER);
    broker.bind("tcp://*:5555");

    String identity = broker.recvStr();
    broker.recv(); //  Envelope delimiter
    String message = broker.recvStr(0);
    // Do something here.

    broker.sendMore(identity);
    broker.sendMore("");
    broker.send("Hello back");
}
```

这看起来非常相似，但并不完全相同。我们将套接字类型设置为ROUTER而不是REP，此套接字类型允许服务器通过识别客户端标识来将消息路由到特定客户端。

当我们在这里接收消息时，我们实际上会收到三部分不同的数据。首先，我们会收到客户端的标识信息，然后是消息分隔符，最后是实际的消息。

同样，当我们发送消息时，我们也需要这样做。我们发送消息的客户端标识，然后是消息分隔符(可以是任何字符串)，最后是实际的消息。

我们来看一下客户端：

```java
try (ZContext context = new ZContext()) {
    ZMQ.Socket worker = context.createSocket(SocketType.REQ);
    worker.setIdentity(Thread.currentThread().getName().getBytes(ZMQ.CHARSET));

    worker.connect("tcp://localhost:5555");
    worker.send("Hello " + 
    String workload = worker.recvStr();
    // Do something with the response.
}
```

这几乎和我们之前的客户端一模一样，**我们现在赋予了客户端一个标识，以便服务器能够识别哪个客户端是哪个。如果没有标识，服务器将无法将响应转发给正确的客户端**。除此之外，这和我们之前看到的完全一样。

由于我们的服务器现在可以指示消息发往哪个客户端，因此我们可以一次处理多个请求——例如，使用[ExecutorService](https://www.baeldung.com/java-executor-service-tutorial)。唯一的要求是，我们永远不会有多个线程同时访问套接字。

## 5. 发布/订阅消息

到目前为止，我们已经看到了客户端发送初始请求，然后服务器返回响应的情况。**如果我们想让服务器只广播客户端可以消费的事件，该怎么办**？

我们可以使用发布/订阅模式来实现这一点，服务器发布消息，然后订阅者消费消息。

首先我们需要有发布者：

```java
try (ZContext context = new ZContext()) {
    ZMQ.Socket pub = context.createSocket(SocketType.PUB);
    pub.bind("tcp://*:5555");

    // Wait until something happens.
    pub.send("Hello");
}
```

这看起来非常简单，但这是因为JeroMQ为我们处理了大部分复杂的事情，我们所做的就是创建一个PUB类型的套接字(PUB的缩写)，监听连接，然后向其中发送消息。

接下来，我们需要一个订阅者：

```java
try (ZContext context = new ZContext()) {
    ZMQ.Socket sub = context.createSocket(SocketType.SUB);
    sub.connect("tcp://localhost:5555");

    sub.subscribe("".getBytes());

    String message = sub.recvStr();
}
```

这稍微复杂一点，但也没那么复杂，我们创建一个SUB类型的套接字(Subscribe的缩写)，并将其连接到我们的发布者。然后我们需要订阅消息，这需要一组字节作为所有传入消息的前缀，或者使用空的字节集来订阅所有消息。

完成此操作后，我们就可以接收消息了，我们会收到订阅者发送的任何相应消息，**请注意，我们只能接收订阅后发送的消息-在此之前发送的任何消息都将丢失**。

### 5.1 多客户端

和之前一样，如果我们想要多个客户端，也可以。**每个连接的订阅者都会收到发布者发送的所有相应消息，这意味着它就像一个多播-例如，类似于JMS主题(Topic)，而不是JMS队列(Queue)**。

我们还可以允许不同的客户端订阅不同的内容，这意味着它们各自只会收到广播消息中合适的子集，所有这些都完全符合我们的预期，我们无需进行任何额外的操作。

### 5.2 异步处理

**这里我们遇到的一个问题是recv()方法会阻塞，直到有消息可用为止**。如果我们的订阅者只是等待来自此套接字的消息，然后对其做出反应，那么这样做没有问题。但是，如果我们想让订阅者执行其他操作(例如等待多个套接字)，那么这种方法就行不通了。

我们使用的recv()或recvStr()方法有一个可选的签名，允许设置一些标志。如果设置了标志ZMQ.DONTWAIT，该方法将立即返回，而不是阻塞。如果没有消息可以读取，则返回null。

这将允许我们轮询套接字以查看是否有等待的消息，如果有则处理它，如果没有，则在此期间做其他事情。

## 6. 总结

以上我们简要介绍了JeroMQ的功能，但是，它的功能远不止这些。