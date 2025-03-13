---
layout: post
title:  使用Netty的HTTP服务器
category: netty
copyright: netty
excerpt: Netty
---

## 1. 概述

在本教程中，我们将**使用Netty通过[HTTP](https://www.w3.org/Protocols/HTTP/1.1/draft-ietf-http-v11-spec-01.html)实现一个简单的大写服务器**，Netty是一个异步框架，让我们可以灵活地用Java开发网络应用程序。

## 2. 服务器

在开始之前，我们应该了解[Netty的基本概念](https://www.baeldung.com/netty#core-concepts)，例如通道、处理程序、编码器和解码器。

这里我们将直接进入启动服务器，这与简单协议服务器基本相同：

```java
public class HttpServer {

    private int port;
    private static Logger logger = LoggerFactory.getLogger(HttpServer.class);

    // constructor

    // main method, same as simple protocol server

    public void run() throws Exception {
        // ...
        ServerBootstrap b = new ServerBootstrap();
        b.group(bossGroup, workerGroup)
            .channel(NioServerSocketChannel.class)
            .handler(new LoggingHandler(LogLevel.INFO))
            .childHandler(new ChannelInitializer() {
                @Override
                protected void initChannel(SocketChannel ch) throws Exception {
                    ChannelPipeline p = ch.pipeline();
                    p.addLast(new HttpRequestDecoder());
                    p.addLast(new HttpResponseEncoder());
                    p.addLast(new CustomHttpServerHandler());
                }
            });
        // ...
    }
}

```

因此，**这里只有childHandler与我们想要实现的协议不同**，对我们来说是HTTP。

我们向服务器的管道添加了三个处理程序：

1. Netty的[HttpResponseEncoder](https://netty.io/4.0/api/io/netty/handler/codec/http/HttpResponseEncoder.html)–用于序列化
2. Netty的[HttpRequestDecoder](https://netty.io/4.0/api/io/netty/handler/codec/http/HttpRequestDecoder.html)–用于反序列化
3. 我们自己的CustomHttpServerHandler–用于定义服务器的行为

接下来我们详细看看最后一个处理程序。

## 3. CustomHttpServerHandler

我们的自定义处理程序的作用是处理传入数据并发送响应。

让我们分解它来了解它的工作原理。

### 3.1 处理程序的结构

CustomHttpServerHandler扩展了Netty的抽象SimpleChannelInboundHandler并实现了其生命周期方法：

```java
public class CustomHttpServerHandler extends SimpleChannelInboundHandler {
    private HttpRequest request;
    StringBuilder responseData = new StringBuilder();

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) {
        ctx.flush();
    }

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, Object msg) {
       // implementation to follow
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        cause.printStackTrace();
        ctx.close();
    }
}
```

正如方法名称所示，channelReadComplete会在通道中的最后一条消息被使用后刷新处理程序上下文，以便可用于下一条传入消息。exceptionCaught方法用于处理任何异常。

到目前为止，我们看到的只是样板代码。

现在让我们继续讨论有趣的东西，即channelRead0的实现。

### 3.2 读取通道

我们的用例很简单，服务器只会将请求主体和查询参数(如果有)转换为大写。这里要注意在响应中反映请求数据-我们这样做只是为了演示，以了解如何使用Netty实现HTTP服务器。

在这里，**我们将消费消息或请求，并按照[协议的建议](https://www.w3.org/Protocols/HTTP/1.1/draft-ietf-http-v11-spec-01.html#Response)设置其响应**(请注意，RequestUtils是我们稍后将编写的内容)：

```java
if (msg instanceof HttpRequest) {
    HttpRequest request = this.request = (HttpRequest) msg;

    if (HttpUtil.is100ContinueExpected(request)) {
        writeResponse(ctx);
    }
    responseData.setLength(0);            
    responseData.append(RequestUtils.formatParams(request));
}
responseData.append(RequestUtils.evaluateDecoderResult(request));

if (msg instanceof HttpContent) {
    HttpContent httpContent = (HttpContent) msg;
    responseData.append(RequestUtils.formatBody(httpContent));
    responseData.append(RequestUtils.evaluateDecoderResult(request));

    if (msg instanceof LastHttpContent) {
        LastHttpContent trailer = (LastHttpContent) msg;
        responseData.append(RequestUtils.prepareLastResponse(request, trailer));
        writeResponse(ctx, trailer, responseData);
    }
}
```

我们可以看到，当我们的通道收到HttpRequest时，它首先检查请求是否需要[100 Continue](https://www.w3.org/Protocols/HTTP/1.1/draft-ietf-http-v11-spec-01.html#Status-Codes)状态。在这种情况下，我们立即回复一个状态为CONTINUE的空响应：

```java
private void writeResponse(ChannelHandlerContext ctx) {
    FullHttpResponse response = new DefaultFullHttpResponse(HTTP_1_1, CONTINUE, 
        Unpooled.EMPTY_BUFFER);
    ctx.write(response);
}
```

之后，处理程序初始化一个要作为响应发送的字符串，并将请求的查询参数添加到其中，以便按原样发送回。

现在让我们定义方法formatParams并将其放在RequestUtils帮助类中来执行此操作：

```java
StringBuilder formatParams(HttpRequest request) {
    StringBuilder responseData = new StringBuilder();
    QueryStringDecoder queryStringDecoder = new QueryStringDecoder(request.uri());
    Map<String, List<String>> params = queryStringDecoder.parameters();
    if (!params.isEmpty()) {
        for (Entry<String, List<String>> p : params.entrySet()) {
            String key = p.getKey();
            List<String> vals = p.getValue();
            for (String val : vals) {
                responseData.append("Parameter: ").append(key.toUpperCase()).append(" = ")
                    .append(val.toUpperCase()).append("rn");
            }
        }
        responseData.append("rn");
    }
    return responseData;
}
```

接下来，在收到HttpContent时，**我们获取请求主体并将其转换为大写**：

```java
StringBuilder formatBody(HttpContent httpContent) {
    StringBuilder responseData = new StringBuilder();
    ByteBuf content = httpContent.content();
    if (content.isReadable()) {
        responseData.append(content.toString(CharsetUtil.UTF_8).toUpperCase())
            .append("\r\n");
    }
    return responseData;
}
```

此外，如果收到的HttpContent是LastHttpContent，我们会添加一条告别消息和尾随标头(如果有)：

```java
StringBuilder prepareLastResponse(HttpRequest request, LastHttpContent trailer) {
    StringBuilder responseData = new StringBuilder();
    responseData.append("Good Bye!\r\n");

    if (!trailer.trailingHeaders().isEmpty()) {
        responseData.append("\r\n");
        for (CharSequence name : trailer.trailingHeaders().names()) {
            for (CharSequence value : trailer.trailingHeaders().getAll(name)) {
                responseData.append("P.S. Trailing Header: ");
                responseData.append(name).append(" = ").append(value).append("\r\n");
            }
        }
        responseData.append("\r\n");
    }
    return responseData;
}
```

### 3.3 编写响应

现在我们要发送的数据已经准备好了，我们可以将响应写入ChannelHandlerContext：

```java
private void writeResponse(ChannelHandlerContext ctx, LastHttpContent trailer, StringBuilder responseData) {
    boolean keepAlive = HttpUtil.isKeepAlive(request);
    FullHttpResponse httpResponse = new DefaultFullHttpResponse(HTTP_1_1, 
        ((HttpObject) trailer).decoderResult().isSuccess() ? OK : BAD_REQUEST,
        Unpooled.copiedBuffer(responseData.toString(), CharsetUtil.UTF_8));
    
    httpResponse.headers().set(HttpHeaderNames.CONTENT_TYPE, "text/plain; charset=UTF-8");

    if (keepAlive) {
        httpResponse.headers().setInt(HttpHeaderNames.CONTENT_LENGTH, httpResponse.content().readableBytes());
        httpResponse.headers().set(HttpHeaderNames.CONNECTION, HttpHeaderValues.KEEP_ALIVE);
    }
    ctx.write(httpResponse);

    if (!keepAlive) {
        ctx.writeAndFlush(Unpooled.EMPTY_BUFFER).addListener(ChannelFutureListener.CLOSE);
    }
}
```

在这个方法中，我们创建了一个HTTP/1.1版本的FullHttpResponse，并添加了我们之前准备好的数据。

如果要保持请求的活动状态，或者换句话说，如果连接不关闭，我们将响应的connection标头设置为keep-alive。否则，我们将关闭连接。

## 4. 测试服务器

为了测试我们的服务器，让我们发送一些[cURL](https://www.baeldung.com/curl-rest)命令并查看响应。

当然，**在此之前我们需要通过运行类HttpServer来启动服务器**。

### 4.1 GET请求

让我们首先调用服务器，并在请求中提供一个cookie：

```bash
curl http://127.0.0.1:8080?param1=one
```

作为回应，我们得到：

```text
Parameter: PARAM1 = ONE

Good Bye!
```

我们也可以从任何浏览器访问http://127.0.0.1:8080?param1=one来查看相同的结果。

### 4.2 POST请求

作为我们的第二个测试，让我们发送一个带有正文示例内容的POST：

```bash
curl -d "sample content" -X POST http://127.0.0.1:8080
```

以下是回复：

```text
SAMPLE CONTENT
Good Bye!
```

这次，由于我们的请求包含正文，服务器以大写形式返回。

## 5. 总结

在本教程中，我们了解了如何实现HTTP协议，特别是使用Netty的HTTP服务器。

[Netty中的HTTP/2](https://www.baeldung.com/netty-http2)演示了HTTP/2协议的客户端-服务器实现。