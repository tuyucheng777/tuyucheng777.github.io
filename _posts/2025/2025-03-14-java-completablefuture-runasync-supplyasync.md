---
layout: post
title:  Java中的CompletableFuture runAsync()与supplyAsync()
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 简介

Java的[CompletableFuture](https://www.baeldung.com/java-completablefuture)框架提供了强大的异步编程能力，可以方便地并发执行任务。

在本教程中，我们将深入研究CompletableFuture提供的两种基本方法-runAsync()和supplyAsync()，我们将探讨它们的区别、用例以及何时选择其中一种。

## 2. 理解CompletableFuture、runAsync()和supplyAsync()

**CompletableFuture是Java中一个强大的框架，它支持[异步编程](https://www.baeldung.com/java-asynchronous-programming)，有助于并发执行任务而不会阻塞主线程**，runAsync()和supplyAsync()是CompletableFuture类提供的方法。

在进行深入比较之前，让我们先了解一下runAsync()和supplyAsync()各自的功能。这两种方法都启动异步任务，使我们能够并发执行代码而不会阻塞主线程。

**runAsync()是一种用于异步执行不产生结果的任务的方法**，它适用于我们希望异步执行代码而不等待结果的即发即弃任务。例如，记录日志、发送通知或触发后台任务。

**另一方面，supplyAsync()是一种用于异步执行产生结果的任务的方法**，它非常适合需要结果进行进一步处理的任务。例如，从数据库获取数据、进行API调用或异步执行计算。

## 3. 输入与返回

runAsync()和supplyAsync()之间的主要区别在于它们接收的输入和它们产生的返回值的类型。

### 3.1 runAsync()

当要执行的异步任务没有产生任何结果时，将使用runAsync()方法。它接收[Runnable](https://www.baeldung.com/java-runnable-vs-extending-thread#implementing-a-runnable)函数式接口并异步启动任务，**它返回CompletableFuture<Void\>，对于重点在于完成任务而不是获得特定结果的场景很有用**。

以下代码片段展示了它的用法：

```java
CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
    // Perform non-result producing task
    System.out.println("Task executed asynchronously");
});
```

在此示例中，runAsync()方法用于异步执行不产生结果的任务，提供的Lambda表达式封装了要执行的任务。完成后，它会打印：

```text
Task completed successfully
```

### 3.2 supplyAsync()

另一方面，当异步任务产生结果时，将使用supplyAsync()。它接收[Supplier](https://www.baeldung.com/java-callable-vs-supplier#supplier)函数式接口并异步启动任务。**随后，它返回CompletableFuture<T\>，其中T是任务产生的结果的类型**。

让我们用一个例子来说明这一点：

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    // Perform result-producing task
    return "Result of the asynchronous computation";
});

// Get the result later
String result = future.get();
System.out.println("Result: " + result);
```

在此示例中，supplyAsync()用于异步执行产生结果的任务，supplyAsync()中的Lambda表达式表示异步计算结果的任务。完成后，它会打印获得的结果：

```text
Result: Result of the asynchronous computation
```

## 4. 异常处理

在本节中，我们将讨论这两种方法如何处理异常。

### 4.1 runAsync()

使用runAsync()时，异常处理非常简单，**该方法不提供在异步任务中处理异常的显式机制**。因此，在调用CompletableFuture上的get()方法时，任务执行期间抛出的任何异常都会传播到调用线程。这意味着我们必须在调用get()后手动处理异常。

以下是使用runAsync()进行异常处理的演示代码片段：

```java
CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
    throw new RuntimeException("Exception occurred in asynchronous task");
});

try {
    future.get(); // Exception will be thrown here
} catch (ExecutionException ex) {
    Throwable cause = ex.getCause();
    System.out.println("Exception caught: " + cause.getMessage());
}
```

然后打印异常消息：

```text
Exception caught: Exception occurred in asynchronous task
```

### 4.2 supplyAsync()

相比之下，supplyAsync()提供了一种更方便的异常处理方式。**它通过exceptionally()方法提供了一种异常处理机制**，此方法允许我们指定一个函数，如果原始异步任务异常完成，则将调用该函数，我们可以使用此函数来处理异常并返回默认值或执行任何必要的清理操作。

让我们看一个示例，演示supplyAsync()如何实现异常处理：

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    throw new RuntimeException("Exception occurred in asynchronous task");
}).exceptionally(ex -> {
    // Exception handling logic
    return "Default value";
});

String result = future.join(); // Get the result or default value
System.out.println("Result: " + result);
```

在此示例中，如果在执行异步任务期间发生异常，则会调用exceptionally()方法，这使我们能够妥善处理异常并在需要时提供后备值。

它不会引发异常，而是打印：

```text
Task completed with result: Default value
```

## 5. 执行行为

在本节中，我们将探讨CompletableFuture的runAsync()和supplyAsync()方法的执行行为。

### 5.1 runAsync()

使用runAsync()时，任务会立即在公共线程池中启动，其行为与调用new Thread(runnable).start()类似。**这意味着任务在调用后立即开始执行，没有任何延迟或调度考虑**。

### 5.2 supplyAsync()

另一方面，supplyAsync()将任务安排在公共线程池中，如果其他任务排队，则可能会延迟其执行。**这种调度方法有利于资源管理，因为它有助于防止线程创建的突然爆发**。通过对任务进行排队并根据线程的可用性安排其执行，supplyAsync()可确保高效的资源利用率。

