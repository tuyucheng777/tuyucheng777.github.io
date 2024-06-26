---
layout: post
title:  记录响应序列
category: springreactive
copyright: springreactive
excerpt: Spring Reactive
---

## 1. 概述

随着[Spring WebFlux](https://www.baeldung.com/spring-webflux)的引入，我们获得了另一个强大的工具来编写响应式、非阻塞的应用程序。虽然现在使用这项技术比以前容易得多，但**在Spring WebFlux中调试响应序列可能会非常麻烦**。

在这个快速教程中，我们将了解如何轻松地记录异步序列中的事件以及如何避免一些简单的错误。

## 2. Maven依赖

让我们将Spring WebFlux依赖项添加到我们的项目中，以便我们可以创建响应流：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

我们可以从Maven Central获取最新的[spring-boot-starter-webflux](https://central.sonatype.com/artifact/org.springframework.boot/spring-boot-starter-webflux/3.0.3)依赖。

## 3. 创建响应流

首先，让我们使用Flux创建一个响应式流并使用log()方法启用日志记录：

```java
Flux<Integer> reactiveStream = Flux.range(1, 5).log();
```

接下来，我们将订阅它以消费生成的值：

```java
reactiveStream.subscribe();
```

## 4. 记录响应流

运行上面的应用程序后，我们看到我们的记录器在运行：

```shell
16:13:29.735 [main] INFO reactor.Flux.Range.1 - | onSubscribe([Synchronous Fuseable] FluxRange.RangeSubscription)
16:13:29.738 [main] INFO reactor.Flux.Range.1 - | request(unbounded)
16:13:29.739 [main] INFO reactor.Flux.Range.1 - | onNext(1)
16:13:29.739 [main] INFO reactor.Flux.Range.1 - | onNext(2)
16:13:29.739 [main] INFO reactor.Flux.Range.1 - | onNext(3)
16:13:29.739 [main] INFO reactor.Flux.Range.1 - | onNext(4)
16:13:29.739 [main] INFO reactor.Flux.Range.1 - | onNext(5)
16:13:29.739 [main] INFO reactor.Flux.Range.1 - | onComplete()
```

我们可以看到流中发生的每个事件。发出了五个值，然后使用onComplete()事件关闭流。

## 5. 高级日志记录场景

我们可以修改我们的应用程序以查看更有趣的场景。让我们将take()添加到Flux中，这将指示流仅提供特定数量的事件：

```java
Flux<Integer> reactiveStream = Flux.range(1, 5).log().take(3);
```

执行代码后，我们将看到以下输出：

```shell
16:13:29.741 [main] INFO reactor.Flux.Range.2 - | onSubscribe([Synchronous Fuseable] FluxRange.RangeSubscription)
16:13:29.741 [main] INFO reactor.Flux.Range.2 - | request(unbounded)
16:13:29.741 [main] INFO reactor.Flux.Range.2 - | onNext(1)
16:13:29.741 [main] INFO reactor.Flux.Range.2 - | onNext(2)
16:13:29.741 [main] INFO reactor.Flux.Range.2 - | onNext(3)
16:13:29.741 [main] INFO reactor.Flux.Range.2 - | cancel()
```

如我们所见，take()导致流在发出三个事件后取消。

**log()在流中的位置至关重要**。让我们看看将log()放在take()之后如何产生不同的输出：

```java
Flux<Integer> reactiveStream = Flux.range(1, 5).take(3).log();
```

输出：

```shell
16:13:29.742 [main] INFO reactor.Flux.TakeFuseable.3 - | onSubscribe([Fuseable] FluxTake.TakeFuseableSubscriber)
16:13:29.742 [main] INFO reactor.Flux.TakeFuseable.3 - | request(unbounded)
16:13:29.742 [main] INFO reactor.Flux.TakeFuseable.3 - | onNext(1)
16:13:29.742 [main] INFO reactor.Flux.TakeFuseable.3 - | onNext(2)
16:13:29.742 [main] INFO reactor.Flux.TakeFuseable.3 - | onNext(3)
16:13:29.742 [main] INFO reactor.Flux.TakeFuseable.3 - | onComplete()
```

正如我们所看到的，改变观察点会改变输出。现在流产生了三个事件，但我们看到的不是cancel()，而是onComplete()。**这是因为我们观察到使用take()的输出，而不是此方法请求的输出**。

## 6. 总结

在这篇快速文章中，我们了解了如何使用内置的log()方法记录响应流。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-reactive-modules/spring-5-reactive-3)上获得。