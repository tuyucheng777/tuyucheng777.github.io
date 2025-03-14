---
layout: post
title:  如何在ScheduledExecutorService中使用虚拟线程
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 简介

虚拟线程是JDK 21中官方引入的一个有用功能，作为提高高吞吐量应用程序性能的解决方案。

**但是，JDK没有内置使用虚拟线程的任务调度工具。因此，我们必须编写使用虚拟线程运行的任务调度程序**。

在本文中，我们将使用[Thread.sleep()](https://www.baeldung.com/java-delay-code-execution)方法和[ScheduledExecutorService](https://www.baeldung.com/java-executor-service-tutorial#bd-ScheduledExecutorService)类为虚拟线程创建自定义调度程序。

## 2. 什么是虚拟线程？

**虚拟线程在[JEP-444](https://openjdk.org/jeps/444)中引入，作为Thread类的轻量级版本，最终提高了高吞吐量应用程序的并发性**。

虚拟线程比通常的操作系统线程(或平台线程)占用的空间要少得多，因此，我们可以在应用程序中同时生成比平台线程更多的虚拟线程。毫无疑问，这会增加最大并发单元数，从而也增加了应用程序的吞吐量。

一个关键点是虚拟线程并不比平台线程快，在我们的应用程序中，它们的数量比平台线程多，只是为了执行更多的并行工作。

虚拟线程成本低廉，因此我们不需要使用资源池之类的技术将任务调度到有限数量的线程。相反，我们可以在现代计算机中几乎无限地生成它们，而不会出现内存问题。

最后，虚拟线程是动态的，而平台线程的大小是固定的。因此，虚拟线程比平台线程更适合执行简单的HTTP或数据库调用等小任务。

## 3. 调度虚拟线程

我们已经看到虚拟线程的一大优势是它们体积小且成本低，我们可以在一台简单的机器上有效地生成数十万个虚拟线程，而不会陷入内存不足的错误。因此，像我们处理平台线程和网络或数据库连接等更昂贵的资源那样将虚拟线程池化并没有多大意义。

通过保留线程池，我们又增加了池中可用线程的任务池化开销，这更加复杂，并且可能更慢。此外，Java中的大多数线程池都受到平台线程数量的限制，该数量始终小于程序中可能的虚拟线程数量。

**因此，我们必须避免将虚拟线程与ForkJoinPool或ThreadPoolExecutor等线程池API一起使用。相反，我们应该始终为每个任务创建一个新的虚拟线程**。

目前，Java不提供标准API来调度虚拟线程，像ScheduledExecutorService的schedule()方法等其他并发API提供的那样。因此，为了有效地让虚拟线程运行计划任务，我们需要编写自己的调度程序。

### 3.1 使用Thread.sleep()调度虚拟线程

我们将看到的创建自定义调度程序的最直接的方法是使用Thread.sleep()方法让程序等待当前线程执行：

```java
static Future<?> schedule(Runnable task, int delay, TemporalUnit unit, ExecutorService executorService) {
    return executorService.submit(() -> {
        try {
            Thread.sleep(Duration.of(delay, unit));
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }

        task.run();
    });
}
```

schedule()方法接收要安排的任务、延迟和ExecutorService。然后，我们使用ExecutorService的submit()启动任务，在try块中，我们通过调用Thread.sleep()使将执行任务的线程等待所需的延迟。因此，线程可能会在等待时中断，因此我们通过中断当前线程执行来[处理InterruptedException](https://www.baeldung.com/java-interrupted-exception#how-to-handle-an-interruptedexception)。

最后，等待之后，我们使用收到的任务调用run()。

为了使用自定义的schedule()方法调度虚拟线程，我们需要将虚拟线程的执行器服务传递给它：

```java
ExecutorService virtualThreadExecutor = Executors.newVirtualThreadPerTaskExecutor();

try (virtualThreadExecutor) {
    var taskResult = schedule(() -> 
        System.out.println("Running on a scheduled virtual thread!"), 5, ChronoUnit.SECONDS,
        virtualThreadExecutor);

    try {
        Thread.sleep(10 * 1000); // Sleep for 10 seconds to wait task results
    } catch (InterruptedException e) {
        Thread.currentThread()
            .interrupt();
    }

    System.out.println(taskResult.get());
}
```

**首先，我们实例化一个ExecutorService，它会为我们提交的每个任务生成一个新的虚拟线程**。然后，我们将virtualThreadExecutor变量包装在[try-with-resources](https://www.baeldung.com/java-try-with-resources)语句中，使ExecutorService保持打开状态，直到我们完成使用它为止。或者，在使用ExecutorService后，我们可以使用[shutdown()](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ExecutorService.html#shutdown())正确完成它。

我们调用schedule()在5秒后运行任务，然后等待10秒再尝试获取任务执行结果。

### 3.2 使用SingleThreadExecutor调度虚拟线程

我们了解了如何使用sleep()将任务调度到虚拟线程，或者，我们可以在虚拟线程执行器中为每个提交的任务实例化一个新的单线程调度程序：

```java
static Future<?> schedule(Runnable task, int delay, TimeUnit unit, ExecutorService executorService) {
    return executorService.submit(() -> {
        ScheduledExecutorService singleThreadScheduler = Executors.newSingleThreadScheduledExecutor();

        try (singleThreadScheduler) {
            singleThreadScheduler.schedule(task, delay, unit);
        }
    });
}
```

**代码还使用作为参数传递的虚拟线程ExecutorService来提交任务。但现在，对于每个任务，我们使用newSingleThreadScheduledExecutor()方法实例化单个线程的单个ScheduledExecutorService**。

然后，在try-with-resources块中，我们使用单线程执行器schedule()方法调度任务，该方法接收task和delay作为参数，并且不会像sleep()那样抛出受检的InterruptedException。

最后，我们可以使用schedule()将任务安排到虚拟线程执行器：

```java
ExecutorService virtualThreadExecutor = Executors.newVirtualThreadPerTaskExecutor();

try (virtualThreadExecutor) {
    var taskResult = schedule(() -> 
        System.out.println("Running on a scheduled virtual thread!"), 5, TimeUnit.SECONDS,
        virtualThreadExecutor);

    try {
        Thread.sleep(10 * 1000); // Sleep for 10 seconds to wait task results
    } catch (InterruptedException e) {
        Thread.currentThread()
            .interrupt();
    }

    System.out.println(taskResult.get());
}
```

这与3.1节的schedule()方法的用法类似，但在这里，我们传递的是TimeUnit而不是ChronoUnit。

### 3.3 使用sleep()调度任务与使用单线程执行器

在sleep()调度方法中，我们只是调用一个方法来等待，然后才能有效地运行任务。因此，很容易理解代码在做什么，也更容易调试。另一方面，每个任务使用一个调度的执行程序服务取决于库的调度程序代码，因此调试或故障排除可能更困难。

**此外，如果我们选择使用sleep()，我们只能安排任务在固定延迟后运行。相比之下，使用ScheduledExecutorService，我们可以访问三种调度方法：schedule()、scheduleAtFixedRate()和scheduleWithFixedDelay()**。

ScheduledExecutorService的schedule()方法添加了延迟，就像sleep()一样。scheduleAtFixedRate()和scheduleWithFixedDelay()方法为调度添加了周期性，因此我们可以在固定大小的时间段内重复执行任务。因此，使用ScheduledExecutorService内置Java库来调度任务时，我们可以更加灵活。

## 4. 总结

在本文中，我们介绍了使用虚拟线程相对于传统平台线程的一些优势。然后，我们研究了如何使用Thread.sleep()和ScheduledExecutorService来安排任务在虚拟线程中运行。