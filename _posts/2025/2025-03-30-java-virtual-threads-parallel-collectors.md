---
layout: post
title:  使用并行收集器和虚拟线程进行并行收集处理
category: libraries
copyright: libraries
excerpt: Parallel Collectors
---

## 1. 简介

在[上一篇文章](https://www.baeldung.com/java-parallel-collectors)中，我们介绍了[parallel-collectors](https://github.com/pivovarit/parallel-collectors)，这是一个小型零依赖库，可以在自定义线程池上实现Stream API的并行处理。

[Project Loom](https://www.baeldung.com/openjdk-project-loom)是将轻量级虚拟线程(以前称为Fibers)引入JVM的组织工作的代号，该工作已在JDK 21中完成。

让我们看看如何在并行收集器中利用这一点。

## 2. Maven依赖

如果我们想要开始使用该库，我们需要在Maven的pom.xml文件中添加一个条目：

```xml
<dependency>
    <groupId>com.pivovarit</groupId>
    <artifactId>parallel-collectors</artifactId>
    <version>3.0.0</version>
</dependency>
```

或者Gradle构建文件中的一行：

```groovy
compile 'com.pivovarit:parallel-collectors:3.0.0'
```

最新版本可以在[Maven Central](https://mvnrepository.com/artifact/com.pivovarit/parallel-collectors)上找到。

## 3. 使用操作系统线程与虚拟线程进行并行处理

### 3.1 操作系统线程并行性

让我们看看为什么使用虚拟线程进行并行处理是一个重点。

我们先创建一个简单的例子，我们需要一个并行操作，即人为延迟的字符串拼接：

```java
private static String fetchById(int id) {
    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        // ignore shamelessly
    }
    return "user-" + id;
}
```

我们还将使用自定义代码来测量执行时间：

```java
private static <T> T timed(Supplier<T> supplier) {
    var before = Instant.now();
    T result = supplier.get();
    var after = Instant.now();
    log.info("Execution time: {} ms", Duration.between(before, after).toMillis());
    return result;
}
```

现在，让我们创建一个简单的并行流处理示例，其中我们创建n个元素，然后在n个线程上以n的并行度处理它们：

```java
@Test
public void processInParallelOnOSThreads() {
    int parallelProcesses = 5_000;
    var e = Executors.newFixedThreadPool(parallelProcesses);

    var result = timed(() -> Stream.iterate(0, i -> i + 1).limit(parallelProcesses)
            .collect(ParallelCollectors.parallel(i -> fetchById(i), toList(), e, parallelProcesses))
            .join());

    log.info("{}", result);
}
```

当我们运行它时，我们可以观察到它显然完成了工作，因为我们不需要等待5000秒才能获得结果：

```text
Execution time: 1321 ms
[user-0, user-1, user-2, ...]
```

但是让我们看看如果我们尝试将并行处理的元素数量增加到20000会发生什么：

```text
[2.795s][warning][os,thread] Failed to start thread "Unknown thread" - pthread_create failed (...)
[2.795s][warning][os,thread] Failed to start the native thread for java.lang.Thread "pool-1-thread-16111"
```

**基于操作系统线程的方法无法扩展，因为创建线程的成本很高，而且我们很快就会达到资源限制**。

让我们看看如果切换到虚拟线程会发生什么。

### 3.2 虚拟线程并行

在Java 21之前，很难为线程池配置提供合理的默认值。幸运的是，虚拟线程不需要任何默认值-我们可以根据需要创建任意数量的线程，并且它们在共享的ForkJoinPool实例上进行内部调度，这使得它们非常适合运行阻塞操作。

如果我们运行的是[Parallel Collectors](https://www.baeldung.com/java-parallel-collectors) 3.x，我们可以毫不费力地利用虚拟线程：

```java
@Test
public void processInParallelOnVirtualThreads() {
    int parallelProcesses = 5_000;

    var result = timed(() -> Stream.iterate(0, i -> i + 1).limit(parallelProcesses)
            .collect(ParallelCollectors.parallel(i -> fetchById(i), toList()))
            .join());
}
```

我们可以看到，**这就像省略执行器和并行参数一样简单**，因为虚拟线程是默认的执行实用程序。

如果我们尝试运行它，我们可以看到它实际上比原始示例完成得更快：

```text
Execution time: 1101 ms
[user-0, user-1, user-2, ...]
```

这是因为我们创建了5000个虚拟线程，这些线程是使用一组非常有限的操作系统线程进行调度的。

让我们尝试将并行度增加到20000，这在传统的Executor中是不可能的：

```text
Execution time: 1219 ms
[user-0, user-1, user-2, ...]
```

这不仅成功执行，而且比OS线程上小4倍的作业完成得更快。

我们将并行度增加到100000并看看会发生什么：

```text
Execution time: 1587 ms
[user-0, user-1, user-2, ...]
```

尽管观察到相当大的开销，但工作正常。

如果我们将并行度提高到1000000会怎么样？

```text
Execution time: 6416 ms
[user-0, user-1, user-2, ...]
```

2000000？

```text
Execution time: 12906 ms
[user-0, user-1, user-2, ...]
```

5000000？

```text
Execution time: 25952 ms
[user-0, user-1, user-2, ...]
```

可以看到，**我们可以轻松扩展到使用OS线程无法实现的高水平并行性**。除了在较小的并行工作负载上的性能改进外，这也是利用虚拟线程并行处理阻塞操作的主要优势。

### 3.3 虚拟线程和旧版本的Parallel Collectors

利用虚拟线程的最简单方法是升级到库的最新版本，但如果这不可能，我们也可以在JDK 21上运行时使用2.xy版本来实现这一点。

诀窍是手动提供Executors.newVirtualThreadPerTaskExecutor()作为执行器，并提供Integer.MAX_VALUE作为最大并行级别：

```java
@Test
public void processInParallelOnVirtualThreadsParallelCollectors2() {
    int parallelProcesses = 100_000;

    var result = timed(() -> Stream.iterate(0, i -> i + 1).limit(parallelProcesses)
            .collect(ParallelCollectors.parallel(
                    i -> fetchById(i), toList(),
                    Executors.newVirtualThreadPerTaskExecutor(), Integer.MAX_VALUE))
            .join());

    log.info("{}", result);
}
```

## 4. 总结

在本文中，我们了解了如何轻松地利用Parallel Collectors库中的虚拟线程，事实证明，其扩展性比传统的基于操作系统线程的解决方案要好得多。我们的测试机器最终在大约16000个线程时达到资源限制，而轻松扩展到数百万个虚拟线程是可能的。