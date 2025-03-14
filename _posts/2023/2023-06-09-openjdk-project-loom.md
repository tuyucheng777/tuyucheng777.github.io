---
layout: post
title:  OpenJDK Project Loom
category: java-new
copyright: java-new
excerpt: Java 19
---

## 1. 概述

在本教程中，我们将快速了解[Project Loom](https://openjdk.org/projects/loom/)。本质上，**Project Loom的主要目标是探索、孵化和交付基于其构建的Java VM功能和API，以支持Java平台上易于使用、高吞吐量的轻量级并发和新的编程模型**。

## 2. Project Loom

Project Loom 是OpenJDK社区为Java引入轻量级并发构造的一次尝试。一开始，Loom项目设想通过整个JDK中的新特性、API和优化来广泛引入轻量级并发。Loom的原型在JVM和Java库中引入了变化。Project Loom的主要特性是[虚拟线程](https://www.baeldung.com/java-virtual-thread-vs-thread)，它已经实现了。

在讨论Loom的各种概念之前，让我们先讨论一下Java中当前的并发模型。

## 3. Java的并发模型

目前，Thread代表了Java中并发的核心抽象，此抽象和其他[并发API](https://www.baeldung.com/java-util-concurrent)使编写并发应用程序变得容易。具体来说，我们使用Thread创建平台线程，这些线程通常以1:1映射到操作系统内核线程。操作系统为平台线程分配大量堆栈和其他资源；但是，这些资源是有限的。尽管如此，我们仍使用平台线程来执行所有类型的任务。

但是由于Java采用的是[OS内核](https://www.baeldung.com/cs/os-kernel)线程来实现，已经不能满足现在的并发需求，具体来说主要有两点：

1. 线程数无法与域并发单元的规模相匹配。例如，应用程序通常允许多达数百万个事务、用户或会话。但是，内核支持的线程数要少得多。因此，**为每个用户、事务或会话设置一个线程通常是不可行的**。
2. 大多数并发应用程序需要对每个请求进行线程间同步。因此，**OS线程间会发生昂贵的上下文切换**。

**解决此类问题的一个可能方法是使用异步并发API**，常见的例子是[CompletableFuture](https://www.baeldung.com/java-completablefuture)和[RxJava](https://www.baeldung.com/rx-java)。只要这些API不阻塞内核线程，它就会为应用程序提供基于Java线程的更细粒度的并发构造。

另一方面，此类API更难调试，**也更难与旧版API集成**。因此，需要一个独立于内核线程的轻量级并发构造。

## 4. 任务和调度程序

任何线程的实现，无论是轻量级还是重量级，都依赖于两个构造：

1. 任务(也称为Continuation)-可以暂停自身以执行某些阻塞操作的一系列指令
2. 调度程序-用于将Continuation任务分配给CPU，以及从暂停的Continuation任务重新分配CPU

目前，**Java依赖于操作系统实现Continuation和调度程序**。

现在，为了暂停Continuation，需要存储整个调用堆栈。同样，在恢复时检索调用堆栈。**由于Continuation的操作系统实现包括本机调用堆栈以及Java的调用堆栈，因此占用空间很大**。

不过，更大的问题是使用OS调度程序。由于调度程序在内核模式下运行，因此线程之间没有区别。而且它以相同的方式处理每个CPU请求。

**这种类型的调度对于Java应用程序来说尤其不是最佳的**。

例如，假设一个应用程序线程对请求执行某些操作，然后将数据传递给另一个线程进行进一步处理。此时，**最好将这两个线程安排在同一个CPU上**。但是，由于调度程序与请求CPU的线程无关，因此无法保证这一点。

**Project Loom提出通过用户模式线程来解决这个问题，这些线程依赖于Java运行时对Continuation和调度程序的实现，而不是操作系统实现**。

## 5. 虚拟线程

OpenJDK 21引入了虚拟线程，并提供了在现有API(Thread和ThreadFactory)中创建它们的规定。

### 5.1 虚拟线程有何不同？

平台线程和虚拟线程的不同之处在于后者通常是用户模式线程，还有其他区别：

- **调度**-虚拟线程**由Java运行时而不是操作系统调度**
- **用户模式**-**虚拟线程将任何任务包装在内部用户模式Continuation中**，这允许在Java运行时而不是内核中暂停和恢复任务
- **命名**-虚拟线程不需要，或者默认具有线程名称；但是，我们可以设置一个名称
- **线程优先级**-虚拟线程具有固定的线程优先级，我们无法更改
- **守护线程**-虚拟线程是守护线程；因此，它们不会阻止关闭序列

### 5.2 虚拟线程的优点/缺点是什么？

虚拟线程有其优点和缺点：

|                 优点                 |                            缺点                            |
|:----------------------------------:|:--------------------------------------------------------:|
|             虚拟线程是轻量级的              |                  作为轻量级线程，它们不适合CPU密集型任务                   |
|             用户可以创建虚拟线程             | 许多虚拟线程共享同一个操作系统线程虚拟线程在涉及同步方法和语句的构造中阻塞，因为虚拟线程被固定到其底层平台线程。 |
|        当需要时，我们可以很容易地创建虚拟线程         |   Project Loom开发人员必须修改JDK中使用线程的每个API，以便可以无缝地与虚拟线程一起使用。   |
| 虚拟线程通常需要更少的资源。例如，单个JVM可以支持数百万个虚拟线程 |       如果一百万个虚拟线程中每个线程都有其线程局部变量的副本，则线程局部变量将需要更多的内存        |

### 5.3 何时使用虚拟线程？

当我们想要执行大部分时间处于阻塞状态的任务时，我们可以使用虚拟线程。对于大部分时间都在等待I/O操作完成的任务，我们使用轻量级、用户模式虚拟线程代替平台线程。

但是，我们不应该使用虚拟线程来执行长时间运行的CPU密集型操作。

### 5.4 如何创建虚拟线程？

我们有两种创建虚拟线程的主要选项：Thread类添加了一个名为ofVirtual的新类方法，该方法返回用于创建虚拟线程的构建器或用于创建虚拟线程的ThreadFactory。

因此，我们可以启动一个虚拟线程来运行任务：

```java
Thread thread = Thread.ofVirtual().start(Runnable task);
```

或者，我们可以使用等效形式创建一个虚拟线程来执行任务并安排它运行：

```java
Thread thread = Thread.startVirtualThread(Runnable task);
```

此外，我们可以使用创建虚拟线程的ThreadFactory：

```java
ThreadFactory factory = Thread.ofVirtual().factory();
Thread thread = factory.newThread(Runnable task);
```

我们可以使用isVirtual()方法来检查一个线程是否是虚拟线程：

```java
boolean isThreadVirtual = thread.isVirtual();
```

如果此方法返回true，则线程是虚拟拟线程。

### 5.5 虚拟线程是如何实现的？

虚拟线程是使用一小组底层平台线程(称为载体线程)实现的，诸如I/O操作之类的操作可以将载体线程从一个虚拟线程重新安排到另一个虚拟线程。但是，在虚拟线程中运行的代码并不知道底层平台线程。因此，currentThread()方法返回的是虚拟线程的Thread对象，而不是底层平台线程的Thread对象。

让我们看一下针对轻量级并发的其他一些优化。

## 6. 分隔Continuations

Continuations(或Coroutine)是按顺序执行的一系列指令，这些指令可以在稍后的阶段由调用者放弃并恢复。

每个Continuation都有一个入口点和一个让出点，让出点是它被暂停的地方。每当调用者恢复Continuation时，控制权就会返回到最后一个让出点。

重要的是要意识到，**这种暂停/恢复现在发生在语言运行时而不是操作系统中**。因此，它可以防止内核线程之间昂贵的上下文切换。

添加分隔Continuation是为了支持虚拟线程；因此，它们不需要作为公共API公开。让我们通过伪代码示例讨论分隔Continuation，它本质上是一个具有入口点的顺序子程序。我们可以在main()中创建一个Continuation，入口点为one()。

随后，我们可以调用Continuation，将控制权传递给入口点。one()可能会调用其他子例程，例如two()。two()中的执行被暂停，它将控制权传递给Continuation之外，并且main()中对Continuation的第一次调用返回。

让我们在main()中调用Continuation来恢复，它将控制权传递给最后一个暂停点。所有这些都发生在同一个执行上下文中：

```java
one() {  
    ... 
    two()
    ...
}

two() {
    ...
    suspend //  suspension point
    ... // resume point
}

main() {
    c = continuation(one) // create continuation  
    c.continue() // invoke continuation 
    c.continue() // invoke continuation again to resume
}
```

对于堆栈式Continuation(例如我们讨论的Continuation)，JVM需要捕获、存储和恢复调用堆栈，而不是将其作为内核线程的一部分。为了使JVM能够操作调用堆栈，unwind-and-invoke(UAI)是该项目的目标。UAI允许将堆栈展开到某个点，然后使用给定的参数调用方法。

## 7. 虚拟线程中的ForkJoinPool和自定义调度程序支持

前面我们讨论了OS调度程序在同一CPU上调度相关线程的缺点。

尽管Project Loom的目标是允许使用具有虚拟线程的可插拔调度程序，但**异步模式下的[ForkJoinPool](https://www.baeldung.com/java-fork-join)将用作默认调度程序**。OpenJDK 19为ForkJoinPool类添加了几个新的增强功能，包括setParallelism(int size)来设置目标并行度，从而控制工作线程的未来创建、使用和终止。

ForkJoinPool采用工作窃取算法，因此，每个线程都维护一个任务队列并从其头部执行任务。此外，**任何空闲线程都不会阻塞，等待任务，而是从另一个线程的双端队列尾部提取任务**。

异步模式下的唯一区别是**工作线程从另一个双端队列的头部窃取任务**。

**ForkJoinPool将另一个正在运行的任务调度的任务添加到本地队列，因此，在同一CPU上执行它**。

## 8. 结构化并发

OpenJDK引入了结构化并发的预览功能，该功能属于Loom项目的范围。结构化并发的目标是将在不同线程中运行的相关任务组视为具有单一作用域的单个工作单元，它的好处是简化了错误处理和取消，从而提高了可靠性和可观察性。

为此，它引入了预览API java.util.concurrent.StructuredTaskScope，它将任务拆分为多个并发子任务。此外，主任务必须等待子任务完成。使用fork()方法，我们可以启动新线程来运行子任务，并使用join()方法等待线程完成。此API旨在在try-with-resources语句中使用，例如：

```java
Callable<String> task1 = ...
Callable<String> task2 = ...

try (var scope = new StructuredTaskScope<String>()) {

    Subtask<String> subtask1 = scope.fork(task1); //create thread to run first subtask
    Subtask<String> subtask2 = scope.fork(task2); //create thread to run second subtask

    scope.join(); //wait for subtasks to finish

    // process results of subtasks
}
```

之后，我们可以处理子任务的结果。

## 9. 总结

在本文中，我们讨论了Java当前并发模型中的问题以及[Project Loom](https://wiki.openjdk.org/display/loom/Main)提出的改变。

在此过程中，我们讨论了OpenJDK 21 中引入的轻量级虚拟线程如何使用内核线程为Java提供替代方案。