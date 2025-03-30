---
layout: post
title:  AsyncHttpClient中的WebSockets
category: libraries
copyright: libraries
excerpt: AsyncHttpClient
---

## 1. 简介

AsyncHttpClient(AHC)是一个基于Netty的库，旨在轻松执行异步HTTP调用并通过WebSocket协议进行通信。

在本快速教程中，我们将了解如何启动WebSocket连接、发送数据以及处理各种控制帧。

## 2. 设置

最新版本的库可以在[Maven Central](https://mvnrepository.com/artifact/org.asynchttpclient/async-http-client)上找到。我们需要小心使用groupId为org.asynchttpclient的依赖，而不是groupId为com.ning的依赖：

```xml
<dependency>
    <groupId>org.asynchttpclient</groupId>
    <artifactId>async-http-client</artifactId>
    <version>2.2.0</version>
</dependency>
```

## 3. WebSocket客户端配置

**要创建WebSocket客户端，我们首先必须获取一个HTTP客户端(如[本文](https://www.baeldung.com/async-http-client)所示)，并升级它以支持WebSocket协议**。

WebSocket协议升级的处理由WebSocketUpgradeHandler类完成，该类实现了AsyncHandler接口，还为我们提供了一个构建器：

```java
WebSocketUpgradeHandler.Builder upgradeHandlerBuilder = new WebSocketUpgradeHandler.Builder();
WebSocketUpgradeHandler wsHandler = upgradeHandlerBuilder
        .addWebSocketListener(new WebSocketListener() {
            @Override
            public void onOpen(WebSocket websocket) {
                // WebSocket connection opened
            }

            @Override
            public void onClose(WebSocket websocket, int code, String reason) {
                // WebSocket connection closed
            }

            @Override
            public void onError(Throwable t) {
                // WebSocket connection error
            }
        }).build();
```

为了获取WebSocket连接对象，我们使用标准AsyncHttpClient创建一个具有首选连接详细信息(如标头、查询参数或超时)的HTTP请求：

```java
WebSocket webSocketClient = Dsl.asyncHttpClient()
    .prepareGet("ws://localhost:5590/websocket")
    .addHeader("header_name", "header_value")
    .addQueryParam("key", "value")
    .setRequestTimeout(5000)
    .execute(wsHandler)
    .get();
```

## 4. 发送数据

使用WebSocket对象，**我们可以使用isOpen()方法检查连接是否成功打开**。一旦我们打开了连接，我们就可以使用sendTextFrame()和sendBinaryFrame()方法发送带有字符串或二进制有效负载的数据帧：

```java
if (webSocket.isOpen()) {
    webSocket.sendTextFrame("test message");
    webSocket.sendBinaryFrame(new byte[]{'t', 'e', 's', 't'});
}
```

## 5. 处理控制帧

WebSocket协议支持三种类型的控制帧：ping、pong、close。

**ping和pong帧主要用于实现连接的“Keep Alive”机制，我们可以使用sendPingFrame()和sendPongFrame()方法发送这些帧**：

```java
webSocket.sendPingFrame();
webSocket.sendPongFrame();
```

关闭现有连接是通过使用sendCloseFrame()方法发送关闭帧来完成的，其中我们可以以文本的形式提供状态码和关闭连接的原因：

```java
webSocket.sendCloseFrame(404, "Forbidden");
```

## 6. 总结

除了提供一种执行异步HTTP请求的简便方法之外，还支持WebSocket协议，这使得AHC成为一个非常强大的库。