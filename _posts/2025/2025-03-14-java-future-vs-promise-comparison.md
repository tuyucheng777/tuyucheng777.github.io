---
layout: post
title:  Java中Future和Promise的区别
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 简介

[Future](https://www.baeldung.com/java-future)和Promise是用于处理异步任务的工具，允许人们无需等待每个步骤完成即可执行操作。虽然它们都具有相同的用途，但它们表现出关键差异。在本教程中，我们将探讨Future和Promise之间的差异，仔细研究它们的主要特征、用例和独特功能。

## 2. 理解Future

Future充当容器，等待正在进行的操作的结果。开发人员通常使用Future来检查计算状态、在准备就绪时检索结果或优雅地等待操作结束。Future通常与[Executor](https://www.baeldung.com/java-executor-service-tutorial)框架集成，提供一种直接而有效的方法来处理异步任务。 

### 2.1 主要特点

现在，让我们探索一下Future的一些主要特征：

- **采用阻塞设计，这可能会导致等待异步计算完成**。
- 与正在进行的计算的直接交互受到限制，以保持一种直接的方法。

### 2.2 使用场景

**在异步操作的结果是预先确定的并且一旦开始就无法改变的场景中，Future表现出色**。

考虑从数据库获取用户的个人资料信息或从远程服务器下载文件。一旦启动，这些操作就会有一个固定的结果，例如检索到的数据或下载的文件，并且不能在过程中进行修改。

### 2.3 使用Future

要使用Future，我们可以在java.util.concurrent包中找到它们。让我们看一段代码片段，演示如何使用Future进行异步任务处理：

```java
ExecutorService executorService = Executors.newSingleThreadExecutor();

Future<String> futureResult = executorService.submit(() -> {
    Thread.sleep(2000);
    return "Future Result";
});

while (!futureResult.isDone()) {
        System.out.println("Future task is still in progress...");
    Thread.sleep(500);
}

String resultFromFuture = futureResult.get();
System.out.println("Future Result: " + resultFromFuture);

executorService.shutdown();
```

让我们检查一下运行代码时得到的输出：

```text
Future task is still in progress...
Future task is still in progress...
Future task is still in progress...
Future task is still in progress...
Future Result: Future Result
```

代码中，futureResult.get()方法是阻塞调用。这意味着当程序到达这一行时，**它会等待提交给ExecutorService的异步任务完成后再继续执行**。

## 3. 理解Promise

相比之下，Promise的概念并非Java所固有，而是其他编程语言中的一种通用抽象。**Promise充当创建Promise时可能不知道的值的代理**，与Future不同，Promise通常提供更具交互性的方法，允许开发人员在启动异步计算后影响它。

### 3.1 主要特点

现在，让我们探讨一下Promise的一些主要特征：

- 封装可变状态，即使在异步操作开始后也允许修改，从而为处理动态场景提供了灵活性
- 采用回调机制，允许开发人员附加在异步操作完成、失败或进展时执行的回调

### 3.2 使用场景

**Promise非常适合需要动态和交互式控制异步操作的场景**。此外，Promise还提供了在启动后修改正在进行的计算的灵活性。一个很好的例子是金融应用程序中的实时数据流，其中显示内容需要适应实时市场变化。

此外，Promise在处理需要条件分支或基于中间结果进行修改的异步任务时非常有用。一种可能的用例是，当我们需要处理多个异步API调用时，后续操作取决于前一个操作的结果。

### 3.3 使用Promise

Java可能没有像JavaScript那样严格遵守Promise规范的专用Promise类，**但是，我们可以使用[java.util.concurrent.CompletableFuture](https://www.baeldung.com/java-completablefuture-unit-test)实现类似的功能**。CompletableFuture提供了一种处理异步任务的多功能方法，与Promise共享一些特性。需要注意的是，它们并不相同。

让我们探索如何使用CompletableFuture在Java中实现类似Promise的行为：

```java
ExecutorService executorService = Executors.newSingleThreadExecutor();
CompletableFuture<String> completableFutureResult = CompletableFuture.supplyAsync(() -> {
    try {
        Thread.sleep(2000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    return "CompletableFuture Result";
}, executorService);

completableFutureResult.thenAccept(result -> {
        System.out.println("Promise Result: " + result);
  })
          .exceptionally(throwable -> {
        System.err.println("Error occurred: " + throwable.getMessage());
        return null;
        });

        System.out.println("Doing other tasks...");

executorService.shutdown();
```

当我们运行代码时，我们将看到输出：

```text
Doing other tasks...
Promise Result: CompletableFuture Result
```

我们创建一个名为completableFutureResult的CompletableFuture，supplyAsync()方法用于启动异步计算，提供的Lambda函数表示异步任务。

接下来，我们使用thenAccept()和exceptionally()将回调附加到CompletableFuture。thenAccept()回调处理异步任务的成功完成，类似于Promise的解析，而exceptionally()处理任务期间可能发生的任何异常，类似于Promise的拒绝。

## 4. 主要区别

### 4.1 控制流

一旦设置了Future的值，控制流就会向下游进行，不受后续事件或更改的影响。同时，Promise(或CompletableFuture)提供了thenCompose()和whenComplete()等方法，可根据最终结果或异常进行条件执行。

让我们使用CompletableFuture创建一个分支控制流的示例：

```java
CompletableFuture<Integer> firstTask = CompletableFuture.supplyAsync(() -> {
      return 1;
  })
  .thenApplyAsync(result -> {
      return result * 2;
  })
  .whenComplete((result, ex) -> {
      if (ex != null) {
          // handle error here
      }
  });
```

在代码中，我们使用thenApplyAsync()方法来演示异步任务的链接。

### 4.2 错误处理

Future和Promise都提供了处理错误和异常的机制，Future依赖于计算过程中抛出的异常：

```java
ExecutorService executorService = Executors.newSingleThreadExecutor();
Future<String> futureWithError = executorService.submit(() -> {
    throw new RuntimeException("An error occurred");
});

try {
    String result = futureWithError.get();
} catch (InterruptedException | ExecutionException e) {
    e.printStackTrace();
} finally {
    executorService.shutdown();
}
```

在CompletableFuture中，exceptionally()方法用于处理异步计算过程中发生的任何异常。如果发生异常，它会打印错误消息并提供回退值：

```java
CompletableFuture<String> promiseWithError = new CompletableFuture<>();
promiseWithError.completeExceptionally(new RuntimeException("An error occurred"));

promiseWithError.exceptionally(throwable -> {
    return "Fallback value";
});
```

### 4.3 读写访问

Future提供了一个只读视图，允许我们在计算完成后检索结果：

```java
Future<Integer> future = executor.submit(() -> 100);
// Cannot modify future.get() after completion
```

相比之下，CompletableFuture不仅使我们能够读取结果，而且还可以在异步操作开始后主动动态设置值：

```java
ExecutorService executorService = Executors.newSingleThreadExecutor();
CompletableFuture<Integer> totalPromise = CompletableFuture.supplyAsync(() -> {
    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    return 100;
}, executorService);

totalPromise.thenAccept(value -> System.out.println("Total $" + value ));
totalPromise.complete(10);
```

最初，我们将异步任务设置为返回100作为结果。但是，我们在任务自然完成之前进行干预并明确使用值10完成任务。这种灵活性凸显了CompletableFuture的可写入特性，使我们能够在异步执行期间动态更新结果。

## 5. 总结

在本文中，我们探讨了Future和Promise之间的区别。虽然两者都用于处理异步任务，但它们的功能却有很大差异。