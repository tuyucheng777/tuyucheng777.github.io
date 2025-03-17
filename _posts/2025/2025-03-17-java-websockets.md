---
layout: post
title:  WebSocket的Java API指南
category: java
copyright: java
excerpt: Java WebSocket
---

## 1. 概述

WebSocket通过提供双向、全双工、实时的客户端/服务器通信，为服务器和Web浏览器之间高效通信的限制提供了一种替代方案。服务器可以随时向客户端发送数据。由于它通过TCP运行，因此它还提供低延迟、低级通信，并减少了每条消息的开销。

在本教程中，我们将通过创建类似聊天的应用程序来探索WebSockets的Java API。

## 2. JSR 356

[JSR 356](https://jcp.org/en/jsr/detail?id=356)指定了一种API，Java开发人员可以使用该API在其应用程序中集成WebSockets，无论是在服务器端，还是在Java客户端。

此Java API提供服务器端和客户端组件：

- **服务器**：jakarta.websocket.server包中的所有内容
- **客户端**：jakarta.websocket包的内容，包括客户端API，以及服务器和客户端的通用库

## 3. 使用WebSocket建立聊天

让我们构建一个非常简单的聊天类应用程序，任何用户都可以从任何浏览器打开聊天，输入自己的姓名，登录聊天，然后开始与聊天连接的每个人交流。

我们首先向pom.xml文件添加最新的依赖项：

```xml
<dependency>
    <groupId>jakarta.websocket</groupId>
    <artifactId>jakarta.websocket-api</artifactId>
    <version>2.2.0</version>
</dependency>
```

可以在[这里](https://mvnrepository.com/artifact/jakarta.websocket/jakarta.websocket-api)找到最新版本。

为了将Java对象转换为JSON表示形式，反之亦然，我们将使用Gson：

```xml
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.0</version>
</dependency>
```

最新版本可在[Maven Central](https://mvnrepository.com/search?q=com.google.code.gson)仓库中找到。

### 3.1 端点配置

配置端点有两种方式：基于注解和基于扩展。我们可以扩展jakarta.websocket.Endpoint类，也可以使用专用的方法级注解。由于注解模型比编程模型更能产生更简洁的代码，因此注解已成为编码的常规选择。我们将使用以下注解来处理WebSocket端点生命周期事件：

- @ServerEndpoint：如果用@ServerEndpoint修饰，容器可确保该类作为监听特定URI空间的WebSocket服务器的可用性。
- @ClientEndpoint：使用此注解修饰的类被视为WebSocket客户端。
- @OnOpen：当启动新的WebSocket连接时，容器将调用带有@OnOpen的Java方法。
- @OnMessage：当消息发送到端点时，用@OnMessage标注的Java方法从WebSocket容器接收信息。
- @OnError：当通信出现问题时，将调用带有@OnError的方法。
- @OnClose：用于修饰容器在WebSocket连接关闭时调用的Java方法

### 3.2 编写服务器端点

我们将通过使用@ServerEndpoint注解来声明Java类WebSocket服务器端点，我们还将指定部署端点的URI，我们定义相对于服务器容器根的URI，并且它必须以正斜杠开头：

```java
@ServerEndpoint(value = "/chat/{username}")
public class ChatEndpoint {

    @OnOpen
    public void onOpen(Session session) throws IOException {
        // Get session and WebSocket connection
    }

    @OnMessage
    public void onMessage(Session session, Message message) throws IOException {
        // Handle new messages
    }

    @OnClose
    public void onClose(Session session) throws IOException {
        // WebSocket connection closes
    }

    @OnError
    public void onError(Session session, Throwable throwable) {
        // Do error handling here
    }
}
```

上面的代码是我们类似聊天的应用程序的服务器端点框架，如我们所见，我们有四个注解映射到各自的方法。下面，我们可以看到这些方法的实现：

```java
@ServerEndpoint(value="/chat/{username}")
public class ChatEndpoint {

    private Session session;
    private static Set<ChatEndpoint> chatEndpoints = new CopyOnWriteArraySet<>();
    private static HashMap<String, String> users = new HashMap<>();

    @OnOpen
    public void onOpen(Session session, @PathParam("username") String username) throws IOException {
        this.session = session;
        chatEndpoints.add(this);
        users.put(session.getId(), username);

        Message message = new Message();
        message.setFrom(username);
        message.setContent("Connected!");
        broadcast(message);
    }

    @OnMessage
    public void onMessage(Session session, Message message) throws IOException {
        message.setFrom(users.get(session.getId()));
        broadcast(message);
    }

    @OnClose
    public void onClose(Session session) throws IOException {
        chatEndpoints.remove(this);
        Message message = new Message();
        message.setFrom(users.get(session.getId()));
        message.setContent("Disconnected!");
        broadcast(message);
    }

    @OnError
    public void onError(Session session, Throwable throwable) {
        // Do error handling here
    }

    private static void broadcast(Message message)
            throws IOException, EncodeException {

        chatEndpoints.forEach(endpoint -> {
            synchronized (endpoint) {
                try {
                    endpoint.session.getBasicRemote().
                            sendObject(message);
                } catch (IOException | EncodeException e) {
                    e.printStackTrace();
                }
            }
        });
    }
}
```

当新用户登录时，(@OnOpen)会立即映射到活跃用户的数据结构，然后创建一条消息并使用broadcast方法发送到所有端点。

每当任何连接的用户发送新消息(@OnMessage)时，也会使用此方法；这是聊天的主要目的。

如果在某个时刻发生错误，带有注解@OnError的方法会处理它。我们可以使用此方法记录有关错误的信息，并清除端点。

最后，当用户不再连接到聊天时，方法@OnClose将清除端点，并向所有用户广播用户已断开连接。

## 4. 消息类型

WebSocket规范支持两种在线数据格式：文本和二进制。API支持这两种格式，并增加了使用Java对象的功能以及健康检查消息(ping-pong)，如规范中所定义：

- 文本：任何文本数据(java.lang.String、原始类型或其等效的包装类)
- 二进制：由java.nio.ByteBuffer或byte[\](字节数组)表示的二进制数据(例如音频、图像等)
- Java对象：该API允许使用代码中的原生(Java对象)表示，并使用自定义转换器(编码器/解码器)将它们转换为WebSocket协议允许的兼容在线格式(文本、二进制)
- Ping-Pong：jakarta.websocket.PongMessage是WebSocket对等体响应健康检查(ping)请求而发送的确认信息

对于我们的应用程序，我们将使用Java对象，我们将创建用于编码和解码消息的类。

### 4.1 编码器

编码器接收Java对象并生成适合作为消息传输的典型表示，例如JSON、XML或二进制表示。可以通过实现Encoder.Text<T\>或Encoder.Binary<T\>接口来使用编码器。

在下面的代码中，我们将定义要编码的Message类，并在方法编码中，使用Gson将Java对象编码为JSON：

```java
public class Message {
    private String from;
    private String to;
    private String content;

    //standard constructors, getters, setters
}

public class MessageEncoder implements Encoder.Text<Message> {

    private static Gson gson = new Gson();

    @Override
    public String encode(Message message) throws EncodeException {
        return gson.toJson(message);
    }

    @Override
    public void init(EndpointConfig endpointConfig) {
        // Custom initialization logic
    }

    @Override
    public void destroy() {
        // Close resources
    }
}
```

### 4.2 解码器

解码器与编码器相反，我们用它将数据转换回Java对象。解码器可以使用Decoder.Text<T\>或Decoder.Binary<T\>接口实现。

正如我们在编码器中看到的，decode方法是我们从发送到端点的消息中检索JSON，并使用Gson将其转换为名为Message的Java类：

```java
public class MessageDecoder implements Decoder.Text<Message> {

    private static Gson gson = new Gson();

    @Override
    public Message decode(String s) throws DecodeException {
        return gson.fromJson(s, Message.class);
    }

    @Override
    public boolean willDecode(String s) {
        return (s != null);
    }

    @Override
    public void init(EndpointConfig endpointConfig) {
        // Custom initialization logic
    }

    @Override
    public void destroy() {
        // Close resources
    }
}
```

### 4.3 在服务器端点设置编码器和解码器

让我们通过在类级别注解@ServerEndpoint中添加为编码和解码数据创建的类来将所有内容放在一起：

```java
@ServerEndpoint( 
    value="/chat/{username}", 
    decoders = MessageDecoder.class, 
    encoders = MessageEncoder.class )
```

每次消息发送到端点时，它们都会自动转换为JSON或Java对象。

## 5. 总结

在本文中，我们分析了WebSockets的Java API，并了解了它如何帮助我们构建应用程序，例如此实时聊天。

我们讨论了创建端点的两种编程模型：注解和编程，然后我们使用注解模型为我们的应用程序定义了一个端点以及生命周期方法。

此外，为了能够在服务器和客户端之间来回通信，我们证明我们需要编码器和解码器将Java对象转换为JSON，反之亦然。

JSR 356 API非常简单，基于注解的编程模型使得构建WebSocket应用程序变得非常容易。

要运行我们在示例中构建的应用程序，需要做的就是在Web服务器中部署war文件，然后转到URL：[http://localhost:8080/java-websocket/](http://localhost:8080/java-websocket/)。