---
layout: post
title:  线程安全的后进先出数据结构实现
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在本教程中，**我们将讨论线程安全的LIFO数据结构实现的各种选项**。

在后进先出数据结构中，元素的插入和检索是按照后进先出的原则进行的。这意味着首先检索最后插入的元素。

**在计算机科学中，堆栈是用来指代这种数据结构的术语**。

堆栈可以方便地处理一些有趣的问题，如表达式计算、实现撤消操作等。由于它可以在并发执行环境中使用，我们可能需要使其成为线程安全的。

## 2. 了解堆栈

基本上，**Stack必须实现以下方法**：

1.  push()：在顶部添加一个元素
2.  pop()：获取并删除顶部元素
3.  peek()：获取元素而不从底层容器中移除

如前所述，假设我们需要一个命令处理引擎。

在这个系统中，撤消已执行的命令是一个重要的功能。

一般情况下，将所有命令压入栈中，然后可以简单地实现撤销操作：

-   pop()方法获取最后执行的命令
-   在弹出的命令对象上调用undo()方法

## 3. 了解堆栈中的线程安全

**如果一个数据结构不是线程安全的，则在并发访问时，它可能最终会出现争用条件**。

简而言之，当代码的正确执行取决于线程的时间和顺序时，就会发生争用条件。如果多个线程共享数据结构并且此结构不是为此目的而设计的，则主要会发生这种情况。

让我们从Java Collection类ArrayDeque检查下面的方法：

```java
public E pollFirst() {
    int h = head;
    E result = (E) elements[h];
    // ... other book-keeping operations removed, for simplicity
    head = (h + 1) & (elements.length - 1);
    return result;
}
```

为了解释上述代码中潜在的争用条件，让我们假设有两个线程按以下顺序执行此代码：

-   第一个线程执行第三行：用索引“head”处的元素设置result对象
-   第二个线程执行第三行：用索引“head”处的元素设置result对象
-   第一个线程执行第五行：将索引“head”重置为后备数组中的下一个元素
-   第二个线程执行第五行：将索引“head”重置为后备数组中的下一个元素

现在，两次执行都将返回相同的result对象。

为避免此类争用条件，在这种情况下，一个线程不应执行第一行，直到另一个线程在第五行完成重置“head”索引。换句话说，访问索引“head”处的元素并重置索引“head”应该对线程以原子方式发生。

显然，在这种情况下，代码的正确执行取决于线程的计时，因此它不是线程安全的。

## 4. 使用锁的线程安全栈

在本节中，我们将讨论线程安全堆栈的具体实现的两个可能选项。

特别是，我们将介绍Java Stack和线程安全的装饰ArrayDeque。

两者都使用[锁](https://www.baeldung.com/java-concurrent-locks)来进行互斥访问。

### 4.1 使用Java Stack

**Java Collections有一个基于Vector的线程安全[Stack](https://www.baeldung.com/java-stack)的遗留实现，它基本上是ArrayList的同步变体**。

但是，官方文档本身建议考虑使用ArrayDeque。因此，我们不会涉及太多细节。

虽然Java Stack是线程安全的并且使用起来很直接，但是这个类有一些主要的缺点：

-   它不支持设置初始容量
-   它对所有操作都使用锁，这可能会损害单线程执行的性能

### 4.2 使用ArrayDeque

**使用Deque接口是后进先出数据结构最方便的方法，因为它提供了所有需要的堆栈操作**。[ArrayDeque](https://www.baeldung.com/java-array-deque)就是这样一个具体的实现。

由于它没有为操作使用锁，单线程执行可以正常工作。但是对于多线程执行，这是有问题的。

但是，我们可以为ArrayDeque实现一个同步装饰器。虽然这与Java集合框架的Stack类类似，但解决了Stack类的重要问题，即缺少初始容量设置。

让我们来看看这个类：

```java
public class DequeBasedSynchronizedStack<T> {

    // Internal Deque which gets decorated for synchronization.
    private ArrayDeque<T> dequeStore;

    public DequeBasedSynchronizedStack(int initialCapacity) {
        this.dequeStore = new ArrayDeque<>(initialCapacity);
    }

    public DequeBasedSynchronizedStack() {
        dequeStore = new ArrayDeque<>();
    }

    public synchronized T pop() {
        return this.dequeStore.pop();
    }

    public synchronized void push(T element) {
        this.dequeStore.push(element);
    }

    public synchronized T peek() {
        return this.dequeStore.peek();
    }

    public synchronized int size() {
        return this.dequeStore.size();
    }
}
```

请注意，为简单起见，我们的解决方案并未实现Deque本身，因为它包含更多方法。

此外，Guava包含[SynchronizedDeque](https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Queues.html#synchronizedDeque-java.util.Deque-)，它是装饰ArrayDequeue的生产就绪实现。

## 5. 无锁线程安全栈

[ConcurrentLinkedDeque](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ConcurrentLinkedDeque.html)是Deque接口的无锁实现。**该实现是完全线程安全的**，因为它使用了[高效的无锁算法](http://www.cs.rochester.edu/~scott/papers/1996_PODC_queues.pdf)。

与基于锁的实现不同，无锁实现不受以下问题的影响。

-   **[优先级反转](https://www.semanticscholar.org/paper/Avoidance-of-Priority-Inversion-in-Real-Time-Based-Helmy-Jafri/d286108f62af8f65ad8acad184a5360e3acbc112)**：当低优先级线程持有高优先级线程所需的锁时，就会发生这种情况。这可能会导致高优先级线程阻塞
-   **死锁**：当不同的线程以不同的顺序锁定同一组资源时，就会发生这种情况

最重要的是，无锁实现有一些特性，使它们非常适合在单线程和多线程环境中使用。

-   **对于非共享数据结构和单线程访问，性能将与ArrayDeque相当**
-   对于共享数据结构，**性能根据同时访问它的线程数而变化**

在可用性方面，它与ArrayDeque没有什么不同，因为两者都实现了Deque接口。

## 6. 总结

在本文中，我们讨论了堆栈数据结构及其在设计命令处理引擎和表达式计算器等系统时的优势。

此外，我们还分析了Java集合框架中的各种堆栈实现，并讨论了它们在性能和线程安全方面的细微差别。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-1)上获得。