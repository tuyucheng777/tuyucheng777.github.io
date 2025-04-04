---
layout: post
title:  CompletableFuture与Mono
category: reactor
copyright: reactor
excerpt: Reactor
---

## 1. 概述

在本快速教程中，我们将了解Java中的Mono和Project Reactor中的CompletableFuture之间的区别，我们将重点介绍它们如何处理异步任务以及完成这些任务时执行的操作。

我们先来看一下CompletableFuture。

## 2. 理解CompletableFuture

[CompletableFuture](https://www.baeldung.com/java-completablefuture)是在Java 8中引入的，它基于之前的[Future](https://www.baeldung.com/java-future)类，并提供了一种异步运行代码的方法。简而言之，它改进了异步编程并简化了线程处理。

此外，我们可以使用[thenApply()](https://www.baeldung.com/java-completablefuture#1-thenapply)、[thenAccept()](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/CompletableFuture.html#thenAccept(java.util.function.Consumer))和 [thenCompose()](https://www.baeldung.com/java-completablefuture#1-thenapply)等方法创建计算链来协调我们的异步任务。

虽然CompletableFuture是异步的，这意味着主线程会继续执行其他任务，而不会等待当前操作完成，但它**并不是完全非阻塞的**。长时间运行的操作可能会阻塞执行它的线程：

```java
CompletableFuture completableFuture = CompletableFuture.supplyAsync(() -> {
    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
    return "Finished completableFuture";
});
```

上面，我们使用Thread类中的sleep()方法模拟一个耗时操作。如果未定义，supplyAsync()将使用ForkJoinPool并分配一个线程来运行匿名[Lambda函数](https://www.baeldung.com/java-8-functional-interfaces#Lambdas)，并且该线程被sleep()方法阻塞。

此外，**在CompletableFuture实例完成操作之前调用其中的get()方法会阻塞主线程**：

```java
try {
    completableFuture.get(); // This blocks the main thread
} catch (InterruptedException | ExecutionException e) {
    e.printStackTrace();
}
```

为了避免这种情况发生，我们可以使用[回调模式](https://www.baeldung.com/java-callback-functions)中的completeExceptionally()或complete()方法手动处理CompletableFuture完成。例如，假设我们有一个要在不阻塞主线程的情况下运行的函数，然后，我们希望在它失败和成功完成时处理Future：

```java
public void myAsyncCall(String param, BiConsumer<String, Throwable> callback) {
    new Thread(() -> {
        try {
            Thread.sleep(1000);
            callback.accept("Response from API with param: " + param, null);
        } catch (InterruptedException e) {
            callback.accept(null, e);
        }
    }).start();
}
```

然后，我们使用此函数创建一个CompletableFuture：

```java
public CompletableFuture<String> nonBlockingApiCall(String param) {
    CompletableFuture<String> completableFuture = new CompletableFuture<>();
    myAsyncCall(param, (result, error) -> {
        if (error != null) {
            completableFuture.completeExceptionally(error);
        } else {
            completableFuture.complete(result);
        }
    });
    return completableFuture;
}
```

还有一种替代的、更具响应式的方法来处理相同的操作，我们接下来会看到。

## 3. Mono与CompletableFuture的比较

> [Project Reactor](https://www.baeldung.com/reactor-core)中的Mono类采用了响应式原则，与CompletableFuture不同，**Mono旨在以更少的开销支持并发**。

此外，与CompletableFuture的即时执行相比，Mono是惰性的，这意味着除非我们订阅Mono，否则我们的应用程序不会消耗资源：

```java
Mono<String> reactiveMono = Mono.fromCallable(() -> {
    Thread.sleep(1000); // Simulate some computation
    return "Reactive Data";
}).subscribeOn(Schedulers.boundedElastic());

reactiveMono.subscribe(System.out::println);
```

上面，我们使用fromCallable()方法创建一个Mono对象，并以[提供者](https://www.baeldung.com/java-callable-vs-supplier#supplier)的身份提供阻塞操作。然后，我们使用subscribeOn()方法将操作委托给单独的线程。

**Schedulers.boundedElastic()类似于缓存线程池，但对最大线程数有限制**。这确保主线程保持非阻塞状态，阻塞主线程的唯一方法是强制调用block()方法，此方法等待Mono实例的完成(无论成功与否)。

然后，为了运行响应式管道，我们使用subscribe()来通过[方法引用](https://www.baeldung.com/java-method-references)将Mono对象的结果订阅到println()。

Mono类非常灵活，提供了一组运算符来描述性地转换和组合其他Mono对象。它还支持[背压](https://www.baeldung.com/spring-webflux-backpressure#backpressure-in-reactive-streams)，以防止应用程序耗尽所有资源。

## 4. 总结

在这篇简短的文章中，我们将CompletableFuture与Project Reactor中的Mono类进行了比较。首先，我们描述了CompletableFuture如何运行异步任务。然后，我们展示了如果配置不正确，它可能会阻塞正在处理的线程以及主线程。最后，我们展示了如何使用Mono以响应式方式运行异步操作。