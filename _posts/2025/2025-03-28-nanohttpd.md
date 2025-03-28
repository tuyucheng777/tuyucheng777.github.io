---
layout: post
title:  NanoHTTPD指南
category: libraries
copyright: libraries
excerpt: NanoHTTPD
---

## 1. 简介

[NanoHTTPD](https://github.com/NanoHttpd/nanohttpd)是一个用Java编写的开源、轻量级Web服务器。

在本教程中，我们将创建一些REST API来探索其功能。

## 2. 项目设置

让我们将[NanoHTTPD](https://mvnrepository.com/artifact/org.nanohttpd/nanohttpd)核心依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>org.nanohttpd</groupId>
    <artifactId>nanohttpd</artifactId>
    <version>2.3.1</version>
</dependency>
```

要创建一个简单的服务器，我们需要扩展NanoHTTPD并重写其serve方法：

```java
public class App extends NanoHTTPD {
    public App() throws IOException {
        super(8080);
        start(NanoHTTPD.SOCKET_READ_TIMEOUT, false);
    }

    public static void main(String[] args ) throws IOException {
        new App();
    }

    @Override
    public Response serve(IHTTPSession session) {
        return newFixedLengthResponse("Hello world");
    }
}
```

我们将运行端口定义为8080，并将服务器定义为守护进程(无读取超时)。

一旦我们启动应用程序，URL [http://localhost:8080/](http://localhost:8080/)将返回Hello world消息。我们使用NanoHTTPD#newFixedLengthResponse方法作为构建NanoHTTPD.Response对象的便捷方式。

让我们用[cURL](https://www.baeldung.com/curl-rest)尝试我们的项目：

```shell
> curl 'http://localhost:8080/'
Hello world
```

## 3. REST API

在HTTP方法方面，NanoHTTPD允许GET、POST、PUT、DELETE、HEAD、TRACE和其他几种方法。

简单来说，我们可以通过方法枚举找到支持的HTTP动词，让我们看看这是如何实现的。

### 3.1 HTTP GET

首先，我们来看一下GET。例如，我们只想在应用程序收到GET请求时返回内容。

与[Java Servlet容器](https://www.baeldung.com/intro-to-servlets)不同，我们没有可用的doGet方法-相反，我们只是通过getMethod检查值：

```java
@Override
public Response serve(IHTTPSession session) {
    if (session.getMethod() == Method.GET) {
        String itemIdRequestParameter = session.getParameters().get("itemId").get(0);
        return newFixedLengthResponse("Requested itemId = " + itemIdRequestParameter);
    }
    return newFixedLengthResponse(Response.Status.NOT_FOUND, MIME_PLAINTEXT, "The requested resource does not exist");
}
```

这很简单，对吧？让我们通过curl调用新端点进行快速测试，看看请求参数itemId是否正确读取：

```shell
> curl 'http://localhost:8080/?itemId=23Bk8'
Requested itemId = 23Bk8
```

### 3.2 HTTP POST

我们之前对GET做出了响应并从URL中读取了一个参数。

为了涵盖两种最流行的HTTP方法，我们是时候处理POST(从而读取请求正文)了：

```java
@Override
public Response serve(IHTTPSession session) {
    if (session.getMethod() == Method.POST) {
        try {
            session.parseBody(new HashMap<>());
            String requestBody = session.getQueryParameterString();
            return newFixedLengthResponse("Request body = " + requestBody);
        } catch (IOException | ResponseException e) {
            // handle
        }
    }
    return newFixedLengthResponse(Response.Status.NOT_FOUND, MIME_PLAINTEXT, "The requested resource does not exist");
}
```

注意，之前我们请求请求体时，**首先调用了parseBody方法**，这是因为我们想要加载请求体以供稍后检索。

我们将在cURL命令中包含一个正文：

```shell
> curl -X POST -d 'deliveryAddress=Washington nr 4&quantity=5''http://localhost:8080/'
Request body = deliveryAddress=Washington nr 4&quantity=5
```

其余的HTTP方法本质上非常相似，因此我们将跳过它们。

## 4. 跨源资源共享

使用[CORS](https://www.baeldung.com/spring-cors)，我们可以实现跨域通信，最常见的用例是来自不同域的AJAX调用。

我们可以使用的第一种方法是为所有API启用CORS，使用--cors参数，我们将允许访问所有域。我们还可以使用–cors="http://dashboard.myApp.com http://admin.myapp.com"定义允许哪些域。

第二种方法是为单个API启用CORS，让我们看看如何使用addHeader来实现这一点：

```java
@Override 
public Response serve(IHTTPSession session) {
    Response response = newFixedLengthResponse("Hello world"); 
    response.addHeader("Access-Control-Allow-Origin", "*");
    return response;
}
```

现在当我们使用cURL时，我们将得到我们的CORS标头：

```shell
> curl -v 'http://localhost:8080'
HTTP/1.1 200 OK 
Content-Type: text/html
Date: Thu, 13 Jun 2019 03:58:14 GMT
Access-Control-Allow-Origin: *
Connection: keep-alive
Content-Length: 11

Hello world
```

## 5. 文件上传

NanoHTTPD对文件上传[有单独的依赖](https://mvnrepository.com/artifact/org.nanohttpd/nanohttpd-apache-fileupload)，因此让我们将其添加到项目中：

```xml
<dependency>
    <groupId>org.nanohttpd</groupId>
    <artifactId>nanohttpd-apache-fileupload</artifactId>
    <version>2.3.1</version>
</dependency>
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
    <scope>provided</scope>
</dependency>
```

请注意，还需要[servlet-api依赖](https://mvnrepository.com/artifact/javax.servlet/javax.servlet-api)(否则我们会收到编译错误)。

NanoHTTPD公开的是一个名为NanoFileUpload的类：

```java
@Override
public Response serve(IHTTPSession session) {
    try {
        List<FileItem> files = new NanoFileUpload(new DiskFileItemFactory()).parseRequest(session);
        int uploadedCount = 0;
        for (FileItem file : files) {
            try {
                String fileName = file.getName();
                byte[] fileContent = file.get();
                Files.write(Paths.get(fileName), fileContent);
                uploadedCount++;
            } catch (Exception exception) {
                // handle
            }
        }
        return newFixedLengthResponse(Response.Status.OK, MIME_PLAINTEXT,
                "Uploaded files " + uploadedCount + " out of " + files.size());
    } catch (IOException | FileUploadException e) {
        throw new IllegalArgumentException("Could not handle files from API request", e);
    }
    return newFixedLengthResponse(Response.Status.BAD_REQUEST, MIME_PLAINTEXT, "Error when uploading");
}
```

我们来调用一下：

```bash
> curl -F 'filename=@/pathToFile.txt' 'http://localhost:8080'
Uploaded files: 1
```

## 6. 多路由

nanolet类似于Servlet，但配置非常低，我们可以使用它们来定义由单个服务器提供的多个路由(与前面只有一个路由的示例不同)。

首先，让我们添加[nanolets](https://mvnrepository.com/artifact/org.nanohttpd/nanohttpd-nanolets)所需的依赖：

```xml
<dependency>
    <groupId>org.nanohttpd</groupId>
    <artifactId>nanohttpd-nanolets</artifactId>
    <version>2.3.1</version>
</dependency>
```

现在我们将使用RouterNanoHTTPD扩展我们的主类，定义我们的运行端口并让服务器作为守护进程运行。

我们将在addMappings方法来定义处理程序：

```java
public class MultipleRoutesExample extends RouterNanoHTTPD {
    public MultipleRoutesExample() throws IOException {
        super(8080);
        addMappings();
        start(NanoHTTPD.SOCKET_READ_TIMEOUT, false);
    }

    @Override
    public void addMappings() {
        // todo fill in the routes
    }
}
```

下一步是定义我们的addMappings方法，让我们定义一些处理程序。 

第一个是IndexHandler类，用于“/”路径。该类来自NanoHTTPD库，默认返回Hello World消息。当我们希望获得不同的响应时，可以重写getText方法：

```java
addRoute("/", IndexHandler.class); // inside addMappings method
```

为了测试我们的新路由，我们可以这样做：

```shell
> curl 'http://localhost:8080' 
<html><body><h2>Hello world!</h3></body></html>
```

其次，让我们创建一个新的UserHandler类，它扩展了现有的DefaultHandler。它的路由将是/users。在这里，我们处理了文本、MIME类型和返回的状态码：

```java
public static class UserHandler extends DefaultHandler {
    @Override
    public String getText() {
        return "UserA, UserB, UserC";
    }

    @Override
    public String getMimeType() {
        return MIME_PLAINTEXT;
    }

    @Override
    public Response.IStatus getStatus() {
        return Response.Status.OK;
    }
}
```

要调用此路由，我们将再次发出cURL命令：

```shell
> curl -X POST 'http://localhost:8080/users' 
UserA, UserB, UserC
```

最后，我们可以使用新的StoreHandler类来探索GeneralHandler。我们修改了返回的消息以包含URL的storeId部分。

```java
public static class StoreHandler extends GeneralHandler {
    @Override
    public Response get(UriResource uriResource, Map<String, String> urlParams, IHTTPSession session) {
        return newFixedLengthResponse("Retrieving store for id = " + urlParams.get("storeId"));
    }
}
```

让我们检查一下我们的新API：

```shell
> curl 'http://localhost:8080/stores/123' 
Retrieving store for id = 123
```

## 7. HTTPS

为了使用HTTPS，我们需要证书。请参阅我们关于[SSL](https://www.baeldung.com/java-ssl)的文章以获取更多详细信息。

我们可以使用[Let's Encrypt](https://letsencrypt.org/)之类的服务，或者我们可以简单地生成自签名证书，如下所示：

```shell
> keytool -genkey -keyalg RSA -alias selfsigned
  -keystore keystore.jks -storepass password -validity 360
  -keysize 2048 -ext SAN=DNS:localhost,IP:127.0.0.1  -validity 9999
```

接下来，我们将这个keystore.jks复制到我们的类路径上的一个位置，比如说Maven项目的src/main/resources文件夹。

之后，我们可以在调用NanoHTTPD#makeSSLSocketFactory时引用它：

```java
public class HttpsExample  extends NanoHTTPD {

    public HttpsExample() throws IOException {
        super(8080);
        makeSecure(NanoHTTPD.makeSSLSocketFactory("/keystore.jks", "password".toCharArray()), null);
        start(NanoHTTPD.SOCKET_READ_TIMEOUT, false);
    }

    // main and serve methods
}
```

现在我们可以尝试一下，请注意使用--insecure参数，因为cURL默认无法验证我们的自签名证书：

```shell
> curl --insecure 'https://localhost:8443'
HTTPS call is a success
```

## 8. WebSockets

NanoHTTPD支持[WebSockets](https://www.baeldung.com/rest-vs-websockets)。

让我们创建WebSocket的最简单实现。为此，我们需要扩展NanoWSD类。我们还需要为WebSocket添加[NanoHTTPD依赖](https://mvnrepository.com/artifact/org.nanohttpd/nanohttpd-websocket)

```xml
<dependency>
    <groupId>org.nanohttpd</groupId>
    <artifactId>nanohttpd-websocket</artifactId>
    <version>2.3.1</version>
</dependency>
```

对于我们的实现，我们只需使用一个简单的文本有效负载进行回复：

```java
public class WsdExample extends NanoWSD {
    public WsdExample() throws IOException {
        super(8080);
        start(NanoHTTPD.SOCKET_READ_TIMEOUT, false);
    }

    public static void main(String[] args) throws IOException {
        new WsdExample();
    }

    @Override
    protected WebSocket openWebSocket(IHTTPSession ihttpSession) {
        return new WsdSocket(ihttpSession);
    }

    private static class WsdSocket extends WebSocket {
        public WsdSocket(IHTTPSession handshakeRequest) {
            super(handshakeRequest);
        }

        //override onOpen, onClose, onPong and onException methods

        @Override
        protected void onMessage(WebSocketFrame webSocketFrame) {
            try {
                send(webSocketFrame.getTextPayload() + " to you");
            } catch (IOException e) {
                // handle
            }
        }
    }
}
```

这次我们不用cURL，而是使用[wscat](https://github.com/websockets/wscat)：

```shell
> wscat -c localhost:8080
hello
hello to you
bye
bye to you
```

## 9. 总结

总而言之，我们创建了一个使用NanoHTTPD库的项目。接下来，我们定义了RESTful API并探索了更多与HTTP相关的功能。最后，我们还实现了WebSocket。