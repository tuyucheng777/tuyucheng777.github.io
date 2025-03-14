---
layout: post
title:  ExecutorService中execute()和submit()的区别
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

##  1. 概述

多线程和并行处理是现代应用程序开发中的关键概念，**在Java中，Executor框架提供了一种高效管理和控制并发任务执行的方法**。[ExecutorService](https://www.baeldung.com/java-executor-service-tutorial)接口是此框架的核心，它提供了两种常用的提交要执行的任务的方法：submit()和execute()。

在本文中，我们将探讨这两种方法之间的主要区别。我们将使用一个简单的示例来查看submit()和execute()，在这个示例中，我们模拟使用[线程池](https://www.baeldung.com/thread-pool-java-and-guava)计算数组中数字总和的任务。

## 2. ExecutorService.submit()的用法 

我们先从ExecutorService接口中广泛使用的submit()方法开始，**它允许我们提交要执行的任务，并返回一个表示计算结果的[Future](https://www.baeldung.com/java-future)对象**。

然后，这个Future允许我们获取计算结果、处理任务执行期间发生的异常以及监视任务的状态。**我们可以在Future中调用get()来检索结果或异常**。

让我们首先初始化ExecutorService：

```java
ExecutorService executorService = Executors.newFixedThreadPool(2);
```

这里，我们使用大小为2的固定线程池初始化ExecutorService。这将创建一个线程池，该线程池利用固定数量的线程在共享的[无界队列](https://www.baeldung.com/java-blocking-queue)上运行。在我们的例子中，在任何时候，最多有两个线程处于活动处理任务的状态。如果在处理所有现有任务时发送了更多任务，这些任务将被保留在队列中，直到处理线程空闲为止。

接下来，让我们使用[Callable](https://www.baeldung.com/java-runnable-callable)创建一个任务：

```java
Callable<Integer> task = () -> {
    int[] numbers = {1, 2, 3, 4, 5};
    int sum = 0;
    for (int num : numbers) {
        sum += num;
    }
    return sum;
};
```

重要的是，这里的Callable对象表示返回结果并可能抛出异常的任务。在这种情况下，它表示另一个线程可以执行并返回结果或可能抛出异常的任务。此Callable计算数组中整数的总和并返回结果。

现在我们已经将任务定义为Callable，让我们将该任务提交给ExecutorService：

```java
Future<Integer> result = executorService.submit(task);
```

**简而言之，submit()方法接收Callable任务并将其提交给ExecutorService执行**。它返回一个Future<Integer\>对象，该对象表示计算的未来结果。总的来说，executorService.submit()是一种使用ExecutorService异步执行Callable任务的方法，允许并发执行任务并通过返回的Future获取其结果

最后我们来看一下结果：

```java
try {
    int sum = result.get();
    System.out.println("Sum calculated using submit:" + sum);
} catch (InterruptedException | ExecutionException e) {
    e.printStackTrace();
}
```

这里，get()检索任务执行的结果。在本例中，它获取任务计算的总和并打印出来。**但是，需要注意的是，get()方法是一个阻塞调用，如果结果尚未可用，则会等待，这可能会导致线程暂停，直到结果准备好**。如果计算在运行时遇到问题，它还可能抛出InterruptedException或ExecutionException等异常。

最后，让我们关闭ExecutorService：

```java
executorService.shutdown();
```

这会在任务执行完成后关闭ExecutorService并释放服务使用的所有资源。

## 3. ExecutorService.execute()的用法 

execute()方法是一个更简单的方法，定义在Executor接口中，该接口是ExecutorService的父接口。**它用于提交要执行的任务，但不返回Future。这意味着我们无法直接通过Future对象获取任务的结果或处理异常**。

它适用于我们不需要等待任务结果并且不希望出现任何异常的场景，这些任务是为了它们的副作用而执行的。

和以前一样，我们将创建一个固定线程池为2的ExecutorService。但是，我们将创建一个Runnable任务：

```java
Runnable task = () -> {
    int[] numbers = {1, 2, 3, 4, 5};
    int sum = 0;
    for (int num : numbers) {
        sum += num;
    }
    System.out.println("Sum calculated using execute: " + sum);
};
```

重要的是，该任务不返回任何结果；它只是计算总和并将其打印在任务内部。我们现在将Runnable任务提交给ExecutorService：

```java
executorService.execute(task);
```

**这仍然是一个异步操作，表明线程池中的一个线程执行该任务**。

## 4. 总结

在本文中，我们了解了ExecutorService接口中的submit()和execute()的显著特性和用法。总之，这两种方法都提供了一种提交任务并发执行的方法，但它们在处理任务结果和异常方面有所不同。

submit()和execute()之间的选择取决于具体要求，**如果我们需要获取任务结果或处理异常，则应使用submit()。另一方面，如果我们有一个不返回结果的任务，则execute()是正确的选择**。

此外，如果我们处理更复杂的用例并且需要灵活地管理任务和检索结果或异常，则submit()是更好的选择。