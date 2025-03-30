---
layout: post
title:  Jetty ReactiveStreams HTTP客户端
category: libraries
copyright: libraries
excerpt: Jetty ReactiveStreams
---

## 1. 概述

在本教程中，我们将学习如何使用[Jetty中的Reactive HTTP客户端](https://github.com/jetty-project/jetty-reactive-httpclient)。我们将通过创建小型测试用例来演示其与不同Reactive库的用法。

## 2. 什么是Reactive HttpClient？

Jetty的[HttpClient](https://github.com/jetty-project/jetty-reactive-httpclient)允许我们执行阻塞HTTP请求。但是，当我们处理Reactive API时，我们不能使用标准HTTP客户端。为了填补这一空白，Jetty为HttpClient API创建了一个包装器，以便它也支持ReactiveStreams API。

**Reactive HttpClient用于通过HTTP调用消费或生成数据流**。

我们在此演示的示例将有一个响应式HTTP客户端，它将使用不同的Reactive库与Jetty服务器进行通信。我们还将讨论Reactive HttpClient提供的请求和响应事件。

建议阅读有关[Project Reactor](https://www.baeldung.com/reactor-core)、[RxJava](https://www.baeldung.com/rx-java)和[Spring WebFlux](https://www.baeldung.com/spring-webflux)的文章，以更好地理解响应式编程概念及其术语。

## 3. Maven依赖

让我们从在pom.xml中添加[Reactive Streams](https://mvnrepository.com/artifact/org.reactivestreams/reactive-streams)、[Project Reactor](https://mvnrepository.com/artifact/org.springframework/spring-webflux)、[RxJava](https://mvnrepository.com/artifact/io.reactivex.rxjava2/rxjava)、[Spring WebFlux](https://mvnrepository.com/artifact/io.projectreactor/reactor-core)和[Jetty Reactive HTTPClient](https://mvnrepository.com/artifact/org.eclipse.jetty/jetty-reactive-httpclient)的依赖开始示例。除此之外，我们还将添加[Jetty Server](https://mvnrepository.com/artifact/org.eclipse.jetty/jetty-server)的依赖以创建服务器：

```xml
<dependency>
    <groupId>org.eclipse.jetty</groupId>
    <artifactId>jetty-reactive-httpclient</artifactId>
    <version>1.0.3</version>
</dependency>
<dependency>
    <groupId>org.eclipse.jetty</groupId>
    <artifactId>jetty-server</artifactId>
    <version>9.4.19.v20190610</version>
</dependency>
<dependency>
    <groupId>org.reactivestreams</groupId>
    <artifactId>reactive-streams</artifactId>
    <version>1.0.3</version>
</dependency>
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-core</artifactId>
    <version>3.6.0</version>
</dependency>
<dependency>
    <groupId>io.reactivex.rxjava2</groupId>
    <artifactId>rxjava</artifactId>
    <version>2.2.11</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webflux</artifactId>
    <version>5.1.9.RELEASE</version>
</dependency>
```

## 4. 创建服务器和客户端

现在让我们创建一个服务器并添加一个请求处理程序，只需将请求正文写入响应：

```java
public class RequestHandler extends AbstractHandler {
    @Override
    public void handle(String target, Request jettyRequest, HttpServletRequest request, HttpServletResponse response) throws IOException, ServletException {
        jettyRequest.setHandled(true);
        response.setContentType(request.getContentType());
        IO.copy(request.getInputStream(), response.getOutputStream());
    }
}

...

Server server = new Server(8080);
server.setHandler(new RequestHandler());
server.start();
```

然后我们可以编写HttpClient：

```java
HttpClient httpClient = new HttpClient();
httpClient.start();
```

现在我们已经创建了客户端和服务器，让我们看看如何将这个阻塞HTTP客户端转换为非阻塞客户端并创建请求：

```java
Request request = httpClient.newRequest("http://localhost:8080/"); 
ReactiveRequest reactiveRequest = ReactiveRequest.newBuilder(request).build();
Publisher<ReactiveResponse> publisher = reactiveRequest.response();
```

因此，Jetty提供的ReactiveRequest包装器使我们的阻塞HTTP客户端具有响应性，让我们继续看看它与不同的响应式库的用法。

## 5. ReactiveStreams用法

Jetty的HttpClient原生支持[Reactive Streams](http://www.reactive-streams.org/announce-1.0.3)，所以让我们从那里开始。

现在，**Reactive Streams只是一组接口**，因此，为了进行测试，让我们实现一个简单的阻塞订阅者：

```java
public class BlockingSubscriber implements Subscriber<ReactiveResponse> {
    BlockingQueue<ReactiveResponse> sink = new LinkedBlockingQueue<>(1);

    @Override
    public void onSubscribe(Subscription subscription) {
        subscription.request(1);
    }

    @Override
    public void onNext(ReactiveResponse response) {
        sink.offer(response);
    }

    @Override
    public void onError(Throwable failure) { }

    @Override
    public void onComplete() { }

    public ReactiveResponse block() throws InterruptedException {
        return sink.poll(5, TimeUnit.SECONDS);
    }
}
```

请注意，我们需要根据JavaDoc调用Subscription#request，其中指出“在通过此方法发出需求信号之前，[Publisher](http://www.reactive-streams.org/reactive-streams-1.0.0-javadoc/org/reactivestreams/Publisher.html)不会发送任何事件”。

另外，请注意，我们添加了一种安全机制，以便我们的测试在5秒内没有看到该值时可以退出。

现在，我们可以快速测试我们的HTTP请求：

```java
BlockingSubscriber subscriber = new BlockingSubscriber();
publisher.subscribe(subscriber);
ReactiveResponse response = subscriber.block();
Assert.assertNotNull(response);
Assert.assertEquals(response.getStatus(), HttpStatus.OK_200);
```

## 6. Project Reactor使用

现在让我们看看如何将Reactive HttpClient与Project Reactor结合使用，发布者的创建与上一节基本相同。

创建发布者后，让我们使用Project Reactor中的Mono类来获取响应式响应：

```java
ReactiveResponse response = Mono.from(publisher).block();
```

然后，我们可以测试结果的响应：

```java
Assert.assertNotNull(response);
Assert.assertEquals(response.getStatus(), HttpStatus.OK_200);
```

### 6.1 Spring WebFlux使用

与Spring WebFlux一起使用时，将阻塞HTTP客户端转换为响应式客户端非常容易。**Spring WebFlux附带了一个响应式客户端WebClient，可与各种HTTP客户端库一起使用**。我们可以使用它作为直接使用Project Reactor代码的替代方案。

因此首先，让我们使用JettyClientHttpConnector包装Jetty的HTTP客户端，并将其与WebClient绑定：

```java
ClientHttpConnector clientConnector = new JettyClientHttpConnector(httpClient);
```

然后将此连接器传递给WebClient以执行非阻塞HTTP请求：

```java
WebClient client = WebClient.builder().clientConnector(clientConnector).build();
```

接下来，让我们使用刚刚创建的Reactive HTTPClient进行实际的HTTP调用并测试结果：

```java
String responseContent = client.post()
    .uri("http://localhost:8080/").contentType(MediaType.TEXT_PLAIN)
    .body(BodyInserters.fromPublisher(Mono.just("Hello World!"), String.class))
    .retrieve()
    .bodyToMono(String.class)
    .block();
Assert.assertNotNull(responseContent);
Assert.assertEquals("Hello World!", responseContent);
```

## 7. RxJava2使用

现在让我们继续看看如何将Reactive HTTP客户端与RxJava2一起使用。

在这里，让我们稍微改变一下我们的例子，现在在请求中包含一个主体：

```java
ReactiveRequest reactiveRequest = ReactiveRequest.newBuilder(request)
    .content(ReactiveRequest.Content
        .fromString("Hello World!", "text/plain", StandardCharsets.UTF_8))
    .build();
Publisher<String> publisher = reactiveRequest
    .response(ReactiveResponse.Content.asString());
```

代码ReactiveResponse.Content.asString()将响应主体转换为字符串，如果我们只对请求的状态感兴趣，也可以使用ReactiveResponse.Content.discard()方法丢弃响应。

现在，我们可以看到使用RxJava2获取响应实际上与Project Reactor非常相似。基本上，我们只是使用Single而不是Mono：

```java
String responseContent = Single.fromPublisher(publisher)
    .blockingGet();

Assert.assertEquals("Hello World!", responseContent);
```

## 8. 请求和响应事件

Reactive HTTP客户端在执行过程中会发出许多事件，它们分为请求事件和响应事件，这些事件有助于了解Reactive HTTP客户端的生命周期。

这次，让我们使用HTTP客户端而不是请求来稍微不同地发出响应请求：

```java
ReactiveRequest request = ReactiveRequest.newBuilder(httpClient, "http://localhost:8080/")
    .content(ReactiveRequest.Content.fromString("Hello World!", "text/plain", UTF_8))
    .build();
```

现在让我们获取HTTP请求事件的发布者：

```java
Publisher<ReactiveRequest.Event> requestEvents = request.requestEvents();
```

现在，让我们再次使用RxJava。这次，我们将创建一个包含事件类型的列表，并通过订阅发生的请求事件来填充它：

```java
List<Type> requestEventTypes = new ArrayList<>();

Flowable.fromPublisher(requestEvents)
    .map(ReactiveRequest.Event::getType).subscribe(requestEventTypes::add);
Single<ReactiveResponse> response = Single.fromPublisher(request.response());
```

然后，由于我们正在测试，我们可以阻塞我们的响应并验证：

```java
int actualStatus = response.blockingGet().getStatus();

Assert.assertEquals(6, requestEventTypes.size());
Assert.assertEquals(HttpStatus.OK_200, actualStatus);
```

类似地，我们也可以订阅响应事件。由于它们与请求事件订阅类似，因此我们在这里仅添加了后者。可以在本文末尾链接的GitHub仓库库中找到包含请求和响应事件的完整实现。

## 9. 总结

在本教程中，我们了解了Jetty提供的ReactiveStreams HttpClient、它与各种Reactive库的用法以及与Reactive请求相关的生命周期事件。