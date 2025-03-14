---
layout: post
title:  Java并发队列指南
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 概述

在本教程中，我们将介绍Java中并发队列的一些主要实现。

## 2. 队列

在多线程应用程序中，队列需要处理多个并发生产者-消费者场景。**正确选择并发队列对于实现算法的良好性能至关重要**。 

首先，我们将了解阻塞队列和非阻塞队列之间的一些重要区别。然后，我们将了解一些实现和最佳实践。

## 3. 阻塞队列与非阻塞队列

**[BlockingQueue](https://www.baeldung.com/java-blocking-queue)提供了一种简单的线程安全机制**，在此队列中，线程需要等待队列可用。生产者将等待可用容量后再添加元素，而消费者将等到队列为空。在这些情况下，非阻塞队列将抛出异常或返回特殊值，如null或false。

为了实现这种阻塞机制，BlockingQueue接口在普通Queue函数之上公开了两个函数：put和take。这两个函数相当于标准Queue中的add和remove。

## 4. 并发队列实现

### 4.1 ArrayBlockingQueue

顾名思义，此队列在内部使用数组。因此，**它是一个有界队列，意味着它具有固定大小**。

一个简单的工作队列就是一个示例用例。这种场景通常生产者与消费者的比例较低，我们将耗时的任务分摊给多个工作者。由于这个队列不能无限增长，**因此如果内存是一个问题，大小限制将充当安全阈值**。

说到内存，需要注意的是队列会预先分配数组。虽然这可能会提高吞吐量，**但也可能消耗比必要更多的内存**。例如，大容量队列可能会长时间保持空置状态。

此外，ArrayBlockingQueue对put和take操作都使用单个锁。这确保不会覆盖条目，但会降低性能。

### 4.2 LinkedBlockingQueue

[LinkedBlockingQueue](https://www.baeldung.com/java-queue-linkedblocking-concurrentlinked#linkedblockingqueue)使用[LinkedList](https://www.baeldung.com/java-linkedlist)变体，其中每个队列项都是一个新节点。虽然这在原则上使队列不受限制，但它仍然具有Integer.MAX_VALUE的硬性限制。

另一方面，我们可以使用构造函数LinkedBlockingQueue(int capacity)设置队列大小。

此队列对put和take操作使用不同的锁，因此，两个操作可以并行进行并提高吞吐量。

既然LinkedBlockingQueue可以是有界的也可以是无界的，那我们为什么要使用ArrayBlockingQueue而不是这个呢？**每次从队列中添加或删除元素时，LinkedBlockingQueue都需要分配和释放节点**。因此，如果队列增长和收缩都很快，那么ArrayBlockingQueue可能是更好的选择。

据说LinkedBlockingQueue的性能是不可预测的，换句话说，我们总是需要分析我们的场景以确保我们使用正确的数据结构。

### 4.3 PriorityBlockingQueue

**当我们需要按特定顺序使用元素时，[PriorityBlockingQueue](https://www.baeldung.com/java-priority-blocking-queue)是我们的首选解决方案**。为了实现这一点，PriorityBlockingQueue使用基于数组的二叉堆。

虽然它在内部使用单个锁机制，但take操作可以与put操作同时发生。使用简单的自旋锁即可实现这一点。

一个典型的用例是使用具有不同优先级的任务，**我们不希望低优先级的任务取代高优先级的任务**。

### 4.4 DelayQueue

当消费者只能取用过期元素时，我们会使用[DelayQueue](https://www.baeldung.com/java-delay-queue)。有趣的是，它内部使用PriorityQueue按元素过期时间对元素进行排序。

由于这不是通用队列，因此它不像ArrayBlockingQueue或LinkedBlockingQueue那样涵盖那么多场景。例如，我们可以使用此队列实现类似于NodeJS中的简单事件循环，将异步任务放入队列中，以便在它们到期后进行处理。

### 4.5 LinkedTransferQueue

LinkedTransferQueue引入了一种transfer方法，其他队列通常在生产或消费元素时阻塞，**而[LinkedTransferQueue](https://www.baeldung.com/java-transfer-queue)允许生产者等待元素的消费**。

当我们需要确保放入队列中的特定元素已被取走时，我们会使用LinkedTransferQueue。此外，我们可以使用此队列实现一个简单的背压算法。事实上，通过阻塞生产者直到消费，**消费者可以推动生产的消息流**。

### 4.6 SynchronousQueue

虽然队列通常包含许多元素，但[SynchronousQueue](https://www.baeldung.com/java-synchronous-queue)最多只有一个元素。换句话说，我们需要**将SynchronousQueue视为在两个线程之间交换一些数据的简单方法**。

当我们有两个线程需要访问共享状态时，我们经常使用[CountDownLatch](https://www.baeldung.com/java-countdown-latch)或其他同步机制来同步它们。通过使用SynchronousQueue，我们可以避免这种手动线程同步。

### 4.7 ConcurrentLinkedQueue

[ConcurrentLinkedQueue](https://www.baeldung.com/java-queue-linkedblocking-concurrentlinked#concurrentlinkedqueue)是本指南中唯一的非阻塞队列，因此，它提供了一种“无等待”算法，其中**add和poll保证线程安全并立即返回**。此队列使用[CAS(比较和交换)](https://www.baeldung.com/lock-free-programming)而不是锁。

在内部，它基于Maged M. Michael和Michael L. Scott编写的[《简单、快速、实用的非阻塞和阻塞并发队列算法》中的一种算法](https://www.cs.rochester.edu/u/scott/papers/1996_PODC_queues.pdf)。

**它是现代[响应式系统](https://www.baeldung.com/java-reactive-systems)的完美候选者**，在现代反应系统中，使用阻塞数据结构通常是被禁止的。

另一方面，如果我们的消费者最终陷入循环等待，我们可能应该选择阻塞队列作为更好的替代方案。

## 5. 总结

在本指南中，我们介绍了不同的并发队列实现，并讨论了它们的优点和缺点。考虑到这一点，我们可以更好地开发高效、耐用且可用的系统。