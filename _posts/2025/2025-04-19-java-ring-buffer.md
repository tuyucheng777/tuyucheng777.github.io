---
layout: post
title:  在Java中实现环形缓冲区
category: algorithms
copyright: algorithms
excerpt: 环形缓冲区
---

## 1. 概述

在本教程中，我们将学习如何在Java中实现环形缓冲区。

## 2. 环形缓冲区

**环形缓冲区(或循环缓冲区)是一个有界的环形数据结构，用于在两个或多个线程之间缓冲数据**。当我们不断向环形缓冲区写入数据时，它会在到达末尾时绕回。

### 2.1 工作原理

**环形缓冲区是使用在边界处环绕的固定大小数组实现的**。

除了数组之外，它还跟踪三件事：

- 缓冲区中下一个可用插槽，用于插入元素
- 缓冲区中下一个未读元素
- 以及数组的末尾-缓冲区绕回到数组开头的点

![](/assets/images/2025/algorithms/javaringbuffer01.png)

环形缓冲区处理这些需求的机制因具体实现而异，例如，[维基百科](https://en.wikipedia.org/wiki/Circular_buffer#Circular_buffer_mechanics)中关于此主题的条目展示了一种使用四指针的方法。

我们将借用[Disruptor](https://www.baeldung.com/lmax-disruptor-concurrency)使用序列实现环形缓冲区的方法。

我们首先需要知道的是容量-缓冲区的固定最大大小。接下来，**我们将使用两个单调递增的序列**：

- 写序列：从-1开始，每插入一个元素就增加1
- 读序列：从0开始，每消耗一个元素就增加1

我们可以使用mod运算将序列映射到数组中的索引：

```text
arrayIndex = sequence % capacity
```

**模运算将序列环绕边界以得出缓冲区中的槽**：

![](/assets/images/2025/algorithms/javaringbuffer02.png)

让我们看看如何插入元素：

```text
buffer[++writeSequence % capacity] = element
```

我们在插入元素之前预先自增序列。

为了消耗一个元素，我们进行后增：

```text
element = buffer[readSequence++ % capacity]
```

在这种情况下，我们对序列执行后增操作。**消费元素并不会将其从缓冲区中移除-它只是保留在数组中，直到被覆盖**。

### 2.2 空缓冲区和满缓冲区

当我们绕回数组时，我们将开始覆盖缓冲区中的数据。**如果缓冲区已满，我们可以选择覆盖最旧的数据(无论读取器是否已读取)，或者阻止覆盖尚未读取的数据**。

如果读取器能够承受错过中间值或旧值(例如，股票价格行情)，我们可以覆盖数据，而无需等待数据被消费。另一方面，如果读取器必须消费所有值(例如电子商务交易)，我们应该等待(阻塞/忙等待)，直到缓冲区有可用槽。

**如果缓冲区的大小等于其容量，则缓冲区已满**，其中其大小等于未读元素的数量：

```text
size = (writeSequence - readSequence) + 1
isFull = (size == capacity)
```

![](/assets/images/2025/algorithms/javaringbuffer03.png)

**如果写入序列落后于读取序列，则缓冲区为空**：

```text
isEmpty = writeSequence < readSequence
```

![](/assets/images/2025/algorithms/javaringbuffer04.png)

如果缓冲区为空，则返回null值。

### 2.3 优点和缺点

环形缓冲区是一种高效的FIFO缓冲区，它使用一个固定大小的数组，该数组可以预先分配，并支持高效的内存访问模式。所有缓冲区操作(包括元素消费)的时间复杂度均为常数时间O(1)，因为它不需要移动元素。

另一方面，确定环形缓冲区的正确大小至关重要。例如，如果缓冲区大小不足，读取速度缓慢，写入操作可能会阻塞很长时间。我们可以使用动态调整大小，但这需要移动数据，并且会失去上面讨论的大多数优点。

## 3. Java实现

现在我们了解了环形缓冲区的工作原理，让我们继续用Java实现它。

### 3.1 初始化

首先，让我们定义一个构造函数，用预定义的容量初始化缓冲区：

```java
public CircularBuffer(int capacity) {
    this.capacity = (capacity < 1) ? DEFAULT_CAPACITY : capacity;
    this.data = (E[]) new Object[this.capacity];
    this.readSequence = 0;
    this.writeSequence = -1;
}
```

这将创建一个空缓冲区并初始化序列字段，如上一节所述。

### 3.2 Offer

接下来，我们将实现offer操作，该操作将一个元素插入到缓冲区的下一个可用位置，并在成功时返回true。如果缓冲区找不到空位置(即我们无法覆盖未读值)，则返回false。

让我们用Java实现offer方法：

```java
public boolean offer(E element) {
    boolean isFull = (writeSequence - readSequence) + 1 == capacity;
    if (!isFull) {
        int nextWriteSeq = writeSequence + 1;
        data[nextWriteSeq % capacity] = element;
        writeSequence++;
        return true;
    }
    return false;
}
```

因此，我们递增写序列，并计算数组中下一个可用槽的索引。然后，我们将数据写入缓冲区并存储更新后的写序列。

让我们测试一下：

```java
@Test
public void givenCircularBuffer_whenAnElementIsEnqueued_thenSizeIsOne() {
    CircularBuffer buffer = new CircularBuffer<>(defaultCapacity);

    assertTrue(buffer.offer("Square"));
    assertEquals(1, buffer.size());
}
```

### 3.3 Poll

最后，我们将实现poll操作，用于检索并删除下一个未读元素。**poll操作不会删除元素，而是自增读序列**。

让我们来实现它：

```java
public E poll() {
    boolean isEmpty = writeSequence < readSequence;
    if (!isEmpty) {
        E nextValue = data[readSequence % capacity];
        readSequence++;
        return nextValue;
    }
    return null;
}
```

这里，我们通过计算数组中的索引，按照当前读序列读取数据。然后，如果缓冲区不为空，则自增序列并返回值。

让我们测试一下：

```java
@Test
public void givenCircularBuffer_whenAnElementIsDequeued_thenElementMatchesEnqueuedElement() {
    CircularBuffer buffer = new CircularBuffer<>(defaultCapacity);
    buffer.offer("Triangle");
    String shape = buffer.poll();

    assertEquals("Triangle", shape);
}
```

## 4. 生产者-消费者问题

我们讨论了如何使用环形缓冲区在两个或多个线程之间交换数据，这是一个称为[生产者-消费者问题](https://en.wikipedia.org/wiki/Producer–consumer_problem)的同步问题的示例。在Java中，我们可以使用[信号量](https://www.baeldung.com/java-semaphore)、[有界队列](https://www.baeldung.com/java-blocking-queue#multithreaded-producer-consumer-example)、环形缓冲区等各种方法解决生产者-消费者问题。

让我们实现一个基于环形缓冲区的解决方案。

### 4.1 volatile序列字段

**我们对环形缓冲区的实现并非线程安全，让我们针对简单的单生产者和单消费者情况，将其实现为线程安全的**。

生产者将数据写入缓冲区并增加writeSequence的值，而消费者仅从缓冲区读取数据并增加readSequence的值。因此，后备数组是无争用的，我们无需任何同步。

但我们仍然需要确保消费者可以看到writeSequence字段的最新值([可见性](https://www.baeldung.com/java-volatile#1-memory-visibility))，并且在数据实际在缓冲区中可用之前不会更新writeSequence([排序](https://www.baeldung.com/java-volatile#2-reordering))。

**在这种情况下，我们可以通过将序列字段设置为[volatile](https://www.baeldung.com/java-volatile)来实现环形缓冲区的并发和无锁**：

```java
private volatile int writeSequence = -1, readSequence = 0;
```

在offer方法中，对volatile字段writeSequence的写入保证了写入缓冲区的操作发生在更新序列之前。同时，volatile可见性保证确保消费者始终能够看到writeSequence的最新值。

### 4.2 生产者

让我们实现一个写入环形缓冲区的简单生产者Runnable：

```java
public void run() {
    for (int i = 0; i < items.length;) {
        if (buffer.offer(items[i])) {
           System.out.println("Produced: " + items[i]);
            i++;
        }
    }
}
```

生产者线程将循环等待空槽(忙等待)。

### 4.3 消费者

我们将实现一个从缓冲区读取的消费者Callable：

```java
public T[] call() {
    T[] items = (T[]) new Object[expectedCount];
    for (int i = 0; i < items.length;) {
        T item = buffer.poll();
        if (item != null) {
            items[i++] = item;
            System.out.println("Consumed: " + item);
        }
    }
    return items;
}
```

如果消费者线程从缓冲区接收到null值，则它将继续而不进行打印。

让我们编写驱动程序代码：

```java
executorService.submit(new Thread(new Producer<String>(buffer)));
executorService.submit(new Thread(new Consumer<String>(buffer)));
```

执行我们的生产者-消费者程序会产生如下输出：

```text
Produced: Circle
Produced: Triangle
  Consumed: Circle
Produced: Rectangle
  Consumed: Triangle
  Consumed: Rectangle
Produced: Square
Produced: Rhombus
  Consumed: Square
Produced: Trapezoid
  Consumed: Rhombus
  Consumed: Trapezoid
Produced: Pentagon
Produced: Pentagram
Produced: Hexagon
  Consumed: Pentagon
  Consumed: Pentagram
Produced: Hexagram
  Consumed: Hexagon
  Consumed: Hexagram
```

## 5.总结

在本教程中，我们学习了如何实现环形缓冲区，并探讨了如何使用它来解决生产者-消费者问题。