---
layout: post
title:  Java中的嵌入式Jetty服务器
category: libraries
copyright: libraries
excerpt: Jetty
---

## 1. 概述

在本文中，我们将介绍[Jetty](http://www.eclipse.org/jetty/)库。Jetty提供了一个可以作为嵌入式容器运行的Web服务器，并且可以轻松地与javax.servlet库集成。

## 2. Maven依赖

首先，我们将[jetty-server](https://mvnrepository.com/artifact/org.eclipse.jetty/jetty-server)和[jetty-servlet](https://mvnrepository.com/artifact/org.eclipse.jetty/jetty-servlet)依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>org.eclipse.jetty</groupId>
    <artifactId>jetty-server</artifactId>
    <version>9.4.3.v20170317</version>
</dependency>
<dependency>
    <groupId>org.eclipse.jetty</groupId>
    <artifactId>jetty-servlet</artifactId>
    <version>9.4.3.v20170317</version>
</dependency>
```

## 3. 使用Servlet启动Jetty服务器

启动Jetty嵌入式容器很简单，我们需要实例化一个新的Server对象并将其设置为在给定端口上启动：

```java
public class JettyServer {
    private Server server;

    public void start() throws Exception {
        server = new Server();
        ServerConnector connector = new ServerConnector(server);
        connector.setPort(8090);
        server.setConnectors(new Connector[]{connector});
    }
}
```

假设我们要创建一个端点，如果一切顺利，它将以HTTP状态码200和简单的JSON有效负载进行响应。

我们将创建一个扩展HttpServlet类的类来处理这样的请求；此类将是单线程的并且会阻塞直到完成：

```java
public class BlockingServlet extends HttpServlet {

    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response)
            throws ServletException, IOException {

        response.setContentType("application/json");
        response.setStatus(HttpServletResponse.SC_OK);
        response.getWriter().println("{ \"status\": \"ok\"}");
    }
}
```

接下来，我们需要使用addServletWithMapping()方法在ServletHandler对象中注册BlockingServlet类并启动服务器：

```java
servletHandler.addServletWithMapping(BlockingServlet.class, "/status");
server.start();
```

如果我们希望测试我们的Servlet逻辑，我们需要使用之前创建的JettyServer类来启动我们的服务器，该类是测试设置中实际Jetty服务器实例的包装器：

```java
@Before
public void setup() throws Exception {
    jettyServer = new JettyServer();
    jettyServer.start();
}
```

一旦启动，我们将向/status端点发送测试HTTP请求：

```java
String url = "http://localhost:8090/status";
HttpClient client = HttpClientBuilder.create().build();
HttpGet request = new HttpGet(url);

HttpResponse response = client.execute(request);
 
assertThat(response.getStatusLine().getStatusCode()).isEqualTo(200);
```

## 4. 非阻塞Servlet

Jetty对异步请求处理有良好的支持。

假设我们拥有大量I/O密集型资源，需要很长时间才能加载，从而长时间阻塞执行线程。最好能释放该线程来处理其他请求，而不是等待某些I/O资源。

为了使用Jetty提供此类逻辑，我们可以创建一个使用[AsyncContext](http://docs.oracle.com/javaee/6/api/javax/servlet/AsyncContext.html)类的Servlet，方法是调用HttpServletRequest上的startAsync()方法。此代码不会阻塞正在执行的线程，而是在单独的线程中执行I/O操作，并在准备就绪时使用AsyncContext.complete()方法返回结果：

```java
public class AsyncServlet extends HttpServlet {
    private static String HEAVY_RESOURCE = "This is some heavy resource that will be served in an async way";

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        ByteBuffer content = ByteBuffer.wrap(HEAVY_RESOURCE.getBytes(StandardCharsets.UTF_8));

        AsyncContext async = request.startAsync();
        ServletOutputStream out = response.getOutputStream();
        out.setWriteListener(new WriteListener() {
            @Override
            public void onWritePossible() throws IOException {
                while (out.isReady()) {
                    if (!content.hasRemaining()) {
                        response.setStatus(200);
                        async.complete();
                        return;
                    }
                    out.write(content.get());
                }
            }

            @Override
            public void onError(Throwable t) {
                getServletContext().log("Async Error", t);
                async.complete();
            }
        });
    }
}
```

我们将ByteBuffer写入OutputStream，并且一旦写入整个缓冲区，我们就会通过调用complete()方法来发出信号，表示结果已准备好返回给客户端。

接下来，我们需要添加AsyncServlet作为Jetty Servlet映射：

```java
servletHandler.addServletWithMapping(AsyncServlet.class, "/heavy/async");
```

现在我们可以向/heavy/async端点发送请求-该请求将由Jetty以异步方式处理：

```java
String url = "http://localhost:8090/heavy/async";
HttpClient client = HttpClientBuilder.create().build();
HttpGet request = new HttpGet(url);
HttpResponse response = client.execute(request);

assertThat(response.getStatusLine().getStatusCode())
    .isEqualTo(200);
String responseContent = IOUtils.toString(r
  esponse.getEntity().getContent(), StandardCharsets.UTF_8);
assertThat(responseContent).isEqualTo("This is some heavy resource that will be served in an async way");
```

当我们的应用程序以异步方式处理请求时，我们应该明确配置线程池。在下一节中，我们将配置Jetty以使用自定义线程池。

## 5. Jetty配置

当我们在生产环境中运行Web应用程序时，我们可能需要调整Jetty服务器处理请求的方式，这可以通过定义线程池并将其应用于Jetty服务器来实现。

为此，我们可以设置三个配置：

- maxThreads：指定Jetty可以在池中创建和使用的最大线程数
- minThreads：设置Jetty将使用的池中的初始线程数
- idleTimeout：此值(以毫秒为单位)定义线程在停止并从线程池中移除之前可以空闲多长时间，池中剩余的线程数永远不会低于minThreads设置

通过这些，我们可以通过将配置的线程池传递给服务器构造函数来以编程方式配置嵌入式Jetty服务器：

```java
int maxThreads = 100;
int minThreads = 10;
int idleTimeout = 120;

QueuedThreadPool threadPool = new QueuedThreadPool(maxThreads, minThreads, idleTimeout);

server = new Server(threadPool);
```

然后，当我们启动服务器时，它将使用来自特定线程池的线程。

## 6. 总结

在本快速教程中，我们了解了如何集成嵌入式服务器与Jetty并测试了我们的Web应用程序。