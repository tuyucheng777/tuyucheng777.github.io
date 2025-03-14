---
layout: post
title:  Executors newCachedThreadPool()与newFixedThreadPool()
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 概述

说到[线程池](https://www.baeldung.com/java-executor-service-tutorial)实现，Java标准库提供了大量选项可供选择。在这些实现中，固定和缓存线程池非常普遍。

在本教程中，我们将了解线程池的底层工作原理，然后比较这些实现及其用例。

## 2. 缓存线程池

让我们看一下当我们调用[Executors.newCachedThreadPool()](https://github.com/openjdk/jdk/blob/6bab0f539fba8fb441697846347597b4a0ade428/src/java.base/share/classes/java/util/concurrent/Executors.java#L217)时Java是如何创建缓存线程池的：

```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS, 
        new SynchronousQueue<Runnable>());
}
```

缓存线程池使用“同步切换”来排队新任务，同步切换的基本思想很简单，但又违反直觉：当且仅当另一个线程同时获取该元素时，才能将该元素排队。换句话说，**SynchronousQueue不能容纳任何任务**。

假设有新任务到来，**如果队列中有空闲线程在等待，则任务生产者将任务交给该线程。否则，由于队列始终是满的，因此执行器将创建一个新线程来处理该任务**。

缓存池从0线程开始，并可能增长到拥有Integer.MAX_VALUE个线程。实际上，缓存线程池的唯一限制是可用的系统资源。

为了更好地管理系统资源，缓存线程池将删除空闲一分钟的线程。

### 2.1 使用场景

缓存线程池配置会将线程缓存一小段时间(因此得名)，以便将其重新用于其他任务。**因此，当我们处理合理数量的短期任务时，它效果最佳**。 

这里的关键是“合理”和“短暂”，为了阐明这一点，让我们评估一个缓存池不太适合的场景。在这里，我们将提交一百万个任务，每个任务需要100微秒才能完成：

```java
Callable<String> task = () -> {
    long oneHundredMicroSeconds = 100_000;
    long startedAt = System.nanoTime();
    while (System.nanoTime() - startedAt <= oneHundredMicroSeconds);

    return "Done";
};

var cachedPool = Executors.newCachedThreadPool();
var tasks = IntStream.rangeClosed(1, 1_000_000).mapToObj(i -> task).collect(toList());
var result = cachedPool.invokeAll(tasks);
```

这将创建大量线程，导致不合理的内存使用，甚至更糟的是，大量CPU上下文切换。这两种异常都会严重损害整体性能。

**因此，当执行时间不可预测时，例如IO密集型任务，我们应该避免使用该线程池**。

## 3. 固定线程池

让我们看看[固定线程池](https://github.com/openjdk/jdk/blob/6bab0f539fba8fb441697846347597b4a0ade428/src/java.base/share/classes/java/util/concurrent/Executors.java#L91)的工作原理：

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS, 
        new LinkedBlockingQueue<Runnable>());
}
```

与缓存线程池相反，此线程池使用无界队列，队列中有固定数量的永不过期线程。**因此，固定线程池不会使用不断增加的线程数，而是尝试使用固定数量的线程来执行传入的任务**。当所有线程都处于繁忙状态时，执行器将排队新任务。这样，我们就可以更好地控制程序的资源消耗。

因此，固定线程池更适合执行时间不可预测的任务。

## 4. 不幸的相似之处

到目前为止，我们仅列举了缓存线程池和固定线程池之间的区别。

除了这些差异之外，它们都使用[AbortPolicy](https://www.baeldung.com/java-rejectedexecutionhandler#1-abort-policy)作为[饱和策略](https://www.baeldung.com/java-rejectedexecutionhandler)。因此，我们预计这些执行器在无法接收甚至排队更多任务时会抛出异常。

让我们看看现实世界中会发生什么。

缓存线程池在极端情况下会继续创建越来越多的线程，因此**实际上它们永远不会达到饱和点**。同样，固定线程池会继续在其队列中添加越来越多的任务。**因此，固定线程池也永远不会达到饱和点**。

**由于这两个池都不会饱和，因此当负载异常高时，它们将消耗大量内存来创建线程或排队任务。更糟糕的是，缓存线程池还会引起大量的处理器上下文切换**。

无论如何，为了更好地控制资源消耗，强烈建议创建自定义[ThreadPoolExecutor](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ThreadPoolExecutor.html)：

```java
var boundedQueue = new ArrayBlockingQueue<Runnable>(1000);
new ThreadPoolExecutor(10, 20, 60, SECONDS, boundedQueue, new AbortPolicy());
```

在这里，我们的线程池最多可以有20个线程，并且最多只能排队1000个任务。此外，当它无法再接收任何负载时，它只会抛出异常。

## 5. 总结

在本教程中，我们深入了解了JDK源代码，了解了不同的Executor的底层工作原理。然后，我们比较了固定和缓存线程池及其用例。

最后，我们尝试使用自定义线程池来解决这些池的资源消耗失控的问题。