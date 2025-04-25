---
layout: post
title:  每连接线程与每请求线程
category: java-net
copyright: java-net
excerpt: Java Net
---

## 1. 简介

在本教程中，**我们将比较两种常用的服务器[线程模型](https://www.baeldung.com/java-threading-models)：每个连接一个线程和每个请求一个线程**。

首先，我们将准确定义“连接”和“请求”。然后，我们将实现两个[基于套接字](https://www.baeldung.com/a-guide-to-java-sockets)的Java Web服务器，并遵循不同的范例。最后，我们将讨论一些关键要点。

## 2. 连接与请求线程模型

让我们从一些简洁的定义开始。

线程模型是指程序如何以及何时创建和同步线程以实现并发和多任务处理的方法，为了说明这一点，我们以客户端和服务器之间的HTTP连接为例，我们将请求视为在建立连接期间客户端向服务器发出的一次单次执行。

**当客户端需要与服务器通信时，它会实例化一个新的HTTP-over-TCP连接并发起新的HTTP-over-TCP请求**，为了避免开销，如果连接已存在，客户端可以重用同一连接发送另一个请求。这种机制被称为Keep-Alive，自HTTP 1.0以来就已存在，并在HTTP 1.1中成为默认行为。

理解了这个概念，我们现在可以介绍本文所比较的两种线程模型。

在下图中，我们可以看到，**如果我们使用“每个连接一个线程”范例，Web服务器将如何为每个连接使用一个线程；而当采用“每个请求一个线程”模型时，Web服务器将为每个请求使用一个线程**，无论该请求是否属于现有连接：

![](/assets/images/2025/javanetwork/javathreadperconnectionvsperrequest01.png)

在接下来的章节中，我们将分析这两种方法的优缺点，并查看一些使用套接字的代码示例。**这些示例将是真实案例的简化版本，为了尽可能简化代码，我们将避免引入在实际服务器架构中广泛使用的优化(例如[线程池](https://www.baeldung.com/thread-pool-java-and-guava))**。

## 3. 理解每个连接的线程数

采用“每个连接一个线程”的方法，每个客户端连接都会获得其专用的线程，同一个线程负责处理来自该连接的所有请求。

让我们通过构建一个简单的基于Java套接字的服务器来说明每个连接线程模型的工作原理：

```java
public class ThreadPerConnectionServer {

    private static final int PORT = 8080;

    public static void main(String[] args) {
        try (ServerSocket serverSocket = new ServerSocket(PORT)) {
            logger.info("Server started on port {}", PORT);
            while (!serverSocket.isClosed()) {
                try {
                    Socket newClient = serverSocket.accept();
                    logger.info("New client connected: {}", newClient.getInetAddress());
                    ClientConnection clientConnection = new ClientConnection(newClient);
                    new ThreadPerConnection(clientConnection).start();
                } catch (IOException e) {
                    logger.error("Error accepting connection", e);
                }
            }
        } catch (IOException e) {
            logger.error("Error starting server", e);
        }
    }
}
```

ClientConnection是一个简单的包装器，它实现了Closeable接口，并且包括我们将用来读取请求和写回响应的[BufferedReader](https://www.baeldung.com/java-buffered-reader)和[PrintWriter](https://www.baeldung.com/java-printstream-vs-printwriter)：

```java
public class ClientConnection implements Closeable {
    // ...

    public ClientConnection(Socket socket) throws IOException {
        this.socket = socket;
        this.reader = new BufferedReader(new InputStreamReader(socket.getInputStream()));
        this.writer = new PrintWriter(socket.getOutputStream(), true);
    }

    @Override
    public void close() throws IOException {
        try (Writer writer = this.writer; Reader reader = this.reader; Socket socket = this.socket) {
            // resources all closed when this block exits
        }
    }
}
```

ThreadPerConnectionServer在端口8080上创建一个ServerSocket，并重复调用accept()方法，该方法会阻塞执行，直到接收到新的连接为止。

当客户端连接时，服务器立即启动一个新的ThreadPerConnection线程：

```java
public class ThreadPerConnection extends Thread {
    // ...

    @Override
    public void run() {
        try (ClientConnection client = this.clientConnection) {
            String request;
            while ((request = client.getReader().readLine()) != null) {
                Thread.sleep(1000); // simulate server doing work
                logger.info("Processing request: {}", request);
                clientConnection.getWriter()
                        .println("HTTP/1.1 200 OK - Processed request: " + request);
                logger.info("Processed request: {}", request);
            }
        } catch (Exception e) {
            logger.error("Error processing request", e);
        }
    }
}
```

这个简单的实现读取来自客户端的输入，并将其与响应前缀一起回显。当此单个连接不再有请求传入时，套接字将利用[try-with-resource](https://www.baeldung.com/java-try-with-resources)语法自动关闭。每个连接都会获得其专用的线程，而while循环中的主线程仍然空闲，可以接收新的连接。

每个连接一个线程模型最显著的优势在于其极其简洁和易于实现，如果10个客户端创建了10个并发连接，Web服务器需要10个线程来同时处理所有连接，如果同一个线程处理同一个用户，应用程序就可以避免线程上下文切换。

## 4. 理解每个请求的线程数

使用每个请求一个线程的模型，即使使用的连接是持久的，也会使用不同的线程来处理每个请求。

与前一种情况一样，让我们看一个采用每请求一个线程的线程模型的基于Java套接字的服务器的简化示例：

```java
public class ThreadPerRequestServer {
    // ...

    public static void main(String[] args) {
        List<ClientSocket> clientSockets = new ArrayList<ClientSocket>();
        try (ServerSocket serverSocket = new ServerSocket(PORT)) {
            logger.info("Server started on port {}", PORT);
            while (!serverSocket.isClosed()) {
                acceptNewConnections(serverSocket, clientSockets);
                handleRequests(clientSockets);
            }
        } catch (IOException e) {
            logger.error("Server error: {}", e.getMessage());
        } finally {
            closeClientSockets(clientSockets);
        }
    }
}
```

这里，我们维护一个clientSockets列表，而不是像之前那样只维护一个。服务器会接收新的连接，直到服务器套接字关闭，并处理来自这些套接字的所有请求。当服务器套接字关闭时，我们还需要关闭所有仍处于活动状态的客户端套接字连接(如果有)。

首先，让我们定义接收新连接的方法：

```java
private static void acceptNewConnections(ServerSocket serverSocket, List<Socket> clientSockets)
        throws SocketException {
    serverSocket.setSoTimeout(100);
    try {
        Socket newClient = serverSocket.accept();
        ClientConnection clientConnection = new ClientConnection(newClient);
        clientConnections.add(clientConnection);
        logger.info("New client connected: {}", newClient.getInetAddress());
    } catch (IOException ignored) {
        // ignored
    }
}
```

理论上，接收新连接的方法和处理请求的方法应该在两个不同的主线程中执行。在这个简单的例子中，为了不阻塞唯一的主线程和执行流程，我们需要在服务器上设置一个较短的套接字超时时间。如果在100毫秒内没有收到任何连接，我们就认为没有可用的连接，并继续执行下一个用于处理请求的方法：

```java
private static void handleRequests(List<Socket> clientSockets) throws IOException {
    Iterator<ClientConnection> iterator = clientSockets.iterator();
    while (iterator.hasNext()) {
        Socket clientSocket = iterator.next();
        if (clientSocket.getSocket().isClosed()) {
            logger.info("Client disconnected: {}", clientSocket.getInetAddress());
            iterator.remove();
            continue;
        }
        try {
            BufferedReader reader = client.getReader();
            if (reader.ready()) {
                String request = reader.readLine();
                if (request != null) {
                    new ThreadPerRequest(client.getWriter(), request).start();
                }
            }
        } catch (IOException e) {
            logger.error("Error reading from client {}", client.getSocket()
                    .getInetAddress(), e);
        }
    }
}
```

在此方法中，对于每个包含要处理的新有效请求的连接，我们启动一个仅处理该单个请求的新线程：

```java
public class ThreadPerRequest extends Thread {
    // ...

    @Override
    public void run() {
        try {
            Thread.sleep(1000); // simulate server doing work
            logger.info("Processing request: {}", request);
            writer.println("HTTP/1.1 200 OK - Processed request: " + request);
            logger.info("Processed request: {}", request);
        } catch (Exception e) {
            logger.error("Error processing request: {}", e.getMessage());
        }
    }
}
```

在ThreadPerRequest中，我们不会关闭客户端连接，并且只处理一个请求。请求处理完成后，短暂的线程就会被关闭。请注意，在实际使用[线程池](https://www.baeldung.com/thread-pool-java-and-guava)的应用服务器中，请求结束后线程不会停止，而是会被重新用于处理另一个请求。

使用这种线程模型，服务器可能会创建大量线程，并在它们之间进行高上下文切换，但通常会更好地扩展：我们对并发连接没有上限。

## 5. 比较

下表比较了这两种方法，并考虑了服务器架构的一些决定性方面：

|    特征     |     每连接线程     |     每请求线程      |
|:---------:|:-------------:|:--------------:|
| **线程执行生命周期**  |长期存在，仅在连接关闭时暂停 |  短暂，请求处理后立即暂停  |
|  **上下文切换负载**  |  低，受并发连接数限制   |针对每个请求进行高速上下文切换 |
|   **可扩展性**    | 限制服务器可以创建的连接数 | 高效，可能具有很好的扩展性  |
|    **适应性**    |     已知连接数     |     不同的请求量     |

如果JVM提供的最大线程数为N，并且我们采用“每个连接一个线程”的模式，则最多会有N个并发客户端，额外的客户端需要等到一个客户端断开连接，这可能会花费大量时间。相反，如果我们采用“每个请求一个线程”的模式，则最多可以同时处理N个并发请求。额外的请求会一直排队，直到一个请求完成，这通常只需要很短的时间。

最后，**如果连接数已知，那么“每个连接一个线程”模型会非常有效：其实现简单，上下文切换频率低，效果显著。当处理不可预测的突发大量请求时，应该选择“每个请求一个线程”模型**。

## 6. 总结

在本文中，我们比较了两种常用的服务器线程模型，选择“每个连接线程”还是“每个请求线程”模型取决于应用程序的具体需求和预期的流量模式。通常，“每个连接线程”模型在已知数量的客户端情况下更简单且可预测，而“每个请求线程”模型在多变或高负载条件下则具有更高的可扩展性和灵活性。