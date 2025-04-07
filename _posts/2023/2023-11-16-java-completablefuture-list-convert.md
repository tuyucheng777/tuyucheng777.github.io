---
layout: post
title:  将CompletableFuture列表转换为List的CompletableFuture
category: java-concurrency
copyright: java-concurrency
excerpt: Java CompletableFuture
---

## 1. 概述

在本教程中，我们将学习如何将List<[CompletableFuture](https://www.baeldung.com/java-completablefuture) <T\>\>对象转换为CompletableFuture<List<T\>\>。

在许多情况下，这种转换非常有用。**一个典型的例子是，当我们必须多次调用远程服务(通常是异步操作)并将结果聚合到单个List中时**。此外，我们最终会等待一个CompletableFuture对象，该对象在所有操作完成时为我们提供结果列表，或者在一个或多个操作失败时抛出异常。

我们将首先看到一种简单的转换方法，然后再看看一种更简单、更安全的方法。

## 2. 链接CompletableFuture

实现此目的的一种方法是使用其thenCompose()方法链接CompletableFuture。这样，我们就可以创建一个单一对象，一旦所有先前的Future都一一解决，该对象就会解决，类似于多米诺骨牌构造。

### 2.1 实现

首先，让我们创建一个模拟异步操作：

```java
public class Application {
    ScheduledExecutorService asyncOperationEmulation;

    Application initialize() {
        asyncOperationEmulation = Executors.newScheduledThreadPool(10);
        return this;
    }

    CompletableFuture<String> asyncOperation(String operationId) {
        CompletableFuture<String> cf = new CompletableFuture<>();
        asyncOperationEmulation.submit(() -> {
            Thread.sleep(100);
            cf.complete(operationId);
        });
        return cf;
    }
}
```

我们创建了一个Application类来托管我们的测试代码和asyncOperation()方法，该方法仅休眠100毫秒。我们使用具有10个线程的Executor来异步调度所有内容。

为了收集所有操作结果(在本例中为简单的operationId字符串)，我们将链接从asyncOperation()方法生成的CompletableFuture：

```java
void startNaive() {
    List<CompletableFuture<String>> futures = new ArrayList<>();
    for (int i = 1; i <= 1000; i++) {
        String operationId = "Naive-Operation-" + i;
        futures.add(asyncOperation(operationId));
    }
    CompletableFuture<List<String>> aggregate = CompletableFuture.completedFuture(new ArrayList<>());
    for (CompletableFuture<String> future : futures) {
        aggregate = aggregate.thenCompose(list -> {
            list.add(future.get());
            return CompletableFuture.completedFuture(list);
        });
    }
    final List<String> results = aggregate.join();
    for (int i = 0; i < 10; i++) {
        System.out.println("Finished " + results.get(i));
    }

    close();
}
```

我们首先使用静态的CompletedFuture()方法创建一个已完成的CompletableFuture，并提供一个空List作为完成结果。使用thenCompose()我们创建一个Runnable，它在前一个Future完成后立即执行。**thenCompose()方法返回一个新的CompletableFuture，一旦第一个和第二个Future完成，它就会解析**。我们用这个新的Future对象替换聚合引用，这使我们能够在Future列表的迭代循环内继续链接这些调用。

在我们创建的Runnable内部，我们等待Future完成并将结果添加到List中。然后我们返回一个完整的Future，其中包含List和结果，这会将List进一步传递到thenCompose()链，让我们将Future的结果一一添加。

一旦所有Future都被链接起来，我们就对聚合CompletableFuture调用join()。这是专门针对该示例完成的，以便我们可以检索结果并阻止主Java线程在聚合完成之前退出。在真正的异步场景中，我们可能会在thenAccept()或whenComplete()调用中添加回调逻辑。 

需要注意的一件事是我们在最后添加了一个close()调用，实现如下：

```java
void close() {
    asyncOperationEmulation.shutdownNow();
}
```

**应用程序退出时必须关闭所有Executor，否则Java进程将挂起**。

### 2.2 实现问题

这种简单的实现存在一些问题，**Future链不仅引入了不必要的复杂性，而且还创建了大量不需要的对象**，例如thenCompose()生成的所有新CompletableFuture。

当我们执行大量操作时，会出现另一个潜在问题。**如果操作失败，并且根据Java实现如何解析CompletableFuture链，如果解析是递归完成的，我们可能会得到StackOverflowError**。

为了测试异常场景，我们可以通过更改asyncOperation()方法在其中一项操作上引入错误：

```java
CompletableFuture<String> asyncOperation(String operationId) {
    CompletableFuture<String> cf = new CompletableFuture<>();
    asyncOperationEmulation.submit(() -> {
        if (operationId.endsWith("567")) {
            cf.completeExceptionally(new Exception("Error on operation " + operationId));
            return;
        }
        Thread.sleep(100);
        cf.complete(operationId);
    });
    return cf;
}
```

在这种情况下，第567个操作的Future将异常完成，从而使aggregate.join()调用也会抛出运行时异常。

## 3. 使用CompletableFuture.allOf()

一种不同且更好的方法是使用CompletableFuture API的allOf()方法。**此方法接收一组CompletableFuture对象并创建一个新对象，该新对象在所有提供的Future本身解析时解析**。

此外，如果其中一个Future失败，那么整个Future也会失败。**新的Future不包含结果列表**，要获取它们，我们必须检查相应的CompletableFuture对象。

### 3.1 实现

让我们使用allOf()创建一个新的start()方法 ：

```java
void start() {
    List<CompletableFuture<String>> futures = new ArrayList<>();
    for (int i = 1; i <= 1000; i++) {
        String operationId = "Operation-" + i;
        futures.add(asyncOperation(operationId));
    }
    CompletableFuture<?>[] futuresArray = futures.toArray(new CompletableFuture<?>[0]);
    CompletableFuture<List<String>> listFuture = CompletableFuture.allOf(futuresArray)
      .thenApply(v -> futures.stream().map(CompletableFuture::join).collect(Collectors.toList()));
    final List<String> results = listFuture.join();
    System.out.println("Printing first 10 results");
    for (int i = 0; i < 10; i++) {
        System.out.println("Finished " + results.get(i));
    }

    close();
}
```

设置和结果打印是相同的，但是我们现在有一个futuresArray并将其提供给allOf()。我们使用thenApply()在allOf()解析后添加逻辑，**在此回调中，我们使用[CompletableFuture.join()](https://www.baeldung.com/java-completablefuture-allof-join)方法收集所有Future结果并将它们收集到List中**。该列表是thenApply()生成的CompletableFuture中包含的结果，即listFuture。

为了展示聚合结果，我们使用join()方法，该方法会阻塞主线程，直到listFuture完成。最后，不要忘记close()的调用。

### 3.2 allOf()的优点

基于allOf()的实现比链接Future更简单、更清晰。它不会生成不需要的对象，从而减少内存占用。此外，它为我们提供了一种处理CompletableFuture的任意组合的简单方法。

## 4. 总结

在本文中，我们学习了如何将List<CompletableFuture<T\>\>转换为 CompletableFuture<List<T\>\>。我们理解了为什么这种转换很有用，并看到了两种实现方式，一种是简单的实现，另一种是使用正确的Java API。我们讨论了前者的潜在问题以及后者如何避免这些问题。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-concurrency-basic-3)上获得。