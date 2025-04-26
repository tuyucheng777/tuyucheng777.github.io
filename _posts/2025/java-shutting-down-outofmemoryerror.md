---
layout: post
title:  Java中发生OutOfMemoryError时关闭
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

维护应用程序的一致状态比保持其运行更重要，在大多数情况下都是如此。

在本教程中，我们将学习如何在发生OutOfMemoryError时明确停止应用程序。在某些情况下，如果没有正确的处理，我们可能会在错误的状态下继续运行应用程序。

## 2. OutOfMemoryError

OutOfMemoryError错误发生在应用程序外部，并且至少在大多数情况下是不可恢复的。该错误的名称暗示应用程序没有足够的RAM，但这并非完全正确。**更准确地说，应用程序[无法分配](https://www.baeldung.com/java-memory-leaks)所请求的内存量**。

在单线程应用程序中，情况非常简单，**如果我们遵循[指南](https://www.baeldung.com/java-errors-vs-exceptions#error)并且没有捕获OutOfMemoryError，应用程序就会终止**，这是处理此错误的预期方式。

在某些[特定情况](https://www.baeldung.com/java-jvm-out-of-memory-during-runtime#root-cause)下，捕获OutOfMemoryError是合理的。此外，我们还可以处理一些更具体的情况，在这些情况下，捕获OutOfMemoryError之后继续执行也是合理的。**但是，在大多数情况下，OutOfMemoryError意味着应用程序应该停止**。

## 3. 多线程

[多线程](https://www.baeldung.com/java-concurrency)是大多数现代应用程序不可或缺的一部分，**线程在异常处理方面遵循拉斯维加斯规则：线程中发生的事情，就留在线程中**，这并非总是如此，但我们可以将其视为一种普遍行为。

**因此，即使线程中最严重的错误也不会传播到主应用程序，除非我们明确处理它们**。让我们考虑以下内存泄漏的示例：

```java
public static final Runnable MEMORY_LEAK = () -> {
    List<byte[]> list = new ArrayList<>();
    while (true) {
        list.add(tenMegabytes());
    }
};

private static byte[] tenMegabytes() {
    return new byte[1024 * 1014 * 10];
}
```

如果我们在单独的线程中运行此代码，应用程序就不会失败：

```java
@Test
void givenMemoryLeakCode_whenRunInsideThread_thenMainAppDoestFail() throws InterruptedException {
    Thread memoryLeakThread = new Thread(MEMORY_LEAK);
    memoryLeakThread.start();
    memoryLeakThread.join();
}
```

发生这种情况是因为所有导致OutOfMemoryError的数据都与线程相关，当线程死亡时，[List](https://www.baeldung.com/java-arraylist)失去了其[垃圾回收根](https://www.baeldung.com/java-gc-roots)，可以被回收。**因此，最初导致OutOfMemoryError的数据会随着线程的死亡而被移除**。

如果我们多次运行此代码，应用程序就不会失败：

```java
@Test
void givenMemoryLeakCode_whenRunSeveralTimesInsideThread_thenMainAppDoestFail() throws InterruptedException {
    for (int i = 0; i < 5; i++) {
        Thread memoryLeakThread = new Thread(MEMORY_LEAK);
        memoryLeakThread.start();
        memoryLeakThread.join();
    }
}
```

同时垃圾回收[日志](https://www.baeldung.com/java-verbose-gc)显示如下情况：

![](/assets/images/2025/javajvm/javashuttingdownoutofmemoryerror01.png)

**在每个循环中，我们消耗6GB的可用RAM，终止线程，运行垃圾回收，移除数据，然后继续执行**。我们得到了这种堆过山车式的运行，虽然它没有做任何合理的工作，但应用程序不会失败。

同时，我们可以在日志中看到错误，在某些情况下，忽略OutOfMemoryError是合理的，我们不想因为一个bug或用户漏洞而导致整个Web服务器瘫痪。

此外，实际应用中的行为可能有所不同。线程之间可能存在互连，并且可能存在其他共享资源。因此，任何线程都可能抛出OutOfMemoryError异常，这是一个异步异常；它们并不绑定到特定的线程。**但是，如果主应用程序线程中没有发生OutOfMemoryError异常，应用程序仍将运行**。

## 4. 终止JVM

在某些应用程序中，线程执行着至关重要的工作，应该可靠地完成，最好停止所有进程，调查并解决问题。

想象一下，我们正在处理一个包含历史银行数据的大型XML文件，我们将数据块加载到内存中，进行计算，然后将结果写入磁盘。**这个例子可能更复杂，但主要思想是，有时我们严重依赖线程中进程的事务性和正确性**。

幸运的是，[JVM](https://www.baeldung.com/jvm-vs-jre-vs-jdk#jvm)将OutOfMemoryError视为一种特殊情况，我们可以使用以下参数在应用程序中出现OutOfMemoryError时退出或使JVM崩溃：

```text
-XX:+ExitOnOutOfMemoryError
-XX:+CrashOnOutOfMemoryError
```

如果我们使用任何这些参数运行示例，应用程序都会停止，这将使我们能够调查问题并检查发生了什么。

**这些选项之间的区别在于-XX:+CrashOnOutOfMemoryError会产生崩溃转储**：

```text
#
# A fatal error has been detected by the Java Runtime Environment:
#
#  Internal Error (debug.cpp:368), pid=69477, tid=39939
#  fatal error: OutOfMemory encountered: Java heap space
#
...
```

它包含我们可以用来分析的信息，为了简化这个过程，我们还可以进行堆转储以进一步调查，有一个特殊选项可以在发生OutOfMemoryError时[自动](https://www.baeldung.com/java-heap-dump-capture#automatically)执行此操作。

我们还可以为多线程应用程序创建线程转储，它没有专用参数，但是，我们可以使用[脚本](https://www.baeldung.com/jvm-parameters#handling-out-of-memory)并通过OutOfMemoryError触发它。

**如果我们想以类似的方式处理其他异常，则必须使用[Future](https://www.baeldung.com/java-future)来确保线程按预期完成其工作，将异常包装到OutOfMemoryError中以避免实现正确的线程间通信是一个糟糕的主意**：

```java
@Test
void givenBadExample_whenUseItInProductionCode_thenQuestionedByEmployerAndProbablyFired() throws InterruptedException {
    Thread npeThread = new Thread(() -> {
        String nullString = null;
        try {
            nullString.isEmpty();
        } catch (NullPointerException e) {
            throw new OutOfMemoryError(e.getMessage());
        }
    });
    npeThread.start();
    npeThread.join();
}
```

## 5. 总结

在本文中，我们讨论了OutOfMemoryError经常导致应用程序处于错误状态的原因。虽然在某些情况下我们可以恢复，但总体而言，我们应该考虑终止并重新启动应用程序。

虽然单线程应用程序不需要对OutOfMemoryError进行任何额外的处理，但多线程代码需要额外的分析和配置，以确保应用程序能够退出或崩溃。