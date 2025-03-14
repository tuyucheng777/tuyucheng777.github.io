---
layout: post
title:  Java中线程和虚拟线程的区别
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 简介

在本教程中，我们将展示Java中的传统线程与[Project Loom](https://www.baeldung.com/openjdk-project-loom)中引入的虚拟线程之间的区别。

接下来，我们将分享虚拟线程的几个用例以及该项目引入的API。

## 2. 线程与虚拟线程的高层概述

从高层次上讲，**线程由操作系统管理和调度，而虚拟线程由虚拟机管理和调度。现在，要创建新的内核线程，我们必须执行系统调用，这是一项成本高昂的操作**。

这就是我们使用线程池而不是根据需要重新分配和释放线程的原因。接下来，如果我们想通过添加更多线程来扩展我们的应用程序，由于上下文切换及其内存占用，维护这些线程的成本可能会很高，并影响处理时间。

然后，通常我们不想阻塞这些线程，这会导致使用非阻塞I/O API和异步API，这可能会使我们的代码混乱。

相反，**虚拟线程由JVM管理**。因此，它们的分配不需要系统调用，**并且不受操作系统上下文切换的影响**。此外，虚拟线程在载体线程上运行，载体线程是实际在后台使用的内核线程。因此，由于我们不受系统上下文切换的影响，可以生成更多这样的虚拟线程。

接下来，虚拟线程的一个关键属性是它们不会阻塞我们的载体线程。因此，阻塞虚拟线程将成为一个更便宜的操作，因为JVM将调度另一个虚拟线程，而载体线程不会被阻塞。

最终，我们不需要使用NIO或异步PI。这样代码的可读性会更高，更容易理解和调试。不过，**Continuation可能会阻塞载体线程**-尤其是当线程调用原生方法并从那里执行阻塞操作时。

## 3. 新的线程生成器API

在Loom中，我们在Thread类中获得了新的构建器API，以及几个工厂方法。让我们看看如何创建标准和虚拟工厂并将它们用于我们的线程执行：

```java
Runnable printThread = () -> System.out.println(Thread.currentThread());
        
ThreadFactory virtualThreadFactory = Thread.builder().virtual().factory();
ThreadFactory kernelThreadFactory = Thread.builder().factory();

Thread virtualThread = virtualThreadFactory.newThread(printThread);
Thread kernelThread = kernelThreadFactory.newThread(printThread);

virtualThread.start();
kernelThread.start();
```

以下是上述运行的输出：

```text
Thread[Thread-0,5,main]
VirtualThread[<unnamed>,ForkJoinPool-1-worker-3,CarrierThreads]
```

这里，第一个条目是内核线程的标准toString输出。

现在，我们在输出中看到虚拟线程没有名称，并且它正在来自CarrierThreads线程组的Fork-Join池的工作线程上执行。

我们可以看到，**无论底层实现如何，API都是相同的，这意味着我们可以轻松地在虚拟线程上运行现有代码**。

此外，我们不需要学习新的API来使用它们。

## 4. 虚拟线程组成

它是**一个Continuation和一个调度程序**，一起构成了一个虚拟线程。现在，我们的用户态调度程序可以是Executor接口的任何实现。上面的示例向我们展示了，默认情况下，我们在ForkJoinPool上运行。

现在，类似于内核线程(可以在CPU上执行，然后暂停、重新安排，然后恢复执行)，Continuation是一个执行单元，可以启动，然后暂停(让出)、重新安排，然后以相同的方式从中断的地方恢复执行，并且仍然由JVM进行管理，而不必依赖操作系统。

请注意，Continuation是一个低级API，程序员应该使用更高级别的API(例如构建器API)来运行虚拟线程。

然而，为了展示它的内部工作原理，现在我们将继续进行实验：

```java
var scope = new ContinuationScope("C1");
var c = new Continuation(scope, () -> {
    System.out.println("Start C1");
    Continuation.yield(scope);
    System.out.println("End C1");
});

while (!c.isDone()) {
    System.out.println("Start run()");
    c.run();
    System.out.println("End run()");
}
```

以下是上述运行的输出：

```text
Start run()
Start C1
End run()
Start run()
End C1
End run()
```

在这个例子中，我们运行了Continuation，并在某个时候决定停止处理。然后，一旦我们重新运行它，Continuation就会从它停止的地方继续执行。通过输出，我们看到run()方法被调用了两次，但Continuation启动了一次，然后在第二次运行中从它停止的地方继续执行。

这就是JVM处理阻塞操作的方式，**一旦发生阻塞操作，Continuation将让出**，从而使载体线程保持畅通。

因此，发生的事情是，我们的主线程在其调用堆栈上为run()方法创建了一个新的栈帧，并继续执行。然后，在继续执行后，JVM保存了其执行的当前状态。

接下来，主线程继续执行，就像run()方法返回并继续执行while循环一样。在第二次调用Continuation的run方法后，JVM将主线程的状态恢复到Continuation已放弃并完成执行的点。

## 5. 总结

在本文中，我们讨论了内核线程和虚拟线程之间的区别。接下来，我们展示了如何使用Project Loom中的新线程构建器API来运行虚拟线程。

最后，我们展示了什么是Continuation以及它在底层是如何工作的。