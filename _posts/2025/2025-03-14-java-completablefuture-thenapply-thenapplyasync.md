---
layout: post
title:  CompletableFuture中thenApply()和thenApplyAsync()之间的区别
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 简介

在[CompletableFuture](https://www.baeldung.com/java-completablefuture)框架中，thenApply()和thenApplyAsync()是促进异步编程的重要方法。

在本教程中，我们将深入研究CompletableFuture中thenApply()和thenApplyAsync()之间的区别。我们将探讨它们的功能、用例以及何时选择其中一个。

## 2. 理解thenApply()和thenApplyAsync()

CompletableFuture提供thenApply()和thenApplyAsync()方法，用于对计算结果应用转换，**这两种方法都允许对CompletableFuture的结果执行链式操作**。

### 2.1 thenApply()

thenApply()是一种在CompletableFuture完成时将函数应用于其结果的方法，**它接收[Function](https://www.baeldung.com/java-8-functional-interfaces)函数接口，将函数应用于结果，并返回一个带有转换结果的新CompletableFuture**。

### 2.2 thenApplyAsync()

thenApplyAsync()是一种异步执行所提供函数的方法，它接收一个Function函数接口和一个可选的[Executor](https://www.baeldung.com/java-executor-service-tutorial)，并返回一个带有异步转换结果的新CompletableFuture。

## 3. 执行线程

thenApply()和thenApplyAsync()之间的主要区别在于它们的执行行为。

### 3.1 thenApply()

**默认情况下，thenApply()方法使用完成当前CompletableFuture的同一线程执行转换函数**，这意味着转换函数的执行可能在结果可用后立即发生，如果转换函数运行时间长或资源密集，这可能会阻塞线程。

但是，如果我们在尚未完成的CompletableFuture上调用thenApply()，它会在[执行器池中](https://www.baeldung.com/thread-pool-java-and-guava#2-threadpoolexecutor)的另一个线程中异步执行转换函数。

下面是演示thenApply()的代码片段：

```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> 5);
CompletableFuture<String> thenApplyResultFuture = future.thenApply(num -> "Result: " + num);

String thenApplyResult = thenApplyResultFuture.join();
assertEquals("Result: 5", thenApplyResult);
```

在此示例中，如果结果已可用且当前线程兼容，则thenApply()可能会同步执行该函数。但是，需要注意的是，**CompletableFuture会根据各种因素(例如结果的可用性和线程上下文)智能地决定是同步执行还是异步执行**。

### 3.2 thenApplyAsync()

**相比之下，thenApplyAsync()通过利用执行器池中的线程(通常是ForkJoinPool.commonPool())来保证异步执行提供的函数**，这确保该函数异步执行并可以在单独的线程中运行，从而防止当前线程被阻塞。

以下是我们使用thenApplyAsync()的方法：

```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> 5);
CompletableFuture<String> thenApplyAsyncResultFuture = future.thenApplyAsync(num -> "Result: " + num);

String thenApplyAsyncResult = thenApplyAsyncResultFuture.join();
assertEquals("Result: 5", thenApplyAsyncResult);
```

在这个例子中，即使结果立即可用，thenApplyAsync()也总是安排该函数在单独的线程上异步执行。

## 4. 控制线程

虽然thenApply()和thenApplyAsync()都支持异步转换，但它们在指定自定义执行器并因此控制执行线程的支持方面有所不同。

### 4.1 thenApply()

thenApply()方法不直接支持指定自定义执行器来控制执行线程，它依赖于CompletableFuture的默认行为，该行为可能在完成上一阶段的同一线程上执行转换函数，通常是来自公共池的线程。

### 4.2 thenApplyAsync()

**相比之下，thenApplyAsync()允许我们显式指定执行器来控制执行线程**。通过提供自定义执行器，我们可以指定转换函数的执行位置，从而实现更精确的线程管理。

下面我们来演示使用thenApplyAsync()的自定义执行器：

```java
ExecutorService customExecutor = Executors.newFixedThreadPool(4);

CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
    try {
        Thread.sleep(2000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    return 5;
}, customExecutor);

CompletableFuture<String> resultFuture = future.thenApplyAsync(num -> "Result: " + num, customExecutor);

String result = resultFuture.join();
assertEquals("Result: 5", result);

customExecutor.shutdown();
```

在此示例中，创建了一个固定线程池大小为4的自定义执行器，thenApplyAsync()方法随后使用此自定义执行器，为转换函数提供对执行线程的控制。

## 5. 异常处理

thenApply()和thenApplyAsync()在异常处理上的主要区别在于异常何时以及如何变得可见。

### 5.1 thenApply()

如果提供给thenApply()的转换函数抛出异常，则thenApply()阶段会立即以异常方式完成CompletableFuture。此异常完成将抛出的异常包含在CompletionException中，包装原始异常。

让我们用一个例子来说明这一点：

```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> 5);
CompletableFuture<String> resultFuture = future.thenApply(num -> "Result: " + num / 0);
assertThrows(CompletionException.class, () -> resultFuture.join());
```

在此示例中，我们尝试将5除以0，结果引发了ArithmeticException。**此CompletionException直接传播到下一阶段或调用者，这意味着函数中的任何异常都会立即可见并可供处理**。因此，如果我们尝试使用get()、join()或thenAccept()等方法访问结果，我们会直接遇到CompletionException：

```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> 5);
CompletableFuture<String> resultFuture = future.thenApply(num -> "Result: " + num / 0);
try {
    // Accessing the result using join()
    String result = resultFuture.join();
    assertEquals("Result: 5", result);
} catch (CompletionException e) {
    assertEquals("java.lang.ArithmeticException: / by zero", e.getMessage());
}
```

在此示例中，函数传递到thenApply()期间抛出的异常，该阶段识别出问题并将原始异常包装在CompletionException中，以便我们进一步处理它。

### 5.2 thenApplyAsync()

**虽然转换函数异步运行，但其中的任何异常都不会直接传播到返回的CompletableFuture**。当我们调用get()、join()或thenAccept()等方法时，异常不会立即可见。这些方法会阻塞，直到异步操作完成，如果处理不当，可能会导致死锁。

为了处理thenApplyAsync()中的异常，我们必须使用专用方法，如handle()、[exceptionally()](https://www.baeldung.com/java-exceptions-completablefuture)或whenComplete()，这些方法允许我们在异步发生异常时拦截并处理异常。

下面是演示使用handle进行明确处理的代码片段：

```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> 5);
CompletableFuture<String> thenApplyAsyncResultFuture = future.thenApplyAsync(num -> "Result: " + num / 0);

String result = thenApplyAsyncResultFuture.handle((res, error) -> {
      if (error != null) {
          // Handle the error appropriately, e.g., return a default value
          return "Error occurred";
      } else {
          return res;
      }
  })
  .join(); // Now join() won't throw the exception
assertEquals("Error occurred", result);
```

在这个例子中，尽管在thenApplyAsync()中发生了异常，但它在resultFuture中并不直接可见。join()方法会阻塞并最终解包CompletionException，从而显示原始的ArithmeticException。

## 6. 用例

在本节中，我们将探讨CompletableFuture中thenApply()和thenApplyAsync()方法的常见用例。

### 6.1 thenApply()

thenApply()方法在下列场景中特别有用：

- 顺序转换：**需要按顺序对CompletableFuture的结果进行转换**，这可能涉及将数字结果转换为字符串或根据结果执行计算等任务。
- 轻量级操作：**它非常适合执行小型、快速的转换，不会对调用线程造成严重阻塞**；示例包括将数字转换为字符串、根据结果执行计算或操作数据结构。

### 6.2 thenApplyAsync()

另一方面，thenApplyAsync()方法适用于以下情况：

- 异步转换：**当需要异步应用转换时，可能会利用多个线程进行并行执行**。例如，在用户上传图像进行编辑的Web应用程序中，使用CompletableFuture进行异步转换有利于同时应用调整大小、滤镜和水印，从而提高处理效率和用户体验。
- 阻塞操作：**在转换函数涉及阻塞操作、I/O操作或计算密集型任务的情况下，thenApplyAsync()会变得有利**。通过将此类计算卸载到单独的线程，有助于防止阻塞调用线程，从而确保更流畅的应用程序性能。

## 7. 对比

下面是一个汇总表，比较了thenApply()和thenApplyAsync()之间的主要区别。

|    特征    |                         thenApply()                         | thenApplyAsync() |
|:--------:| :----------------------------------------------------------: |:----------------:|
|   执行行为   | 与前一阶段相同的线程或来自执行器池的单独线程(如果在完成之前调用) |    将线程与执行器池分开    |
| 自定义执行器支持 |                            不支持                            |  支持自定义执行器进行线程控制  |
|   异常处理   |            立即在CompletionException中传播异常             |  异常不直接可见，需要明确处理  |
|    性能    |                      可能会阻塞调用线程                      |   可以避免阻塞并提高性能    |
|   使用场景   |                     顺序转换，轻量级操作                     |    异步转换、阻塞操作     |

## 8. 总结

在本文中，我们探讨了CompletableFuture框架中thenApply()和thenApplyAsync()方法之间的功能和差异。

**thenApply()可能会阻塞线程**，因此它适合轻量级转换或可以接受同步执行的场景。另一方面，**thenApplyAsync()保证异步执行**，因此它非常适合涉及潜在阻塞的操作或计算密集型任务，这些任务对响应性至关重要。