---
layout: post
title:  在非阻塞上下文中处理阻塞方法警告
category: spring-reactive
copyright: spring-reactive
excerpt: Spring Reactive
---

## 1. 概述

在本文中，我们将探讨警告：“Possibly blocking call in non-blocking context could lead to thread starvation”。首先，我们将使用一个简单的示例重新生成警告，并探讨如何在与我们的情况不相关的情况下抑制它。

然后，我们将讨论忽视它的风险并探讨两种有效解决该问题的方法。

## 2. 非阻塞上下文中的阻塞方法

**如果我们尝试在[响应式上下文](https://www.baeldung.com/java-reactive-systems)中使用阻塞操作，IntelliJ IDEA将提示“Possibly blocking call in non-blocking context could lead to thread starvation”警告**。

假设我们正在使用[Spring WebFlux](https://www.baeldung.com/spring-webflux/)和[Netty](https://www.baeldung.com/netty)服务器开发一个响应式Web应用程序，如果我们在处理应保持非阻塞的HTTP请求时引入阻塞操作，我们将遇到此警告：

![](/assets/images/2025/springreactive/javahandleblockingmethodinnonblockingcontextwarning01.png)

此警告源自IntelliJ IDEA的静态分析，如果我们确信它不会影响我们的应用程序，我们可以使用“BlockingMethodInNonBlockingContext”检查名称轻松抑制警告：

```java
@SuppressWarnings("BlockingMethodInNonBlockingContext")
@GetMapping("/warning")
Mono<String> warning() {
    // ...
}
```

然而，了解潜在问题并评估其影响至关重要。**在某些情况下，这可能会导致阻塞负责处理HTTP请求的线程，从而造成严重影响**。

## 3. 理解警告

让我们展示一个场景，忽略此警告可能会导致线程饥饿并阻塞传入的HTTP流量。在这个例子中，我们将添加另一个端点，并使用Thread.sleep()故意阻塞线程2秒钟，尽管处于响应式上下文中：

```java
@GetMapping("/blocking")
Mono<String> getBlocking() {
    return Mono.fromCallable(() -> {
        try {
            Thread.sleep(2_000);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        return "foo";
    });
}
```

**在这种情况下，Netty处理传入HTTP请求的事件循环线程可能会很快被阻塞，导致无响应**。例如，如果我们发送200个并发请求，即使不涉及任何计算，应用程序也需要32秒才能响应所有请求。此外，这也会影响其他端点-即使它们不需要阻塞操作。

出现这种延迟的原因是Netty HTTP线程池的大小为12，因此它一次只能处理12个请求。如果我们检查IntelliJ分析器，我们可以预期看到线程大部分时间被阻塞，并且整个测试过程中CPU使用率非常低：

![](/assets/images/2025/springreactive/javahandleblockingmethodinnonblockingcontextwarning02.png)

## 4. 修复问题

**理想情况下，我们应该改用响应式API来解决这个问题**。但是，如果这不可行，我们应该为此类操作使用单独的线程池，以避免阻塞HTTP线程。

### 4.1 使用响应式替代方案

首先，我们应尽可能采取响应式方法，这意味着要找到替代阻塞操作的响应式方法。

**例如，我们可以尝试将响应式数据库驱动程序与[Spring Data Reactive Repository](https://www.baeldung.com/spring-data-mongodb-reactive)或响应式HTTP客户端(如[WebClient](https://www.baeldung.com/spring-5-webclient))一起使用**。在我们的简单情况下，我们可以使用[Mono的API](https://projectreactor.io/docs/core/release/reference/#mono)将响应延迟2秒，而不是依赖于阻塞Thread.sleep()：

```java
@GetMapping("/non-blocking")
Mono<String> getNonBlocking() {
    return Mono.just("bar")
        .delayElement(Duration.ofSeconds(2));
}
```

通过这种方法，应用程序可以处理数百个并发请求，并在我们引入的2秒延迟后发送所有响应。

### 4.2 使用专用调度程序执行阻塞操作

另一方面，有些情况下我们没有选择使用响应式API。一种常见的情况是使用非响应式驱动程序查询数据库，这将导致阻塞操作：

```java
@GetMapping("/blocking-with-scheduler")
Mono<String> getBlockingWithDedicatedScheduler() {
    String data = fetchDataBlocking();
    return Mono.just("retrieved data: " + data);
}
```

**在这些情况下，我们可以将阻塞操作包装在Mono中，并使用subscribeOn()指定其执行的调度程序**。这为我们提供了一个Mono<String\>，稍后可以将其映射到我们所需的响应格式：

```java
@GetMapping("/blocking-with-scheduler")
Mono<String> getBlockingWithDedicatedScheduler() {
    return Mono.fromCallable(this::fetchDataBlocking)
        .subscribeOn(Schedulers.boundedElastic())
        .map(data -> "retrieved data: " + data);
}
```

## 5. 总结

在本教程中，我们介绍了IntelliJ静态分析器生成的“Possibly blocking call in non-blocking context could lead to thread starvation”警告。通过代码示例，我们演示了忽略此警告如何会阻塞Netty的线程池来处理传入的HTTP请求，从而使应用程序无响应。

之后，我们了解到尽可能使用响应式API可以帮助我们解决这个问题。此外，我们还了解到，当我们没有响应式替代方案时，我们应该为阻塞操作使用单独的线程池。