## 6. 链式操作

在本节中，我们将探讨CompletableFuture的runAsync()和supplyAsync()方法如何支持链式操作，并强调它们的区别。

### 6.1 runAsync()

runAsync()方法不能直接与thenApply()或thenAccept()等方法链接，因为它不会产生结果。**但是，我们可以在runAsync()任务完成后使用thenRun()执行另一项任务**，此方法允许我们链接其他任务以进行顺序执行，而无需依赖初始任务的结果。

下面是一个展示使用runAsync()和thenRun()的链式操作的示例：

```java
CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
    System.out.println("Task executed asynchronously");
});

future.thenRun(() -> {
    // Execute another task after the completion of runAsync()
    System.out.println("Another task executed after runAsync() completes");
});
```

在此示例中，我们首先使用runAsync()异步执行任务。然后，我们使用thenRun()指定在初始任务完成后要执行的另一个任务，这使我们能够按顺序链接多个任务，从而得到以下输出：

```text
Task executed asynchronously
Another task executed after runAsync() completes
```

### 6.2 supplyAsync()

相比之下，supplyAsync()允许链式操作，因为它有返回值。由于supplyAsync()会产生结果，因此我们可以使用thenApply()等方法来转换结果，使用thenAccept()来使用结果，或者使用thenCompose()来链接进一步的异步操作，这种灵活性使我们能够通过将多个任务链接在一起来构建复杂的异步工作流。

下面是一个示例，演示supplyAsync()和thenApply()的链接操作：

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    return "Result of the asynchronous computation";
});

future.thenApply(result -> {
    // Transform the result
    return result.toUpperCase();
}).thenAccept(transformedResult -> {
    // Consume the transformed result
    System.out.println("Transformed Result: " + transformedResult);
});
```

在此示例中，我们首先使用supplyAsync()异步执行任务，该任务会产生结果。然后，我们使用thenApply()转换结果，并使用thenAccept()使用转换后的结果。这演示了使用supplyAsync()链接多个操作，从而实现更复杂的异步工作流。

以下是输出示例：

```text
Transformed Result: RESULT OF THE ASYNCHRONOUS COMPUTATION
```

## 7. 性能

虽然runAsync()和supplyAsync()都异步执行任务，但它们的[性能](https://www.baeldung.com/java-completablefuture-non-blocking)特征会根据任务的性质和底层执行环境而有所不同。

### 7.1 runAsync()

由于runAsync()不会产生任何结果，因此与supplyAsync()相比，它的性能可能略好一些，这是**因为它避免了创建Supplier对象的开销**。在某些情况下，没有结果处理逻辑可以加快任务执行速度。

### 7.2 supplyAsync()

各种因素都会影响supplyAsync()的性能，包括任务的复杂性、资源的可用性以及底层执行环境的效率。

在任务涉及复杂计算或资源密集型操作的情况下，使用supplyAsync()对性能的影响可能更为明显。但是，处理结果和任务之间依赖关系的能力可以抵消任何潜在的性能开销。

## 8. 用例

在本节中，我们将探讨这两种方法的具体用例。

### 8.1 runAsync()

当重点在于完成任务而不是获取特定结果时，runAsync()方法特别有用。**runAsync()通常用于执行后台任务或不需要返回值的操作的场景**，例如，运行定期清理例程、记录事件或触发通知都可以通过runAsync()高效实现。

### 8.2 supplyAsync()

与runAsync()相比，supplyAsync()方法在任务完成涉及生成可在应用程序流程稍后使用的值时特别有用。supplyAsync()的一个典型用例是从外部源(例如数据库、API或远程服务器)获取数据。

**此外，supplyAsync()适合执行产生结果值的计算任务，例如执行复杂的计算或处理输入数据**。

## 9. 比较

下面是一个汇总表，比较了runAsync()和supplyAsync()之间的主要区别：

|   特征   |            runAsync()            |             supplyAsync()              |
| :------: | :------------------------------: | :------------------------------------: |
|   输入   |   接收表示无结果任务的Runnable   |  接收代表产生结果的任务的Supplier<T\>  |
| 返回类型 |     CompletableFuture<Void\>     | CompletableFuture<T\>(其中T是结果类型) |
|   用例   |     没有结果的“即发即弃”任务     |      需要结果进行进一步处理的任务      |
| 异常处理 | 没有内置机制；异常会传播给调用者 | 提供exceptionly()以实现优雅的异常处理  |
| 执行行为 |           立即启动任务           |         安排可能延迟执行的任务         |
| 链接操作 |    支持thenRun()执行后续任务     |    支持thenApply()等方法来链接任务     |
|   性能   |        可能会有更好的性能        |       性能受任务复杂性和资源影响       |
| 使用场景 |   后台任务、定期例行事务、通知   |    数据获取、计算任务、结果依赖任务    |

## 10. 总结

在本文中，我们探讨了runAsync()和supplyAsync()方法。我们讨论了它们的功能、差异、异常处理机制、执行行为、链式操作、性能注意事项和具体用例。

当需要结果时，supplyAsync()是首选，而runAsync()适用于仅关注任务完成而不需要特定结果的场景。