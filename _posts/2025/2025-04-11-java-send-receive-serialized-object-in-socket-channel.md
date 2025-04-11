---
layout: post
title:  在套接字通道中发送和接收序列化对象
category: java-nio
copyright: java-nio
excerpt: NIO
---

## 1. 简介

在本教程中，我们将探索如何使用Java的[java.nio](https://docs.oracle.com/en/java/javase/22/core/java-nio.html)包中的[SocketChannel](https://www.baeldung.com/java-nio2-async-socket-channel)发送和接收序列化对象，这种方法可以在客户端和服务器之间实现高效、无阻塞的网络通信。

## 2. 理解序列化

[序列化](https://www.baeldung.com/java-serialization)是将对象转换为字节流的过程，使其能够通过网络传输或存储在文件中。序列化与套接字通道结合使用时，可以实现应用程序之间复杂数据结构的无缝传输，对于必须通过网络交换对象的分布式系统来说，这项技术至关重要。

### 2.1 Java序列化中的关键类

ObjectOutputStream和ObjectInputStream类在Java序列化中至关重要，它们处理对象和字节流之间的转换：

- ObjectOutputStream用于将对象序列化为字节序列，例如，当通过网络发送Message对象时，ObjectOutputStream会将对象的字段和元数据写入输出流。
- ObjectInputStream从接收端的字节流中重建对象。

## 3. 理解套接字通道

套接字通道是Java NIO包的一部分，它提供了一种灵活、可扩展的传统套接字通信替代方案。它们支持阻塞和非阻塞模式，非常适合高效处理多个连接的高性能网络应用程序。

套接字通道对于创建客户端-服务器通信系统至关重要，客户端可以通过TCP/IP连接到服务器，通过使用SocketChannel，我们可以实现异步通信，从而实现更佳性能和更低延迟。

### 3.1 Socket Channel的关键组件

套接字通道有3个关键组件：

- ServerSocketChannel：监听传入的TCP连接，它绑定到特定端口并等待客户端连接。
- SocketChannel：表示客户端与服务器之间的连接，它支持阻塞和非阻塞模式。
- Selector：用于使用单个线程监控多个套接字通道，它有助于处理诸如传入连接或数据可读之类的事件，从而减少为每个连接设置专用线程的开销。

## 4. 设置服务器和客户端

在实现服务器和客户端之前，让我们首先定义一个要通过套接字发送的示例对象。在Java中，对象必须实现Serializable接口才能转换为字节流，这对于通过网络连接传输对象是必需的。

### 4.1 创建可序列化对象

让我们编写MyObject类，作为我们将通过SocketChannel发送和接收的可序列化对象的示例：

```java
class MyObject implements Serializable {
    private String name;
    private int age;

    public MyObject(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }
}
```

MyObject类实现了Serializable接口，这是将对象转换为字节流并通过套接字连接传输所必需的。

### 4.2 实现服务器

在服务器端，我们将使用ServerSocketChannel来监听传入的客户端连接并处理接收到的序列化对象：

```java
private static final int PORT = 6000;

try (ServerSocketChannel serverSocket = ServerSocketChannel.open()) {
    serverSocket.bind(new InetSocketAddress(PORT));
    logger.info("Server is listening on port " + PORT);

    while (true) {
        try (SocketChannel clientSocket = serverSocket.accept()) {
            System.out.println("Client connected...");
            // To receive object here
        }
    }
} catch (IOException e) {
    // handle exception
}
```

服务器在6000端口监听客户端连接，一旦收到客户端请求，服务器将等待接收对象。

### 4.3 实现客户端

客户端将创建一个MyObject实例，对其进行序列化，然后将其发送到服务器，我们使用SocketChannel连接到服务器并传输对象：

```java
private static final String SERVER_ADDRESS = "localhost";
private static final int SERVER_PORT = 6000;

try (SocketChannel socketChannel = SocketChannel.open()) {
    socketChannel.connect(new InetSocketAddress(SERVER_ADDRESS, SERVER_PORT));
    logger.info("Connected to the server...");

    // To send object here
} catch (IOException e) {
   // handle exception
}
```

此代码连接到在本地主机端口6000上运行的服务器，它将序列化的对象发送到服务器。

## 5. 序列化并发送对象

要通过SocketChannel发送对象，我们首先需要将其序列化为字节数组。由于SocketChannel只支持[ByteBuffer](https://www.baeldung.com/java-bytebuffer)类型，因此在通过网络发送之前，我们需要将对象转换为字节数组，并将其包装在ByteBuffer中：

```java
void sendObject(SocketChannel channel, MyObject obj) throws IOException {
    ByteArrayOutputStream byteStream = new ByteArrayOutputStream();
    try (ObjectOutputStream objOut = new ObjectOutputStream(byteStream)) {
        objOut.writeObject(obj);
    }
    byte[] bytes = byteStream.toByteArray();

    ByteBuffer buffer = ByteBuffer.wrap(bytes);
    while (buffer.hasRemaining()) {
        channel.write(buffer);
    }
}
```

这里，我们首先将MyObject序列化为字节数组，然后将其包装到ByteBuffer中，并写入套接字通道。接下来，我们从客户端发送该对象：

```java
try (SocketChannel socketChannel = SocketChannel.open()) {
    socketChannel.connect(new InetSocketAddress(SERVER_ADDRESS, SERVER_PORT));
    MyObject objectToSend = new MyObject("Alice", 25);
    sendObject(socketChannel, objectToSend); // Serialize and send
}
```

在此示例中，客户端连接到服务器并发送一个序列化的MyObject，其中包含名称“Alice”和年龄25。

## 6. 接收和反序列化对象

在服务器端，我们从SocketChannel读取字节并将其反序列化为MyObject实例：

```java
MyObject receiveObject(SocketChannel channel) throws IOException, ClassNotFoundException {
    ByteBuffer buffer = ByteBuffer.allocate(1024);
    ByteArrayOutputStream byteStream = new ByteArrayOutputStream();

    while (channel.read(buffer) > 0) {
        buffer.flip();
        byteStream.write(buffer.array(), 0, buffer.limit());
        buffer.clear();
    }

    byte[] bytes = byteStream.toByteArray();
    try (ObjectInputStream objIn = new ObjectInputStream(new ByteArrayInputStream(bytes))) {
        return (MyObject) objIn.readObject();
    }
}
```

我们从SocketChannel读取字节到ByteBuffer中，并将它们存储在[ByteArrayOutputStream](https://www.baeldung.com/java-outputstream)中，然后反序列化字节数组并返回原始对象。之后，我们就可以在服务器上接收该对象了：

```java
try (SocketChannel clientSocket = serverSocket.accept()) {
    MyObject receivedObject = receiveObject(clientSocket);
    logger.info("Received Object - Name: " + receivedObject.getName());
}
```

## 7. 处理多个客户端

为了并发处理多个客户端，我们可以使用[Selector](https://www.baeldung.com/java-nio-selector)以非阻塞模式管理多个套接字通道，这确保服务器可以同时处理多个连接，而不会阻塞任何单个连接：

```java
class NonBlockingServer {
    private static final int PORT = 6000;

    public static void main(String[] args) throws IOException {
        ServerSocketChannel serverChannel = ServerSocketChannel.open();
        serverChannel.bind(new InetSocketAddress(PORT));
        serverChannel.configureBlocking(false);

        Selector selector = Selector.open();
        serverChannel.register(selector, SelectionKey.OP_ACCEPT);

        while (true) {
            selector.select();
            Set<SelectionKey> selectedKeys = selector.selectedKeys();
            Iterator<SelectionKey> iter = selectedKeys.iterator();

            while (iter.hasNext()) {
                SelectionKey key = iter.next();
                iter.remove();

                if (key.isAcceptable()) {
                    SocketChannel client = serverChannel.accept();
                    client.configureBlocking(false);
                    client.register(selector, SelectionKey.OP_READ);
                } else if (key.isReadable()) {
                    SocketChannel client = (SocketChannel) key.channel();
                    MyObject obj = receiveObject(client);
                    System.out.println("Received from client: " + obj.getName());
                }
            }
        }
    }
}
```

在此示例中，configureBlocking(false)将服务器设置为非阻塞模式，这意味着诸如accept()和read()之类的操作在等待事件时不会阻塞执行，这使得服务器可以继续处理其他任务，而不是卡在等待客户端连接上。

接下来，我们使用Selector监听多个通道上的事件，它会检测何时有新的连接(OP_ACCEPT)或传入数据(OP_READ)可用，并进行相应的处理，确保通信顺畅且可扩展。

## 8. 测试用例

让我们验证通过SocketChannel进行的对象的序列化和反序列化：

```java
@Test
void givenClientSendsObject_whenServerReceives_thenDataMatches() throws Exception {
    try (ServerSocketChannel server = ServerSocketChannel.open().bind(new InetSocketAddress(6000))) {
        int port = ((InetSocketAddress) server.getLocalAddress()).getPort();
        ExecutorService executor = Executors.newSingleThreadExecutor();
        Future<MyObject> future = executor.submit(() -> {
            try (SocketChannel client = server.accept();
                 ObjectInputStream objIn = new ObjectInputStream(Channels.newInputStream(client))) {
                return (MyObject) objIn.readObject();
            }
        });

        try (SocketChannel client = SocketChannel.open()) {
            client.configureBlocking(true);
            client.connect(new InetSocketAddress("localhost", 6000));

            while (!client.finishConnect()) {
                Thread.sleep(10);
            }

            try (ObjectOutputStream objOut = new ObjectOutputStream(Channels.newOutputStream(client))) {
                objOut.writeObject(new MyObject("Test User", 25));
            }
        }

        MyObject received = future.get(2, TimeUnit.SECONDS);
        assertEquals("Test User", received.getName());
        assertEquals(25, received.getAge());
        executor.shutdown();
    }
}
```

此测试验证序列化和反序列化过程是否通过SocketChannel正确运行。

## 9. 总结

在本文中，我们演示了如何使用Java NIO的SocketChannel来构建客户端-服务器系统，以发送和接收序列化对象。通过使用序列化和非阻塞I/O，我们可以通过网络在系统之间高效地传输复杂的数据结构。