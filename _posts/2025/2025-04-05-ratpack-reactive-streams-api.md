---
layout: post
title:  Ratpack中的Reactive Streams API
category: reactor
copyright: reactor
excerpt: Ratpack
---

## 1. 简介

Ratpack是一个基于[Netty](https://www.baeldung.com/netty)引擎构建的框架，它允许我们快速构建HTTP应用程序。我们在[之前](https://www.baeldung.com/?s=ratpack)的文章中已经介绍过它的基本用法，**这次，我们将展示如何使用其流式API来实现响应式应用程序**。

## 2. 快速回顾响应流

在进入实际实现之前，让我们首先快速回顾一下什么是响应式应用程序。根据[原作者](https://github.com/reactivemanifesto/reactivemanifesto)的说法，此类应用程序必须具有以下属性：

- 响应式
- 可扩展
- 有弹性
- 消息驱动

那么，Reactive Streams如何帮助我们实现这些属性呢？好吧，在这种情况下，消息驱动并不一定意味着使用消息中间件。相反，**解决这个问题实际上需要的是异步请求处理和对非阻塞背压的支持**。

Ratpack响应式支持使用JVM的Reactive Streams API标准作为其实现的基础。**因此，它允许与其他兼容框架(如Project Reactor和RxJava)进行互操作**。

## 3. 使用Ratpack的Streams类

**Ratpack的[Streams](https://ratpack.io/manual/current/streams.html#streams)类提供了几种实用方法来创建Publisher实例，然后我们可以使用这些实例来创建数据处理管道**。

一个很好的起点是publish()方法，我们可以使用它从任何Iterable创建一个Publisher：

```java
Publisher<String> pub = Streams.publish(Arrays.asList("hello", "hello again"));
LoggingSubscriber<String> sub = new LoggingSubscriber<String>();
pub.subscribe(sub);
sub.block();
```

这里，LoggingSubscriber是Subscriber接口的一个测试实现，它只记录发布者发出的每个对象。它还包括一个辅助方法block()，顾名思义，该方法会阻塞调用者，直到发布者发出所有对象或产生错误。

运行测试用例，我们将看到预期的事件序列：

```text
onSubscribe: sub=7311908
onNext: sub=7311908, value=hello
onNext: sub=7311908, value=hello again
onComplete: sub=7311908
```

另一个有用的方法是yield()，它有一个Function参数，该参数接收YieldRequest对象并返回要发出的下一个对象：

```java
@Test
public void whenYield_thenSuccess() {

    Publisher<String> pub = Streams.yield((t) -> {
        return t.getRequestNum() < 5 ? "hello" : null;
    });

    LoggingSubscriber<String> sub = new LoggingSubscriber<String>();
    pub.subscribe(sub);
    sub.block();
    assertEquals(5, sub.getReceived());
}
```

YieldRequest参数允许我们使用其getRequestNum()方法根据迄今为止发出的对象数量实现逻辑。在我们的示例中，我们使用此信息来定义结束条件，并通过返回空值来表示结束条件。

现在，让我们看看如何为定期事件创建Publisher：

```java
@Test
public void whenPeriodic_thenSuccess() {
    ScheduledExecutorService executor = Executors.newScheduledThreadPool(1);
    Publisher<String> pub = Streams.periodically(executor, Duration.ofSeconds(1), (t) -> {
        return t < 5 ? String.format("hello %d",t): null; 
    });

    LoggingSubscriber<String> sub = new LoggingSubscriber<String>();
    pub.subscribe(sub);
    sub.block();
    assertEquals(5, sub.getReceived());
}
```

返回的发布者使用ScheduledExecutorService定期调用生产者函数，直到返回空值。生产者函数接收一个整数值，该整数值对应于已发出的对象数，我们使用该整数值来终止流。

## 4. 使用TransformablePublisher

仔细观察Streams的方法，我们可以看到它们通常返回一个[TransformablePublisher](https://ratpack.io/manual/current/api/ratpack/exec/stream/TransformablePublisher.html)。此接口使用几种实用方法扩展了Publisher，这些方法与我们在Project Reactor的Flux和Mono中找到的方法非常相似，**可以更轻松地从各个步骤创建复杂的处理管道**。

作为示例，让我们使用map方法将整数序列转换为字符串：

```java
@Test
public void whenMap_thenSuccess() throws Exception {
    TransformablePublisher<String> pub = Streams.yield( t -> {
                return t.getRequestNum() < 5 ? t.getRequestNum() : null;
            })
            .map(v -> String.format("item %d", v));

    ExecResult<List<String>> result = ExecHarness.yieldSingle((c) -> pub.toList());
    assertTrue("should succeed", result.isSuccess());
    assertEquals("should have 5 items",5,result.getValue().size());
}
```

在这里，实际执行发生在由测试实用程序类[ExecHarness](https://ratpack.io/manual/current/api/ratpack/test/exec/ExecHarness.html)管理的线程池内。由于YieldSingle()需要一个[Promise](https://www.baeldung.com/ratpack-http-client#3-ratpack-promises)，我们使用toList()来适配我们的发布者。此方法收集订阅者生成的所有结果并将它们存储在List中。

正如文档中所述，使用此方法时必须小心谨慎，将其应用于无限制发布者可能会很快导致JVM内存耗尽。**为了避免这种情况，我们应将其使用主要限制在单元测试中**。

除了map()之外，TransformablePublisher还有几个有用的运算符：

- filter()：根据谓词过滤上游对象
- take()：仅从上游发布者发出前n个对象
- wiretap()：添加一个观察点，我们可以在其中检查流经管道的数据和事件
- reduce()：将上游对象缩减为单个值
- transform()：在流中注入常规发布者

## 5. 与不合规的发布者一起使用buffer()

**在某些情况下，我们必须处理Publisher向其订阅者发送比请求更多的元素的情况**。为了解决这些情况，Ratpack的Streams提供了一种buffer()方法，该方法将这些额外的元素保存在内存中，直到订阅者消费它们。

为了说明其工作原理，让我们创建一个简单的不合规Publisher，忽略请求的元素数量。相反，它将始终生成比请求多至少5个元素：

```java
private class NonCompliantPublisher implements Publisher<Integer> {

    @Override
    public void subscribe(Subscriber<? super Integer> subscriber) {
        log.info("subscribe");
        subscriber.onSubscribe(new NonCompliantSubscription(subscriber));
    }

    private class NonCompliantSubscription implements Subscription {
        private Subscriber<? super Integer> subscriber;
        private int recurseLevel = 0;

        public NonCompliantSubscription(Subscriber<? super Integer> subscriber) {
            this.subscriber = subscriber;
        }

        @Override
        public void request(long n) {
            log.info("request: n={}", n);
            if ( recurseLevel > 0 ) {
                return;
            }
            recurseLevel++;
            for (int i = 0 ; i < (n + 5) ; i ++ ) {
                subscriber.onNext(i);
            }
            subscriber.onComplete();
        }

        @Override
        public void cancel() {
        }
    }
}
```

首先，让我们使用LoggingSubscriber测试这个发布者。我们将使用take()运算符，以便它只接收第一个元素。

```java
@Test
public void whenNonCompliantPublisherWithoutBuffer_thenSuccess() throws Exception {
    TransformablePublisher<Integer> pub = Streams.transformable(new NonCompliantPublisher())
            .wiretap(new LoggingAction(""))
            .take(1);

    LoggingSubscriber<Integer> sub = new LoggingSubscriber<>();
    pub.subscribe(sub);
    sub.block();
}
```

**运行此测试，我们发现尽管收到了cancel()请求，但我们不合规的发布者仍在继续生成新元素**：

```text
RatpackStreamsUnitTest - : event=StreamEvent[DataEvent{subscriptionId=0, data=0}]
LoggingSubscriber - onNext: sub=583189145, value=0
RatpackStreamsUnitTest - : event=StreamEvent[RequestEvent{requestAmount=1, subscriptionId=0}]
NonCompliantPublisher - request: n=1
RatpackStreamsUnitTest - : event=StreamEvent[CancelEvent{subscriptionId=0}]
LoggingSubscriber - onComplete: sub=583189145
RatpackStreamsUnitTest - : event=StreamEvent[DataEvent{subscriptionId=0, data=1}]
... more expurious data event
RatpackStreamsUnitTest - : event=StreamEvent[CompletionEvent{subscriptionId=0}]
LoggingSubscriber - onComplete: sub=583189145
```

现在，让我们在这个流中添加一个buffer()步骤，我们将添加两个wiretap步骤来记录它之前的事件，以便其效果更加明显：

```java
@Test
public void whenNonCompliantPublisherWithBuffer_thenSuccess() throws Exception {
    TransformablePublisher<Integer> pub = Streams.transformable(new NonCompliantPublisher())
            .wiretap(new LoggingAction("before buffer"))
            .buffer()
            .wiretap(new LoggingAction("after buffer"))
            .take(1);

    LoggingSubscriber<Integer> sub = new LoggingSubscriber<>();
    pub.subscribe(sub);
    sub.block();
}
```

这次，运行此代码会产生不同的日志序列：

```text
LoggingSubscriber - onSubscribe: sub=675852144
RatpackStreamsUnitTest - after buffer: event=StreamEvent[RequestEvent{requestAmount=1, subscriptionId=0}]
NonCompliantPublisher - subscribe
RatpackStreamsUnitTest - before buffer: event=StreamEvent[RequestEvent{requestAmount=1, subscriptionId=0}]
NonCompliantPublisher - request: n=1
RatpackStreamsUnitTest - before buffer: event=StreamEvent[DataEvent{subscriptionId=0, data=0}]
... more data events
RatpackStreamsUnitTest - before buffer: event=StreamEvent[CompletionEvent{subscriptionId=0}]
RatpackStreamsUnitTest - after buffer: event=StreamEvent[DataEvent{subscriptionId=0, data=0}]
LoggingSubscriber - onNext: sub=675852144, value=0
RatpackStreamsUnitTest - after buffer: event=StreamEvent[RequestEvent{requestAmount=1, subscriptionId=0}]
RatpackStreamsUnitTest - after buffer: event=StreamEvent[CancelEvent{subscriptionId=0}]
RatpackStreamsUnitTest - before buffer: event=StreamEvent[CancelEvent{subscriptionId=0}]
LoggingSubscriber - onComplete: sub=67585214
```

“before buffer”消息表明，我们的不合规发布者能够在第一次调用request后发送所有值。**尽管如此，下游值仍会逐个发送**，以遵守LoggingSubscriber请求的数量。

## 6. 对慢速订阅者使用batch()

另一种可能降低应用程序吞吐量的情况是下游订阅者请求的数据量较小，我们的LoggingSubscriber就是一个很好的例子：它每次只请求一个元素。

**在实际应用中，这会导致大量上下文切换，从而损害整体性，更好的方法是一次请求更多元素**。batch()方法允许上游发布者使用更高效的请求大小，同时允许下游订阅者使用较小的请求大小。

让我们看看这在实践中是如何运作的。和以前一样，我们将从不使用batch的流开始：

```java
@Test
public void whenCompliantPublisherWithoutBatch_thenSuccess() throws Exception {
    TransformablePublisher<Integer> pub = Streams.transformable(new CompliantPublisher(10))
            .wiretap(new LoggingAction(""));

    LoggingSubscriber<Integer> sub = new LoggingSubscriber<>();
    pub.subscribe(sub);
    sub.block();
}
```

此处，CompliantPublisher只是一个测试Publisher，它生成的最大整数为传递给构造函数的值，但不包括该值。让我们运行它来查看非批处理行为：

```text
CompliantPublisher - subscribe
LoggingSubscriber - onSubscribe: sub=-779393331
RatpackStreamsUnitTest - : event=StreamEvent[RequestEvent{requestAmount=1, subscriptionId=0}]
CompliantPublisher - request: requested=1, available=10
RatpackStreamsUnitTest - : event=StreamEvent[DataEvent{subscriptionId=0, data=0}]
LoggingSubscriber - onNext: sub=-779393331, value=0
... more data events omitted
CompliantPublisher - request: requested=1, available=1
RatpackStreamsUnitTest - : event=StreamEvent[CompletionEvent{subscriptionId=0}]
LoggingSubscriber - onComplete: sub=-779393331
```

**输出显示生产者逐个发出值**。现在，让我们将步骤batch()添加到管道中，以便上游发布者一次最多生成5个元素：

```java
@Test
public void whenCompliantPublisherWithBatch_thenSuccess() throws Exception {
    TransformablePublisher<Integer> pub = Streams.transformable(new CompliantPublisher(10))
            .wiretap(new LoggingAction("before batch"))
            .batch(5, Action.noop())
            .wiretap(new LoggingAction("after batch"));

    LoggingSubscriber<Integer> sub = new LoggingSubscriber<>();
    pub.subscribe(sub);
    sub.block();
}
```

batch()方法接收两个参数：每次request()调用时请求的元素数，以及处理丢弃元素(即请求但未消费的元素)的Action。如果出现错误或下游订阅者调用cancel()，则可能会出现这种情况。让我们看看生成的执行日志：

```text
LoggingSubscriber - onSubscribe: sub=-1936924690
RatpackStreamsUnitTest - after batch: event=StreamEvent[RequestEvent{requestAmount=1, subscriptionId=0}]
CompliantPublisher - subscribe
RatpackStreamsUnitTest - before batch: event=StreamEvent[RequestEvent{requestAmount=5, subscriptionId=0}]
CompliantPublisher - request: requested=5, available=10
RatpackStreamsUnitTest - before batch: event=StreamEvent[DataEvent{subscriptionId=0, data=0}]
... first batch data events omitted
RatpackStreamsUnitTest - before batch: event=StreamEvent[RequestEvent{requestAmount=5, subscriptionId=0}]
CompliantPublisher - request: requested=5, available=6
RatpackStreamsUnitTest - before batch: event=StreamEvent[DataEvent{subscriptionId=0, data=5}]
... second batch data events omitted
RatpackStreamsUnitTest - before batch: event=StreamEvent[RequestEvent{requestAmount=5, subscriptionId=0}]
CompliantPublisher - request: requested=5, available=1
RatpackStreamsUnitTest - before batch: event=StreamEvent[CompletionEvent{subscriptionId=0}]
RatpackStreamsUnitTest - after batch: event=StreamEvent[DataEvent{subscriptionId=0, data=0}]
LoggingSubscriber - onNext: sub=-1936924690, value=0
RatpackStreamsUnitTest - after batch: event=StreamEvent[RequestEvent{requestAmount=1, subscriptionId=0}]
RatpackStreamsUnitTest - after batch: event=StreamEvent[DataEvent{subscriptionId=0, data=1}]
... downstream data events omitted
LoggingSubscriber - onComplete: sub=-1936924690

```

**我们可以看到，现在发布者每次都会收到5个元素的请求**。请注意，在这个测试场景中，我们在日志订阅者获得第一个元素之前就看到了两个对生产者的请求。原因是，在这个测试场景中，我们有一个单线程执行，因此batch()会继续缓冲元素，直到它收到onComplete()信号。

## 7. 在Web应用程序中使用流

Ratpack支持与其异步Web框架结合使用响应流。

### 7.1 接收数据流

对于传入的数据，通过处理程序的Context提供的Request对象提供了getBodyStream()方法，该方法返回ByteBuf对象的TransformablePublisher。

从这个发布者开始，我们可以构建我们的处理管道：

```java
@Bean
public Action<Chain> uploadFile() {

    return chain -> chain.post("upload", ctx -> {
        TransformablePublisher<? extends ByteBuf> pub = ctx.getRequest().getBodyStream();
        pub.subscribe(new Subscriber<ByteBuf>() {
            private Subscription sub;
            @Override
            public void onSubscribe(Subscription sub) {
                this.sub = sub;
                sub.request(1);
            }

            @Override
            public void onNext(ByteBuf t) {
                try {
                    ... do something useful with received data
                    sub.request(1);
                }
                finally {
                    // DO NOT FORGET to RELEASE !
                    t.release();
                }
            }

            @Override
            public void onError(Throwable t) {
                ctx.getResponse().status(500);
            }

            @Override
            public void onComplete() {
                ctx.getResponse().status(202);
            }
        });
    });
}
```

实现订阅者时需要考虑几个细节。首先，我们必须确保在某个时候调用ByteBuf的release()方法，**不这样做会导致内存泄漏**。其次，任何异步处理都必须仅使用Ratpack的原语。这些原语包括Promise、Blocking和类似的构造。

### 7.2 发送数据流

发送数据流最直接的方法是使用Response.sendStream()，此方法接收ByteBuf发布者参数并将数据发送到客户端，根据需要施加背压以避免溢出：

```java
@Bean
public Action<Chain> download() {
    return chain -> chain.get("download", ctx -> {
        ctx.getResponse().sendStream(new RandomBytesPublisher(1024,512));
    });
}
```

虽然很简单，但使用此方法也有一个缺点：**它不会自行设置任何标头，包括Content-Length，这对客户端来说可能是一个问题**：

```shell
$ curl -v --output data.bin http://localhost:5050/download
... request messages omitted
< HTTP/1.1 200 OK
< transfer-encoding: chunked
... download progress messages omitted
```

**另外，更好的方法是使用句柄的Context render()方法，传递一个ResponseChunks对象**。在这种情况下，响应将使用“[chunked”](https://en.wikipedia.org/wiki/Chunked_transfer_encoding)传输编码方法。创建[ResponseChunks](https://ratpack.io/manual/current/streams.html#chunked_transfer_encoding)实例的最直接方法是通过此类中可用的静态方法之一：

```java
@Bean
public Action<Chain> downloadChunks() {
    return chain -> chain.get("downloadChunks", ctx -> {
        ctx.render(ResponseChunks.bufferChunks("application/octetstream", new RandomBytesPublisher(1024,512)));
    });
}
```

经过此更改，响应现在包含content-type标头：

```shell
$ curl -v --output data.bin http://localhost:5050/downloadChunks
... request messages omitted
< HTTP/1.1 200 OK
< transfer-encoding: chunked
< content-type: application/octetstream
<
... progress messages omitted
```

### 7.3 使用SSE

对服务器端事件(SSE)的支持也使用render()方法，但在本例中，我们使用[ServerSentEvents](https://ratpack.io/manual/current/streams.html#server_sent_events)将来自生产者的元素适配到包含一些元数据以及事件负载的事件对象：

```java
@Bean
public Action<Chain> quotes() {
    ServerSentEvents sse = ServerSentEvents.serverSentEvents(quotesService.newTicker(), (evt) -> {
        evt
                .id(Long.toString(idSeq.incrementAndGet()))
                .event("quote")
                .data( q -> q.toString());
    });

    return chain -> chain.get("quotes", ctx -> ctx.render(sse));
}
```

在这里，QuotesService只是一个示例服务，它创建一个Publisher，定期生成随机报价。第二个参数是一个准备发送事件的函数，这包括添加id、事件类型和有效负载本身。

我们可以使用curl来测试此方法，输出的内容包括一系列随机引号以及事件元数据：

```shell
$ curl -v http://localhost:5050/quotes
... request messages omitted
< HTTP/1.1 200 OK
< content-type: text/event-stream;charset=UTF-8
< transfer-encoding: chunked
... other response headers omitted
id: 10
event: quote
data: Quote [ts=2021-10-11T01:20:52.081Z, symbol=ORCL, value=53.0]

... more quotes
```

### 7.4 广播Websocket数据

**我们可以使用[Websockets.websocketBroadcast()](https://ratpack.io/manual/current/streams.html#websockets)将数据从任何发布者传输到WebSocket连接**：

```java
@Bean
public Action<Chain> quotesWS() {
    Publisher<String> pub = Streams.transformable(quotesService.newTicker())
            .map(Quote::toString);
    return chain -> chain.get("quotes-ws", ctx -> WebSockets.websocketBroadcast(ctx, pub));
}
```

这里我们使用之前见过的QuotesService作为向客户端广播报价的事件源，我们再次使用curl来模拟WebSocket客户端：

```shell
$ curl --include -v \
     --no-buffer \
     --header "Connection: Upgrade" \
     --header "Upgrade: websocket" \
     --header "Sec-WebSocket-Key: SGVsbG8sIHdvcmxkIQ==" \
     --header "Sec-WebSocket-Version: 13" \
     http://localhost:5050/quotes-ws
... request messages omitted
< HTTP/1.1 101 Switching Protocols
HTTP/1.1 101 Switching Protocols
< upgrade: websocket
upgrade: websocket
< connection: upgrade
connection: upgrade
< sec-websocket-accept: qGEgH3En71di5rrssAZTmtRTyFk=
sec-websocket-accept: qGEgH3En71di5rrssAZTmtRTyFk=

<
<Quote [ts=2021-10-11T01:39:42.915Z, symbol=ORCL, value=63.0]
... more quotes omitted
```

## 8. 总结

在本文中，我们探讨了Ratpack对响应式流的支持以及如何在不同场景中应用它。