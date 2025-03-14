---
layout: post
title:  ListenableFuture和CompletableFuture中的回调
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 概述

[ListenableFuture](https://www.baeldung.com/guava-futures-listenablefuture)和[CompletableFuture](https://www.baeldung.com/java-completablefuture)构建于Java Future接口之上，前者是Google Guava库的一部分，而后者是Java 8的一部分。

众所周知，**Future接口不提供回调功能，而ListenableFuture和CompletableFuture都填补了这一空白**。在本教程中，我们将学习使用它们的回调机制。

## 2. 异步任务中的回调

让我们定义一个用例，我们需要从远程服务器下载图像文件并将图像文件的名称保留在数据库中。由于该任务由网络上的操作组成并消耗时间，因此它是使用Java异步功能的完美案例。

我们可以创建一个函数，从远程服务器下载文件并附加一个监听器，然后在下载完成时将数据推送到数据库。

让我们看看如何使用ListenableFuture和CompletableFuture。

## 3. ListenableFuture中的回调

首先我们在pom.xml中添加Google的Guava库[依赖](https://mvnrepository.com/artifact/com.google.guava/guava)：

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>32.1.3-jre</version>
</dependency>
```

现在，让我们模拟一个从远程Web服务器下载文件的Future：

```java
ExecutorService executorService = Executors.newFixedThreadPool(1);
ListeningExecutorService pool = MoreExecutors.listeningDecorator(executorService);
ListenableFuture<String> listenableFuture = pool.submit(downloadFile());

private static Callable<String> downloadFile() {
    return () -> {
        // Mimicking the downloading of a file by adding a sleep call
        Thread.sleep(5000);
        return "pic.jpg";
    };
}
```

上面的代码创建了一个ExecutorService包裹在MoreExecutors中创建线程池，在ListenableFutureService的submit方法中，我们传递一个[Callable](https://www.baeldung.com/java-runnable-callable)<String\>下载文件并返回文件名，该文件返回ListenableFuture。

为了在ListenableFuture实例上附加回调，Guava在Future中提供了实用方法：

```java
Futures.addCallback(
    listenableFuture,
    new FutureCallback<>() {
        @Override
        public void onSuccess(String fileName) {
            // code to push fileName to DB
        }

        @Override
        public void onFailure(Throwable throwable) {
            // code to take appropriate action when there is an error
        }
    },
    executorService);
}
```

因此，在这个回调中，成功和异常场景都会被处理，这种使用回调的方式是很自然的。

我们**还可以通过直接将监听器添加到ListenableFuture来添加监听器**：

```java
listenableFuture.addListener(
    new Runnable() {
        @Override
        public void run() {
            // logic to download file
        }
    },
    executorService
);

```

此回调无法访问Future的结果，因为其输入为[Runnable](https://www.baeldung.com/java-runnable-callable)。无论Future是否成功完成，此回调方法都会执行。

在了解了ListenableFuture中的回调之后，下一节将探讨在CompletableFuture中实现相同功能的方法。

## 4. CompletableFuture中的回调

在CompletableFuture中，有多种方法可以附加回调，最流行的方法是使用链式方法，例如thenApply()、thenAccept()、thenCompose()、exceptionally()等，这些方法可以正常执行或异常执行。

在本节中，我们将学习whenComplete()方法，此方法最好的一点是它可以从任何希望它完成的线程中完成。通过上面的文件下载示例，我们来看看如何使用whenComplete()：

```java
CompletableFuture<String> completableFuture = new CompletableFuture<>();
Runnable runnable = downloadFile(completableFuture);
completableFuture.whenComplete(
    (res, error) -> {
        if (error != null) {
            // handle the exception scenario
        } else if (res != null) {
            // send data to DB
        }
    });
new Thread(runnable).start();

private static Runnable downloadFile(CompletableFuture<String> completableFuture) {
    return () -> {
        try {
            // logic to download to file;
        } catch (InterruptedException e) {
            log.error("exception is {} "+e);
        }
        completableFuture.complete("pic.jpg");
    };
}
```

下载文件完成后，whenComplete()方法执行成功或失败条件。

## 5. 总结

在这篇文章中，我们了解了ListenableFuture和CompletableFuture中的回调机制。

我们发现，**与CompletableFuture相比，ListenableFuture是一种更自然、流式的回调API**。

我们可以自由选择最适合我们用例的API，因为CompletableFuture是核心Java的一部分，而ListenableFuture是Guava库的一部分。