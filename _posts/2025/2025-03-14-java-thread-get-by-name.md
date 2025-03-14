---
layout: post
title:  在Java中通过名称获取线程
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 概述

线程是Java中[并发编程](https://www.baeldung.com/java-concurrency)的基本构建块，在许多应用程序中，我们可能需要通过线程名称来定位特定线程，以执行诸如调试、监控甚至与线程状态交互之类的操作。

在本教程中，我们将探讨如何在Java中通过名称检索线程。

## 2. 理解Java中的线程名称

每个线程都有一个唯一的[名称](https://www.baeldung.com/java-set-thread-name)，有助于在执行期间识别它。虽然[JVM](https://www.baeldung.com/jvm-series)会自动命名线程(例如Thread-0、Thread-1等)，但**我们可以为线程分配自定义名称**，以便更好地进行跟踪：

```java
Thread customThread = new Thread(() -> {
    log.info("Running custom thread");
}, "MyCustomThread");

customThread.start();
```

该线程的名称设置为MyCustomThread。

接下来，让我们探索一下通过线程名称获取线程的方法。

## 3. 使用Thread.getAllStackTraces()

**[Thread.getAllStackTraces()](https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/lang/Thread.html#getAllStackTraces())方法提供了所有活动线程及其相应堆栈跟踪的映射**，此映射允许我们循环遍历所有活动线程并按名称搜索特定线程。

下面展示了如何使用该方法：

```java
public static Thread getThreadByName(String name) {
    return Thread.getAllStackTraces()
        .keySet()
        .stream()
        .filter(thread -> thread.getName().equals(name))
        .findFirst()
        .orElse(null); // Return null if thread not found
}
```

让我们深入了解一下此方法的具体内容：

- Thread.getAllStackTraces()方法返回所有活动线程的Map<Thread, StackTraceElement[]\>。
- 我们使用stream()方法更轻松地使用Java的[Stream API](https://www.baeldung.com/java-streams)进行处理，然后过滤流以仅包含名称与输入匹配的线程。
- 如果找到匹配项，我们将返回该线程；否则，我们返回null。

让我们用单元测试来验证此方法：

```java
@Test
public void givenThreadName_whenUsingGetAllStackTraces_thenThreadFound() throws InterruptedException {
    Thread testThread = new Thread(() -> {
        try {
            Thread.sleep(1000); // Simulate some work
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }, "TestThread");
    testThread.start();

    Thread foundThread = ThreadFinder.getThreadByName("TestThread");
    assertNotNull(foundThread);
    assertEquals("TestThread", foundThread.getName());
    testThread.join(); // Ensure the thread finishes
}
```

我们来看看测试的重点：

- 线程创建：我们创建一个名为TestThread的线程，通过短暂的睡眠来模拟一些计算。
- 断言：我们检查使用每种方法是否找到具有给定名称的线程并验证其名称。
- 线程连接：最后，我们确保创建的线程以join()完成，以避免出现线程延迟。

## 4. 使用ThreadGroup

ThreadGroup类是另一种通过名称定位线程的方法，**[ThreadGroup](https://docs.oracle.com/en/java/javase/23/docs/api/java.base/java/lang/ThreadGroup.html)表示一组线程，允许我们将它们作为一个集体实体进行管理或检查**。通过查询特定线程组，我们可以通过名称定位线程。

有多种方法可以访问ThreadGroup：

- 通过Thread.currentThread().getThreadGroup()获取当前ThreadGroup
- 使用new ThreadGroup(name)明确创建一个新的ThreadGroup
- 通过找到父级引用导航到根组

以下是使用ThreadGroup的解决方案：

```java
public static Thread getThreadByThreadGroupAndName(ThreadGroup threadGroup, String name) {
    Thread[] threads = new Thread[threadGroup.activeCount()];
    threadGroup.enumerate(threads);

    for (Thread thread : threads) {
        if (thread != null && thread.getName().equals(name)) {
            return thread;
        }
    }
    return null; // Thread not found
}
```

以下是我们在此解决方案中所做的：

- activeCount()方法估计线程组中活动线程的数量。
- enumerate()方法用所有活动线程填充数组。
- 我们遍历数组来查找并返回与所需名称匹配的线程。
- 如果没有找到匹配项，我们将返回null。

我们对这个方法进行单元测试：

```java
@Test
public void givenThreadName_whenUsingThreadGroup_thenThreadFound() throws InterruptedException {
    ThreadGroup threadGroup = Thread.currentThread().getThreadGroup();
    Thread testThread = new Thread(threadGroup, () -> {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }, "TestThread");
    testThread.start();

    Thread foundThread = ThreadFinder.getThreadByThreadGroupAndName(threadGroup, "TestThread");
    assertNotNull(foundThread);
    assertEquals("TestThread", foundThread.getName());
    testThread.join();
}
```

## 5. 总结

在本文中，我们研究了两种在Java中通过名称获取线程的方法。Thread.getAllStackTraces()方法更简单，但可以检索所有线程而不限定范围，而ThreadGroup方法可以让我们更好地控制特定的线程组。