---
layout: post
title:  将Future转换为CompletableFuture
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 简介

在本教程中，我们将探讨如何将[Future](https://www.baeldung.com/java-future)转换为[CompletableFuture](https://www.baeldung.com/java-completablefuture)。通过这种转换，我们可以利用CompletableFuture的高级功能，例如非阻塞操作、任务链和更好的错误处理，同时仍然可以使用返回Future的API或库。

## 2. 为什么要将Future转换为CompletableFuture？

Java中的Future接口表示异步计算的结果，它提供了检查计算是否完成、等待计算完成以及检索结果的方法。

**但是Future也有局限性，比如阻塞调用需要使用get()来检索结果**，它还不支持链接多个异步任务或处理回调。

另一方面，**Java 8中引入的CompletableFuture解决了这些缺点。它通过[thenApply()](https://www.baeldung.com/java-completablefuture-thenapply-thenapplyasync#1-thenapply)和thenAccept()等方法支持非阻塞操作，用于任务链和回调，以及使用exceptionally()进行错误处理**。

通过将Future转换为CompletableFuture，我们可以利用这些功能，同时仍然可以使用返回Future的API或库。

## 3. 逐步转换

在本节中，我们将演示如何将Future转换为CompletableFuture。

### 3.1 使用ExecutorService模拟Future

为了理解Future的工作原理，我们首先使用[ExecutorService](https://www.baeldung.com/java-executor-service-tutorial)模拟异步计算。ExecutorService是一个用于在单独的线程中管理和调度任务的框架，这将帮助我们理解Future的阻塞性质：

```java
@Test
void givenFuture_whenGet_thenBlockingCall() throws ExecutionException, InterruptedException {
    ExecutorService executor = Executors.newSingleThreadExecutor();

    Future<String> future = executor.submit(() -> {
        Thread.sleep(1000);
        return "Hello from Future!";
    });

    String result = future.get();
    executor.shutdown();

    assertEquals("Hello from Future!", result);
}
```

在此代码中，我们使用executor.submit()模拟一个长时间运行的任务，该任务返回一个Future对象。**future.get()调用会阻塞主线程，直到计算完成，然后打印结果**。

这种阻塞行为凸显了Future的局限性之一，我们旨在通过CompletableFuture解决这一局限性。

### 3.2 将Future包装到CompletableFuture中

为了将Future转换为CompletableFuture，我们需要弥合Future的阻塞性质与CompletableFuture的非阻塞、回调驱动设计之间的差距。

为了实现这一点，我们创建一个名为toCompletableFuture()的方法，它以Future和ExecutorService作为输入并返回CompletableFuture：

```java
static <T> CompletableFuture<T> toCompletableFuture(Future<T> future, ExecutorService executor) {
    CompletableFuture<T> completableFuture = new CompletableFuture<>();
    executor.submit(() -> {
        try {
            completableFuture.complete(future.get());
        } catch (Exception e) {
            completableFuture.completeExceptionally(e);
        }
    });
    return completableFuture;
}
```

在上面的例子中，toCompletableFuture()方法首先创建一个新的CompletableFuture。**然后，来自缓存线程池的单独线程监视Future**。

当Future完成时，使用阻塞get()方法检索其结果，然后使用complete()方法传递给CompletableFuture。**如果Future抛出异常，CompletableFuture将异常完成，以确保错误得到传播**。

这个包装的CompletableFuture允许我们异步处理结果并使用thenAccept()之类的回调方法。让我们演示如何使用toCompletableFuture()：

```java
@Test
void givenFuture_whenWrappedInCompletableFuture_thenNonBlockingCall() throws ExecutionException, InterruptedException {
    ExecutorService executor = Executors.newSingleThreadExecutor();

    Future<String> future = executor.submit(() -> {
        Thread.sleep(1000);
        return "Hello from Future!";
    });

    CompletableFuture<String> completableFuture = toCompletableFuture(future, executor);

    completableFuture.thenAccept(result -> assertEquals("Hello from Future!", result));

    executor.shutdown();
}
```

与future.get()不同，这种方法避免了阻塞主线程并使代码更加灵活。**我们还可以将多个阶段链接在一起，从而实现更复杂的结果处理**。

例如，我们可以转换Future的结果，然后执行其他操作：

```java
@Test
void givenFuture_whenTransformedAndChained_thenCorrectResult() throws ExecutionException, InterruptedException {
    ExecutorService executor = Executors.newSingleThreadExecutor();

    Future<String> future = executor.submit(() -> {
        Thread.sleep(1000);
        return "Hello from Future!";
    });

    CompletableFuture<String> completableFuture = toCompletableFuture(future, executor);

    completableFuture
        .thenApply(result -> result.toUpperCase()) // Transform result
        .thenAccept(transformedResult -> assertEquals("HELLO FROM FUTURE!", transformedResult));

    executor.shutdown();
}
```

在此示例中，将结果转换为大写后，我们将转换后的结果打印出来。这展示了使用CompletableFuture进行链接操作的强大功能。

### 3.3 使用CompletableFuture的supplyAsync()方法

另一种方法是利用CompletableFuture的[supplyAsync()](https://www.baeldung.com/java-completablefuture-runasync-supplyasync#2-supplyasync)方法，它可以异步执行任务并将其结果作为CompletableFuture返回。

让我们看看如何将阻塞的Future调用包装在supplyAsync()方法内部以实现转换：

```java
@Test
void givenFuture_whenWrappedUsingSupplyAsync_thenNonBlockingCall() throws ExecutionException, InterruptedException {
    ExecutorService executor = Executors.newSingleThreadExecutor();

    Future<String> future = executor.submit(() -> {
        Thread.sleep(1000);
        return "Hello from Future!";
    });

    CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(() -> {
        try {
            return future.get();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    });

    completableFuture.thenAccept(result -> assertEquals("Hello from Future!", result));

    executor.shutdown();
}
```

在这种方法中，我们使用CompletableFuture.supplyAsync()异步执行任务。**该任务将阻塞调用future.get()包装在Lambda表达式中**。这样，Future的结果以非阻塞方式检索，使我们能够使用CompletableFuture方法进行回调和链接。

这种方法更简单，因为它避免了手动管理单独的线程，CompletableFuture为我们处理异步执行。

## 4. 将多个Future对象组合成一个CompletableFuture

在某些情况下，我们可能需要使用多个Future对象，这些对象应组合成单个CompletableFuture，**这在聚合不同任务的结果或等待所有任务完成后再进行进一步处理时很常见**。使用CompletableFuture，我们可以有效地组合多个Future对象并以非阻塞方式处理它们。

要组合多个Future对象，我们首先将它们转换为CompletableFuture实例。**然后，我们使用[CompletableFuture.allOf()](https://www.baeldung.com/java-completablefuture-allof-join)等待所有任务完成**。让我们看一个如何实现的示例：

```java
static CompletableFuture<Void> allOfFutures(List<Future<String>> futures, ExecutorService executor) {
    // Convert all Future objects into CompletableFuture instances
    List<CompletableFuture<String>> completableFutures = futures.stream()
        .map(future -> FutureToCompletableFuture.toCompletableFuture(future, executor))
        .toList();

    return CompletableFuture.allOf(completableFutures.toArray(new CompletableFuture[0]));
}
```

所有任务完成后，CompletableFuture.allOf()方法会发出完成信号。为了演示这一点，我们考虑一个场景，其中多个任务返回带有字符串作为结果的Future对象。我们将汇总结果并确保所有任务都成功完成：

```java
@Test
void givenMultipleFutures_whenCombinedWithAllOf_thenAllResultsAggregated() throws Exception {
    ExecutorService executor = Executors.newFixedThreadPool(3);

    List<Future<String>> futures = List.of(
        executor.submit(() -> "Task 1 Result"),
        executor.submit(() -> "Task 2 Result"),
        executor.submit(() -> "Task 3 Result")
    );

    CompletableFuture<Void> allOf = allOfFutures(futures, executor);

    allOf.thenRun(() -> {
        try {
            List<String> results = futures.stream()
                .map(future -> {
                    try {
                        return future.get();
                    } catch (Exception e) {
                        throw new RuntimeException(e);
                    }
                })
                .toList();
            assertEquals(3, results.size());
            assertTrue(results.contains("Task 1 Result"));
            assertTrue(results.contains("Task 2 Result"));
            assertTrue(results.contains("Task 3 Result"));
        } catch (Exception e) {
            fail("Unexpected exception: " + e.getMessage());
        }
    }).join();

    executor.shutdown();
}
```

在此示例中，我们模拟了三个任务，每个任务都使用ExecutorService返回一个结果。接下来，每个任务都会提交并返回一个Future对象。**我们将Future对象列表传递给allOfFutures()方法，该方法将它们转换为CompletableFuture并使用CompletableFuture.allOf()将它们组合起来**。

当所有任务完成后，我们使用thenRun()方法来汇总结果并断言其正确性，**这种方法在需要汇总结果的独立任务的并行处理等场景中非常有用**。

## 5. 总结

在本教程中，我们探讨了如何在Java中将Future转换为CompletableFuture。通过利用CompletableFuture，我们可以利用非阻塞操作、任务链和强大的异常处理。当我们想要增强异步编程模型的功能时，这种转换特别有用。