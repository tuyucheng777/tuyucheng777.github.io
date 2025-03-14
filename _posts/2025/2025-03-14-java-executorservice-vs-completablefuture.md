---
layout: post
title:  ExecutorService与CompletableFuture指南
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 简介

在本教程中，我们将探讨两个用于处理需要并发运行的任务的重要Java类：[ExecutorService](https://www.baeldung.com/java-executor-service-tutorial)和[CompletableFuture](https://www.baeldung.com/java-completablefuture)。我们将学习它们的功能以及如何有效地使用它们，并了解它们之间的主要区别。

## 2. ExecutorService概述

ExecutorService是Java的java.util.concurrent包中的一个功能强大的接口，它简化了需要并发运行的任务的管理。**它抽象了线程创建、管理和调度的复杂性，使我们能够专注于需要完成的实际工作**。

ExecutorService提供submit()和execute()等方法来提交我们想要并发运行的任务，然后这些任务被排队并分配给[线程池](https://www.baeldung.com/thread-pool-java-and-guava)中的可用线程。如果任务返回结果，我们可以使用Future对象来检索它们。**但是，使用Future上的get()等方法检索结果可能会阻塞调用线程，直到任务完成**。

## 3. CompletableFuture概述

CompletableFuture是在Java 8中引入的，**它专注于编写异步操作并以更具声明性的方式处理其最终结果**。CompletableFuture充当保存异步操作最终结果的容器，它可能不会立即产生结果，但它提供了定义在结果可用时要做什么的方法。

与ExecutorService检索结果可能会阻塞线程不同，**CompletableFuture以非阻塞方式运行**。

## 4. 重点与职责

虽然ExecutorService和CompletableFuture都用于Java中的异步编程，但它们的用途不同。让我们探索它们各自的重点和职责。

### 4.1 ExecutorService

**ExecutorService专注于管理线程池和并发执行任务**，它提供了创建具有不同配置的线程池的方法，例如固定大小、缓存和调度。

让我们看一个使用ExecutorService创建并维护三个线程的示例：

```java
ExecutorService executor = Executors.newFixedThreadPool(3);
Future<Integer> future = executor.submit(() -> {
    // Task execution logic
    return 42;
});
```

newFixedThreadPool(3)方法调用创建一个包含3个线程的线程池，确保不会同时执行超过3个任务。然后使用submit()方法在线程池中提交任务执行，并返回表示计算结果的[Future](https://www.baeldung.com/java-future)对象。

### 4.2 CompletableFuture

相比之下，CompletableFuture为编写异步操作提供了更高级别的抽象，**它专注于定义工作流和处理异步任务的最终结果**。

下面是一个使用[supplyAsync()](https://www.baeldung.com/java-completablefuture-runasync-supplyasync)启动返回字符串的异步任务的示例：

```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
    return 42;
});
```

在此示例中，supplyAsync()启动一个异步任务，返回结果42。

## 5. 链接异步任务

ExecutorService和CompletableFuture都提供了链接异步任务的机制，但它们采用不同的方法。

### 5.1 ExecutorService

在ExecutorService中，我们通常提交要执行的任务，然后使用这些任务返回的Future对象来处理依赖项并链接后续任务。然而，**这涉及阻塞并等待每个任务完成后再继续执行下一个任务，这可能导致处理异步工作流的效率低下**。

考虑一下我们向ExecutorService提交两个任务，然后使用Future对象将它们链接在一起的情况：

```java
ExecutorService executor = Executors.newFixedThreadPool(2);

Future<Integer> firstTask = executor.submit(() -> {
    return 42;
});

Future<String> secondTask = executor.submit(() -> {
    try {
        Integer result = firstTask.get();
        return "Result based on Task 1: " + result;
    } catch (InterruptedException | ExecutionException e) {
        // Handle exception
    }
    return null;
});

executor.shutdown();
```

在此示例中，第二个任务依赖于第一个任务的结果。但是，ExecutorService不提供内置链接，因此我们需要明确管理依赖关系，方法是使用get()等待第一个任务完成(这会阻塞线程)，然后再提交第二个任务。

### 5.2 CompletableFuture

**另一方面，CompletableFuture提供了一种更精简、更具表现力的方式来链接异步任务**。它使用[thenApply()](https://www.baeldung.com/java-completablefuture-thenapply-thenapplyasync)等内置方法简化了任务链接，这些方法允许你定义一系列异步任务，其中一个任务的输出成为下一个任务的输入。

以下是使用CompletableFuture的等效示例：

```java
CompletableFuture<Integer> firstTask = CompletableFuture.supplyAsync(() -> {
    return 42;
});

CompletableFuture<String> secondTask = firstTask.thenApply(result -> {
    return "Result based on Task 1: " + result;
});
```

在此示例中，thenApply()方法用于定义第二个任务，该任务取决于第一个任务的结果。当我们使用thenApply()将任务链接到CompletableFuture时，主线程不会等待第一个任务完成后再继续执行，它继续执行代码的其他部分。

## 6. 错误处理

在本节中，我们将研究ExecutorService和CompletableFuture如何管理错误和异常情况。

### 6.1 ExecutorService

使用ExecutorService时，错误可能以两种方式表现出来：

- 提交的任务中抛出的异常：**当我们尝试使用get()等方法对返回的Future对象检索结果时，这些异常会传播回主线程**。如果处理不当，这可能会导致意外行为。
- 线程池管理期间的非受检异常：如果在线程池创建或关闭期间发生非受检异常，则通常是从ExecutorService方法本身抛出的。我们需要在代码中捕获并处理这些异常。

让我们看一个例子，强调一下潜在的问题：

```java
ExecutorService executor = Executors.newFixedThreadPool(2);

Future<String> future = executor.submit(() -> {
    if (someCondition) {
        throw new RuntimeException("Something went wrong!");
    }
    return "Success";
});

try {
    String result = future.get();
    System.out.println("Result: " + result);
} catch (InterruptedException | ExecutionException e) {
    // Handle exception
} finally {
    executor.shutdown();
}
```

在此示例中，如果满足特定条件，则提交的任务会抛出异常。但是，我们需要在future.get()周围使用try-catch块来捕获任务抛出的异常或使用get()检索期间抛出的异常，这种方法在管理多个任务中的错误时可能会变得繁琐。

### 6.2 CompletableFuture

相比之下，CompletableFuture提供了一种更强大的错误处理方法，它使用诸如exceptionally()之类的方法，并在链式方法本身中处理异常。**这些方法允许我们定义如何在异步工作流的不同阶段处理错误，而无需显式的try-catch块**。

下面是使用CompletableFuture进行错误处理的等效示例：

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    if (someCondition) {
        throw new RuntimeException("Something went wrong!");
    }
    return "Success";
})
.exceptionally(ex -> {
    System.err.println("Error in task: " + ex.getMessage());
    return "Error occurred"; // Can optionally return a default value
});

future.thenAccept(result -> System.out.println("Result: " + result));
```

在此示例中，异步任务抛出异常，错误在exceptionally()回调中被捕获和处理。如果出现异常，它会提供默认值(“Error occurred”)。

## 7. 超时管理

超时管理在异步编程中至关重要，以确保任务在指定的时间内完成。让我们探索一下ExecutorService和CompletableFuture如何以不同的方式处理超时。

### 7.1 ExecutorService

ExecutorService不提供内置超时功能，要实现超时，我们需要使用Future对象并可能中断超过时间的任务。此方法需要手动协调：

```java
ExecutorService executor = Executors.newFixedThreadPool(2);

Future<String> future = executor.submit(() -> {
    try {
        Thread.sleep(5000);
    } catch (InterruptedException e) {
        System.err.println("Error occured: " + e.getMessage());
    }
    return "Task completed";
});

try {
    String result = future.get(2, TimeUnit.SECONDS);
    System.out.println("Result: " + result);
} catch (TimeoutException e) {
    System.err.println("Task execution timed out!");
    future.cancel(true); // Manually interrupt the task.
} catch (Exception e) {
    // Handle exception
} finally {
    executor.shutdown();
}
```

在此示例中，我们向ExecutorService提交任务，并在使用get()方法检索结果时指定两秒的超时时间。如果任务完成时间超过指定的超时时间，则会抛出TimeoutException。这种方法容易出错，需要小心处理。

**需要注意的是，虽然超时机制会中断等待任务结果的过程，但任务本身仍会在后台继续运行，直到完成或被中断**。例如，要中断ExecutorService中正在运行的任务，我们需要使用Future.cancel(true)方法。

### 7.2 CompletableFuture

**在Java 9中，CompletableFuture使用[completeOnTimeout()](https://www.baeldung.com/java-completablefuture-timeout)等方法提供了更简化的超时方法**。如果原始任务未在指定的超时期限内完成，则completeOnTimeout()方法将使用指定的值完成CompletableFuture。

让我们看一个例子来说明completeOnTimeout()如何工作：

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    try {
        Thread.sleep(5000);
    } catch (InterruptedException e) {
        // Handle exception
    }
    return "Task completed";
});

CompletableFuture<String> timeoutFuture = future.completeOnTimeout("Timed out!", 2, TimeUnit.SECONDS);

String result = timeoutFuture.join();
System.out.println("Result: " + result);
```

在此示例中，supplyAsync()方法启动一个异步任务，模拟一个长时间运行的操作，需要5秒钟才能完成。但是，我们使用completeOnTimeout()指定2秒的超时时间。如果任务在2秒内未完成，CompletableFuture将自动完成，并显示“Timed out!”值。

## 8. 对比

下面是比较表，总结了ExecutorService和CompletableFuture之间的主要区别：

|     特征     |         ExecutorService          |                 CompletableFuture                 |
| :----------: |:--------------------------------:| :-------------------------------------------------: |
|     重点     |            线程池管理和任务执行            |             编写异步操作并处理最终结果              |
|     链式     |          与Future对象的手动协调          |             内置方法，例如thenApply()             |
|   错误处理   |   Future.get()周围手动包括try-catch块   | exceptionally()，whenComplete()，链接方法中的处理 |
|   超时管理   | 与Future.get(timeout)的手动协调以及潜在的中断 |         内置方法，例如completeOnTimeout()         |
| 阻塞与非阻塞 |    阻塞(通常等待Future.get()来检索结果)     |        非阻塞(链式执行任务而不阻塞主线程)         |

## 9. 总结

在本文中，我们探讨了处理异步任务的两个基本类：ExecutorService和CompletableFuture。ExecutorService简化了线程池和并发任务执行的管理，而CompletableFuture为组合异步操作和处理其结果提供了更高级别的抽象。

我们还研究了它们的功能、差异以及它们如何处理错误处理、超时管理和异步任务链接。