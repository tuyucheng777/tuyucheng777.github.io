---
layout: post
title:  使用JCTools的Java并发实用程序
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 概述

在本教程中，我们将介绍[JCTools](https://github.com/JCTools/JCTools)(Java并发工具)库。

简而言之，JCTools提供了许多适合在多线程环境中工作的实用数据结构。

## 2. 非阻塞算法

**传统上，在可变共享状态下工作的多线程代码使用锁来确保数据一致性和发布(一个线程所做的更改对另一个线程可见)**。

这种方法有许多缺点：

- 线程可能会在尝试获取锁时被阻塞，直到另一个线程的操作完成才会取得进展-这实际上阻止了并行性
- 锁争用越严重，JVM处理线程调度、管理争用和等待线程队列所花的时间就越多，而它所做的实际工作就越少
- 如果涉及多个锁，并且这些锁的获取/释放顺序错误，则可能会出现死锁
- 可能存在[优先级反转](https://en.wikipedia.org/wiki/Priority_inversion)风险-高优先级线程被锁定，试图获取低优先级线程持有的锁
- 大多数情况下，使用粗粒度锁，会严重损害并行性-细粒度锁需要更仔细的设计，增加锁开销，并且更容易出错

**另一种方法是使用非阻塞算法，即任何线程的失败或挂起不会导致另一个线程的失败或挂起的算法**。

如果至少有一个相关线程能够保证在任意时间段内取得进展，即在处理过程中不会出现死锁，则非阻塞算法是无锁的。

此外，如果每个线程的进度都有保证，那么这些算法就是无等待的。

这是来自优秀的[《Java并发实践》](https://www.amazon.com/Java-Concurrency-Practice-Brian-Goetz/dp/0321349601)一书的一个非阻塞Stack示例；它定义了基本状态：

```java
public class ConcurrentStack<E> {

    AtomicReference<Node<E>> top = new AtomicReference<Node<E>>();

    private static class Node <E> {
        public E item;
        public Node<E> next;

        // standard constructor
    }
}
```

还有一些API方法：

```java
public void push(E item){
    Node<E> newHead = new Node<E>(item);
    Node<E> oldHead;
    
    do {
        oldHead = top.get();
        newHead.next = oldHead;
    } while(!top.compareAndSet(oldHead, newHead));
}

public E pop() {
    Node<E> oldHead;
    Node<E> newHead;
    do {
        oldHead = top.get();
        if (oldHead == null) {
            return null;
        }
        newHead = oldHead.next;
    } while (!top.compareAndSet(oldHead, newHead));
    
    return oldHead.item;
}
```

我们可以看到，该算法使用细粒度的比较和交换([CAS](https://en.wikipedia.org/wiki/Compare-and-swap))指令并且是无锁的(即使多个线程同时调用top.compareAndSet()，其中一个也保证成功)，但不是无等待的，因为不能保证CAS最终对任何特定线程成功。

## 3. 依赖

首先，让我们将JCTools依赖项添加到我们的 pom.xml中：

```xml
<dependency>
    <groupId>org.jctools</groupId>
    <artifactId>jctools-core</artifactId>
    <version>2.1.2</version>
</dependency>
```

请注意，最新可用版本可在[Maven Central](https://mvnrepository.com/artifact/org.jctools/jctools-core)上找到。

## 4. JCTools队列

该库提供了许多可在多线程环境中使用的队列，即，一个或多个线程以线程安全无锁的方式写入队列，并且一个或多个线程从队列中读取。

所有Queue实现的通用接口是org.jctools.queues.MessagePassingQueue。

### 4.1 队列类型

所有队列都可以根据其生产者/消费者策略进行分类：

- **单生产者，单消费者**-此类类使用Spsc前缀，例如SpscArrayQueue
- **单生产者，多消费者**-此类类使用Spmc前缀，例如SpmcArrayQueue
- **多生产者，单消费者**-此类类使用Mpsc前缀，例如MpscArrayQueue
- **多生产者，多消费者**-此类类使用Mpmc前缀，例如MpmcArrayQueue

值得注意的是，**内部没有策略检查，也就是说，如果使用不正确，队列可能会悄悄地发生故障**。

例如，下面的测试从两个线程填充单个生产者队列并且通过，即使不能保证消费者看到来自不同生产者的数据：

```java
SpscArrayQueue<Integer> queue = new SpscArrayQueue<>(2);

Thread producer1 = new Thread(() -> queue.offer(1));
producer1.start();
producer1.join();

Thread producer2 = new Thread(() -> queue.offer(2));
producer2.start();
producer2.join();

Set<Integer> fromQueue = new HashSet<>();
Thread consumer = new Thread(() -> queue.drain(fromQueue::add));
consumer.start();
consumer.join();

assertThat(fromQueue).containsOnly(1, 2);
```

### 4.2 队列实现

总结上面的分类，以下是JCTools队列的列表：

- **SpscArrayQueue**：单个生产者，单个消费者，内部使用数组，容量受限
- **SpscLinkedQueue**：单个生产者，单个消费者，内部使用链表，容量不受限制
- **SpscChunkedArrayQueue**：单个生产者，单个消费者，从初始容量开始，逐渐增长至最大容量
- **SpscGrowableArrayQueue**：单个生产者，单个消费者，从初始容量开始，逐渐增长到最大容量。这与SpscChunkedArrayQueue的约定相同，唯一的区别是内部块管理。建议使用SpscChunkedArrayQueue，因为它具有简化的实现
- **SpscUnboundedArrayQueue**：单个生产者，单个消费者，内部使用数组，容量不受限制
- **SpmcArrayQueue**：单个生产者，多个消费者，内部使用数组，限制容量
- **MpscArrayQueue**：多个生产者，单个消费者，内部使用数组，限制容量
- **MpscLinkedQueue**：多个生产者，单个消费者，内部使用链表，容量不受限制
- **MpmcArrayQueue**：多个生产者，多个消费者，内部使用数组，限制容量

### 4.3 原子队列

上一节中提到的所有队列都使用[sun.misc.Unsafe](https://www.baeldung.com/java-unsafe)。然而，随着Java 9和[JEP-260](http://openjdk.java.net/jeps/260)的出现，这个API默认变得无法访问。

因此，有替代队列使用java.util.concurrent.atomic.AtomicLongFieldUpdater(公共API，性能较低)而不是sun.misc.Unsafe。

它们是由上面的队列生成的，并且它们的名称中间插入了单词Atomic，例如SpscChunkedAtomicArrayQueue或MpmcAtomicArrayQueue。

如果可能的话，建议使用“常规”队列，并且仅在sun.misc.Unsafe被禁止/无效的环境中(如HotSpot Java 9+和JRockit)才使用AtomicQueues。

### 4.4 容量

所有JCTools队列也可能有最大容量或不受限制，**当队列已满且受容量限制时，它将停止接收新元素**。

在以下示例中，我们：

- 排队
- 确保它在此之后停止接收新元素
- 排出并确保随后可以添加更多元素

请注意，为了便于阅读，删除了一些代码语句：

```java
SpscChunkedArrayQueue<Integer> queue = new SpscChunkedArrayQueue<>(8, 16);
CountDownLatch startConsuming = new CountDownLatch(1);
CountDownLatch awakeProducer = new CountDownLatch(1);

Thread producer = new Thread(() -> {
    IntStream.range(0, queue.capacity()).forEach(i -> {
        assertThat(queue.offer(i)).isTrue();
    });
    assertThat(queue.offer(queue.capacity())).isFalse();
    startConsuming.countDown();
    awakeProducer.await();
    assertThat(queue.offer(queue.capacity())).isTrue();
});

producer.start();
startConsuming.await();

Set<Integer> fromQueue = new HashSet<>();
queue.drain(fromQueue::add);
awakeProducer.countDown();
producer.join();
queue.drain(fromQueue::add);

assertThat(fromQueue).containsAll(
    IntStream.range(0, 17).boxed().collect(toSet()));
```

## 5. 其他JCTools数据结构

JCTools还提供一些非队列数据结构。

所有内容列示如下：

- **NonBlockingHashMap**：无锁ConcurrentHashMap替代方案，具有更好的扩展性，并且通常突变成本更低。它是通过sun.misc.Unsafe实现的，因此，不建议在HotSpot Java 9+或JRockit环境中使用此类
- **NonBlockingHashMapLong**：与NonBlockingHashMap类似，但使用原始long键
- **NonBlockingHashSet**：NonBlockingHashMap的简单包装器，类似JDK的java.util.Collections.newSetFromMap()
- **NonBlockingIdentityHashMap**：与NonBlockingHashMap类似，但通过对象标识比较键
- **NonBlockingSetInt**：一个多线程位向量集，以原始long数组的形式实现，在静默自动装箱的情况下效果不佳

## 6. 性能测试

让我们使用[JMH](http://openjdk.java.net/projects/code-tools/jmh/)来比较JDK的ArrayBlockingQueue与JCTools队列的性能，JMH是Sun/Oracle JVM专家开发的一个开源微基准测试框架，可以让我们免受编译器/JVM优化算法的不确定性的影响。

请注意，为了提高可读性，下面的代码片段遗漏了几个语句：

```java
public class MpmcBenchmark {

    @Param({PARAM_UNSAFE, PARAM_AFU, PARAM_JDK})
    public volatile String implementation;

    public volatile Queue<Long> queue;

    @Benchmark
    @Group(GROUP_NAME)
    @GroupThreads(PRODUCER_THREADS_NUMBER)
    public void write(Control control) {
        // noinspection StatementWithEmptyBody
        while (!control.stopMeasurement && !queue.offer(1L)) {
            // intentionally left blank
        }
    }

    @Benchmark
    @Group(GROUP_NAME)
    @GroupThreads(CONSUMER_THREADS_NUMBER)
    public void read(Control control) {
        // noinspection StatementWithEmptyBody
        while (!control.stopMeasurement && queue.poll() == null) {
            // intentionally left blank
        }
    }
}
```

结果(摘录自第95个百分位数，每个操作耗时纳秒)：

```text
MpmcBenchmark.MyGroup:MyGroup·p0.95 MpmcArrayQueue sample 1052.000 ns/op
MpmcBenchmark.MyGroup:MyGroup·p0.95 MpmcAtomicArrayQueue sample 1106.000 ns/op
MpmcBenchmark.MyGroup:MyGroup·p0.95 ArrayBlockingQueue sample 2364.000 ns/op
```

我们可以看到**MpmcArrayQueue的性能略优于MpmcAtomicArrayQueue，而ArrayBlockingQueue的速度则慢了两倍**。 

## 7. 使用JCTools的缺点

**使用JCTools有一个重要的缺点-无法强制正确使用库类**。例如，考虑一下当我们在大型成熟项目中开始使用MpscArrayQueue的情况(请注意，必须有一个消费者)。

不幸的是，由于项目很大，有人可能会犯编程或配置错误，导致队列现在从多个线程读取。系统似乎像以前一样工作，但现在消费者可能会错过一些消息。这是一个真正的问题，可能会产生很大的影响，而且很难调试。

理想情况下，应该可以运行具有特定系统属性的系统，从而强制JCTools确保线程访问策略。例如，本地/测试/暂存环境(但不是生产环境)可能已启用该属性。遗憾的是，JCTools不提供这样的属性。

另一个考虑因素是，尽管我们确保JCTools比JDK的对应版本快得多，但这并不意味着我们的应用程序在开始使用自定义队列实现时会获得相同的速度。大多数应用程序不会在线程之间交换大量对象，并且大多受I/O限制。

## 8. 总结

现在，我们对JCTools提供的实用程序类有了基本的了解，并且了解了它们与JDK的同类产品相比在高负载下的表现如何。

总之，仅当我们在线程之间交换大量对象时才值得使用该库，即使这样，也必须非常小心地保留线程访问策略。