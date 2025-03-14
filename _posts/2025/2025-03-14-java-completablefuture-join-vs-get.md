---
layout: post
title:  CompletableFuture join()与get()指南
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 简介

在Java的并发编程中，CompletableFuture是一个强大的工具，它允许我们编写非阻塞代码。使用CompletableFuture时，我们会遇到两种常用方法：join()和get()。这两种方法都用于在计算完成后检索计算结果，但它们有一些关键的区别。

在本教程中，我们将探讨这两种方法之间的区别。

## 2. CompletableFuture概述

在深入研究join()和get()之前，让我们简要回顾一下[CompletableFuture](https://www.baeldung.com/java-completablefuture)是什么。CompletableFuture表示异步计算的未来结果，与回调等传统方法相比，它提供了一种以更易读、更易于管理的方式编写异步代码的方法。让我们看一个例子来说明CompletableFuture的用法。

首先，让我们创建一个CompletableFuture：

```java
CompletableFuture<String> future = new CompletableFuture<>();
```

接下来，让我们用一个值来完成Future：

```java
future.complete("Hello, World!");
```

最后，我们使用join()或get()检索值：

```java
String result = future.join(); // or future.get();
System.out.println(result); // Output: Hello, World!
```

## 3. join()方法

join()方法是检索CompletableFuture结果的直接方法，它等待计算完成，然后返回结果。如果计算遇到异常，join()将抛出非受检异常，具体来说是CompletionException。

以下是join()的语法：

```java
public T join()
```

我们来回顾一下join()方法的特点：

- 计算完成后返回结果
- 如果完成CompletableFuture所涉及的任何计算导致异常，则引发非受检的异常-CompletionException
- **由于CompletionException是非受检异常，因此不需要在方法签名中显式处理或声明**

## 4. get()方法

另一方面，get()方法检索计算结果，如果计算遇到错误，则抛出受检异常。get()方法有两种变体：一种是无限期等待，另一种是等待指定的超时时间。

让我们回顾一下get()的两种变体的语法：

```java
public T get() throws InterruptedException, ExecutionException
public T get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException
```

我们再看看get()方法的特点：

- 计算完成后返回结果
- 引发受检的异常，可能是InterruptedException、ExecutionException或TimeoutException
- **需要在方法签名中明确处理或声明受检异常**

get()方法继承自[Future](https://www.baeldung.com/java-future)接口，CompletableFuture实现了该接口。Future接口是在Java 5中引入的，表示异步计算的结果，它定义了get()方法来检索结果并处理计算过程中可能发生的异常。

当Java 8引入CompletableFuture时，它被设计为与现有的Future接口兼容，以确保与现有代码库的向后兼容性，因此需要在CompletableFuture中包含get()方法。

## 5. 比较：join()与get()

让我们总结一下join()和get()之间的主要区别：

| 方面 | join()                     | get()                                                          |
| -------- |----------------------------|----------------------------------------------------------------|
| 异常类型 | 抛出CompletionException(非受检) | 抛出InterruptedException、ExecutionException和TimeoutException(受检) |
| 异常处理 | 非受检，无需声明或捕获                | 受检，必须声明，否则将被捕获                                                 |
| 超时支持 | 不支持超时                      | 支持超时                                                           |
| 起源     | 特定于CompetableFuture        | 继承自Future接口                                                    |
| 使用建议 | 新代码的首选                     | 为了兼容旧版本                                                        |

## 6. 测试

让我们添加一些测试来确保我们对join()和get()的理解是正确的：

```java
@Test
public void givenJoinMethod_whenThrow_thenGetUncheckedException() {
    CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> "Test join");

    assertEquals("Test join", future.join());

    CompletableFuture<String> exceptionFuture = CompletableFuture.failedFuture(new RuntimeException("Test join exception"));

    assertThrows(CompletionException.class, exceptionFuture::join);
}

@Test
public void givenGetMethod_whenThrow_thenGetCheckedException() {
    CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> "Test get");

    try {
        assertEquals("Test get", future.get());
    } catch (InterruptedException | ExecutionException e) {
        fail("Exception should not be thrown");
    }

    CompletableFuture<String> exceptionFuture = CompletableFuture.failedFuture(new RuntimeException("Test get exception"));

    assertThrows(ExecutionException.class, exceptionFuture::get);
}
```

## 7. 总结

在这篇简短的文章中，我们了解到join()和get()都是用于检索CompletableFuture结果的方法，但它们处理异常的方式不同。join()方法会抛出非受检的异常，因此当我们不想明确处理异常时，它更容易使用。另一方面，get()方法会抛出受检的异常，提供更详细的异常处理和超时支持。通常，对于新代码，应该优先使用join()，因为它很简单，而get()仍然可用，以实现旧版兼容性。