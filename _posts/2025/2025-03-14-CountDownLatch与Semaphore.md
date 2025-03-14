---
layout: post
title:  CountDownLatch与Semaphore
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 简介

在Java多线程中，线程之间的有效协调对于确保正确同步和防止数据损坏至关重要。线程协调的两种常用机制是[CountDownLatch](https://www.baeldung.com/java-countdown-latch)和[Semaphore](https://www.baeldung.com/java-semaphore)，在本教程中，我们将探讨CountDownLatch和Semaphore之间的区别，并讨论何时使用它们。

## 2. 背景

让我们探索这些同步机制背后的基本概念。

### 2.1 CountDownLatch

**CountDownLatch允许一个或多个线程正常暂停，直到指定的一组任务完成**。它通过递减计数器直到计数器达到0来运行，这表示所有先决条件任务都已完成。

### 2.2 Semaphore

**Semaphore是一种同步工具，它通过使用许可证来控制对共享资源的访问**。与CountDownLatch相比，Semaphore许可证可以在整个应用程序中多次释放和获取，从而允许对并发管理进行更细粒度的控制。

## 3. CountDownLatch与Semaphore的区别

在本节中，我们将深入探讨这些同步机制之间的主要区别。

### 3.1 计数机制

CountDownLatch以初始计数开始运行，该计数随着任务的完成而减少，**一旦计数达到0，等待的线程就会被释放**。

Semaphore维护一组许可证，每个许可证代表访问共享资源的权限。**线程获取访问资源的许可证，并在完成后释放它们**。

### 3.2 可复位性

**Semaphore许可证可以多次释放和获取，从而实现动态资源管理**。例如，如果我们的应用程序突然需要更多数据库连接，我们可以释放额外的许可以动态增加可用连接的数量。

而在CountDownLatch中，一旦计数达到0，就无法重置或重新用于另一个同步事件，它专为一次性用例而设计。

### 3.3 动态许可计数

可以使用acquire()和release()方法在运行时动态调整Semaphore许可，**这样可以动态更改允许同时访问共享资源的线程数**。

另一方面，一旦CountDownLatch用计数初始化，它就保持不变，并且在运行时不能改变。

### 3.4 公平性

Semaphore支持公平性概念，确保等待获取许可的线程按到达的顺序获得服务(先进先出)，这有助于防止在高争用情况下出现线程匮乏的情况。

**相比之下，CountDownLatch没有公平性概念**，它通常用于一次性同步事件，其中线程执行的具体顺序不太重要。

### 3.5 用例

CountDownLatch常用于协调多个线程的启动、等待并行操作完成、在执行主任务之前同步系统初始化等场景。例如，在并发数据处理应用程序中，CountDownLatch可以确保在开始数据分析之前所有数据加载任务都已完成。

另一方面，Semaphore适用于管理对共享资源的访问、实现资源池、控制对关键代码段的访问或限制并发数据库连接的数量。例如，在数据库连接池系统中，Semaphore可以限制并发数据库连接的数量，以防止数据库服务器不堪重负。

### 3.6 性能

由于CountDownLatch主要涉及递减计数器，因此它在处理和资源利用方面产生的开销很小。**此外，Semaphore在管理许可方面引入了开销，特别是在频繁获取和释放许可时**，每次调用acquire()和release()都涉及额外的处理来管理许可计数，这可能会影响性能，尤其是在高并发场景中。

### 3.7 总结

下表总结了CountDownLatch和Semaphore在各个方面的主要区别：

|     特征     |      CountDownLatch      |            Semaphore             |
| :----------: | :----------------------: | :------------------------------: |
|     作用     | 同步线程直到一组任务完成 |       控制对共享资源的访问       |
|   计数机制   |        递减计数器        |         管理许可证(令牌)         |
|   可复位性   |   不可重置(一次性同步)   | 可重置(许可证可以多次发布和获取) |
| 动态许可计数 |            否            |    是(许可证可以在运行时调整)    |
|    公平性    |    没有具体的公平保证    |     提供公平性(先进先出顺序)     |
|     性能     |     低开销(最少处理)     |   由于许可证管理，管理代价略高   |

## 4. 实现方式比较

在本节中，我们将重点介绍CountDownLatch和Semaphore在语法和功能上的实现差异。

### 4.1 CountDownLatch实现

首先，我们创建一个CountDownLatch，其初始计数等于要完成的任务数。每个工作线程模拟一项任务，并在任务完成后使用countDown()方法减少计数。主线程使用await()方法等待所有任务完成：

```java
int numberOfTasks = 3;
CountDownLatch latch = new CountDownLatch(numberOfTasks);

for (int i = 1; i <= numberOfTasks; i++) {
    new Thread(() -> {
        System.out.println("Task completed by Thread " + Thread.currentThread().getId());
        latch.countDown();
    }).start();
}

latch.await();
System.out.println("All tasks completed. Main thread proceeds.");
```

所有任务完成且计数达到零后，尝试调用countDown()将不起作用。此外，由于计数已经为0，因此对await()的任何后续调用都会立即返回，而不会阻塞线程：

```java
latch.countDown();
latch.await(); // This line won't block
System.out.println("Latch is already at zero and cannot be reset.");
```

现在让我们观察程序的执行并检查输出：

```text
Task completed by Thread 11
Task completed by Thread 12
Task completed by Thread 13
All tasks completed. Main thread proceeds.
Latch is already at zero and cannot be reset.
```

### 4.2 Semaphore实现

在此示例中，我们创建一个具有固定许可数NUM_PERMITS的信号量，每个工作线程通过在访问资源之前使用acquire()方法获取许可来模拟资源访问。需要注意的一点是，当线程调用acquire()方法获取许可时，它可能会在等待许可时被中断。因此，在try–catch块中捕获[InterruptedException](https://www.baeldung.com/java-interrupted-exception)以妥善处理此中断至关重要。

完成资源访问后，线程使用release()方法释放许可：

```java
int NUM_PERMITS = 3;
Semaphore semaphore = new Semaphore(NUM_PERMITS);

for (int i = 1; i <= 5; i++) {
    new Thread(() -> {
        try {
            semaphore.acquire();
            System.out.println("Thread " + Thread.currentThread().getId() + " accessing resource.");
            Thread.sleep(2000); // Simulating resource usage
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            semaphore.release();
        }
    }).start();
}
```

接下来，我们通过释放额外的许可来模拟重置Semaphore，使计数恢复到初始许可值，这表明Semaphore许可可以在运行时动态调整或重置：

```java
try {
    Thread.sleep(5000);
    semaphore.release(NUM_PERMITS); // Resetting the semaphore permits to the initial count
    System.out.println("Semaphore permits reset to initial count.");
} catch (InterruptedException e) {
    e.printStackTrace();
}
```

程序运行后的输出如下：

```text
Thread 11 accessing resource.
Thread 12 accessing resource.
Thread 13 accessing resource.
Thread 14 accessing resource.
Thread 15 accessing resource.
Semaphore permits reset to initial count.
```

## 5. 总结

在本文中，我们探讨了CountDownLatch和Semaphore的主要特性。CountDownLatch非常适合需要完成一组固定任务才能允许线程继续执行的场景，因此适合一次性同步事件。相比之下，Semaphore用于通过限制可以同时访问共享资源的线程数来控制对共享资源的访问，从而提供对并发管理的更细粒度的控制。