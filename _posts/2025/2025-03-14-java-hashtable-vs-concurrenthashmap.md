---
layout: post
title:  Java中Hashtable和ConcurrentHashMap之间的区别
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 概述

在Java应用程序中管理键值对时，我们经常会考虑两个主要选项：[Hashtable](https://www.baeldung.com/java-hash-table)和[ConcurrentHashMap](https://www.baeldung.com/java-concurrent-map)。

**虽然这两个集合都具有线程安全的优势，但它们的底层架构和功能却有很大不同**。无论我们是在构建遗留系统还是在开发基于微服务的现代云应用程序，了解这些细微差别对于做出正确的选择都至关重要。

在本教程中，我们将剖析Hashtable和ConcurrentHashMap之间的区别，深入研究它们的性能指标、同步功能和其他各个方面，以帮助我们做出明智的决定。

## 2. Hashtable

Hashtable是Java中最古老的集合类之一，自JDK 1.0以来就已存在，它提供键值存储和检索API：

```java
Hashtable<String, String> hashtable = new Hashtable<>();
hashtable.put("Key1", "1");
hashtable.put("Key2", "2");
hashtable.putIfAbsent("Key3", "3");
String value = hashtable.get("Key2");
```

**Hashtable的主要卖点是线程安全，这是通过方法级同步实现的**。

put()、putIfAbsent()、get()和remove()等方法是同步的。在给定时间内，只有一个线程可以在Hashtable实例上执行这些方法中的任何一个，从而确保数据一致性。

## 3. ConcurrentHashMap

ConcurrentHashMap是一个更现代的替代方案，它是作为Java 5的一部分与[Java集合框架](https://www.baeldung.com/java-collections)一起引入的。

Hashtable和ConcurrentHashMap都实现了Map接口，这解释了方法签名的相似性：

```java
ConcurrentHashMap<String, String> concurrentHashMap = new ConcurrentHashMap<>();
concurrentHashMap.put("Key1", "1");
concurrentHashMap.put("Key2", "2");
concurrentHashMap.putIfAbsent("Key3", "3");
String value = concurrentHashMap.get("Key2");
```

## 4. 差异

在本节中，我们将研究Hashtable和ConcurrentHashMap之间的主要区别，包括并发性、性能和内存使用情况。

### 4.1 并发

正如我们前面讨论的，Hashtable通过方法级同步实现线程安全。

**另一方面，ConcurrentHashMap提供了更高并发性的线程安全性**。它允许多个线程同时读取和执行有限的写入，而无需锁定整个数据结构，这在读操作多于写操作的应用程序中尤其有用。

### 4.2 性能

虽然Hashtable和ConcurrentHashMap都保证线程安全，但由于底层同步机制不同，它们在性能上有所不同。

**Hashtable在写操作期间锁定整个表，从而阻止其他读取或写入，这在高并发环境中可能成为瓶颈**。

**但是，ConcurrentHashMap允许并发读取和有限的并发写入，这使得它在实践中更具可扩展性并且通常更快**。

对于较小的数据集，性能数字的差异可能不明显。然而，ConcurrentHashMap通常会在较大的数据集和更高的并发级别中显示其优势。

为了证实性能数据，我们使用[JMH(Java微基准测试工具)](https://www.baeldung.com/java-microbenchmark-harness)运行基准测试，它使用10个线程来模拟并发活动，并执行三次预热迭代，然后进行五次测量迭代。它测量每个基准测试方法所花费的平均时间，表明平均执行时间：

```java
@Benchmark
@Group("hashtable")
public void benchmarkHashtablePut() {
    for (int i = 0; i < 10000; i++) {
        hashTable.put(String.valueOf(i), i);
    }
}

@Benchmark
@Group("hashtable")
public void benchmarkHashtableGet(Blackhole blackhole) {
    for (int i = 0; i < 10000; i++) {
        Integer value = hashTable.get(String.valueOf(i));
        blackhole.consume(value);
    }
}

@Benchmark
@Group("concurrentHashMap")
public void benchmarkConcurrentHashMapPut() {
    for (int i = 0; i < 10000; i++) {
        concurrentHashMap.put(String.valueOf(i), i);
    }
}

@Benchmark
@Group("concurrentHashMap")
public void benchmarkConcurrentHashMapGet(Blackhole blackhole) {
    for (int i = 0; i < 10000; i++) {
        Integer value = concurrentHashMap.get(String.valueOf(i));
        blackhole.consume(value);
    }
}
```

测试结果如下：

```text
Benchmark                                                        Mode  Cnt   Score   Error
BenchMarkRunner.concurrentHashMap                                avgt    5   1.788 ± 0.406
BenchMarkRunner.concurrentHashMap:benchmarkConcurrentHashMapGet  avgt    5   1.157 ± 0.185
BenchMarkRunner.concurrentHashMap:benchmarkConcurrentHashMapPut  avgt    5   2.419 ± 0.629
BenchMarkRunner.hashtable                                        avgt    5  10.744 ± 0.873
BenchMarkRunner.hashtable:benchmarkHashtableGet                  avgt    5  10.810 ± 1.208
BenchMarkRunner.hashtable:benchmarkHashtablePut                  avgt    5  10.677 ± 0.541
```

基准测试结果提供了对Hashtable和ConcurrentHashMap特定方法的平均执行时间的了解。

**分数越低表示性能越好，结果表明，平均而言，ConcurrentHashMap在get()和put()操作方面都优于Hashtable**。 

### 4.3 Hashtable迭代器

Hashtable迭代器是[“快速失败”](https://www.baeldung.com/java-fail-safe-vs-fail-fast-iterator)的，这意味着如果在创建迭代器后修改了Hashtable的结构，迭代器将抛出[ConcurrentModificationException](https://www.baeldung.com/java-concurrentmodificationexception)。此机制通过在检测到并发修改时快速失败来帮助防止不可预测的行为。

在下面的例子中，我们有一个包含三个键值对的Hashtable，并且我们启动两个线程：

- iteratorThread：遍历Hashtable键并以100毫秒的延迟打印它们
- modifierThread：等待50毫秒，然后向Hashtable添加新的键值对

**当modifierThread将新的键值对添加到Hashtable时，iteratorThread将引发ConcurrentModificationException，表示在迭代过程中Hashtable结构已被修改**：

```java
Hashtable<String, Integer> hashtable = new Hashtable<>();
hashtable.put("Key1", 1);
hashtable.put("Key2", 2);
hashtable.put("Key3", 3);
AtomicBoolean exceptionCaught = new AtomicBoolean(false);

Thread iteratorThread = new Thread(() -> {
    Iterator<String> it = hashtable.keySet().iterator();
    try {
        while (it.hasNext()) {
            it.next();
            Thread.sleep(100);
        }
    } catch (ConcurrentModificationException e) {
        exceptionCaught.set(true);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
});

Thread modifierThread = new Thread(() -> {
    try {
        Thread.sleep(50);
        hashtable.put("Key4", 4);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
});

iteratorThread.start();
modifierThread.start();

iteratorThread.join();
modifierThread.join();

assertTrue(exceptionCaught.get());
```

### 4.4 ConcurrentHashMap迭代器

与使用“快速失败”迭代器的Hashtable相比，ConcurrentHashMap采用“弱一致”迭代器。

这些迭代器可以承受对原始Map的并发修改，反映迭代器创建时Map的状态。它们可能还会反映进一步的更改，但不能保证这样做。因此，我们可以在一个线程中修改ConcurrentHashMap，在另一个线程中迭代它，而不会引发ConcurrentModificationException。

下面的例子演示了ConcurrentHashMap中迭代器的弱一致性：

- iteratorThread：遍历ConcurrentHashMap的键并以100毫秒的延迟打印它们
- modifierThread：等待50毫秒，然后向ConcurrentHashMap添加新的键值对

**与Hashtable“快速失败”迭代器不同，此处的弱一致性迭代器不会抛出ConcurrentModificationException**。iteratorThread中的迭代器继续运行而不会出现任何问题，这展示了ConcurrentHashMap是如何为高并发场景设计的：

```java
ConcurrentHashMap<String, Integer> concurrentHashMap = new ConcurrentHashMap<>();
concurrentHashMap.put("Key1", 1);
concurrentHashMap.put("Key2", 2);
concurrentHashMap.put("Key3", 3);
AtomicBoolean exceptionCaught = new AtomicBoolean(false);

Thread iteratorThread = new Thread(() -> {
    Iterator<String> it = concurrentHashMap.keySet().iterator();
    try {
        while (it.hasNext()) {
            it.next();
            Thread.sleep(100);
        }
    } catch (ConcurrentModificationException e) {
        exceptionCaught.set(true);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
});

Thread modifierThread = new Thread(() -> {
    try {
        Thread.sleep(50);
        concurrentHashMap.put("Key4", 4);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
});

iteratorThread.start();
modifierThread.start();

iteratorThread.join();
modifierThread.join();

assertFalse(exceptionCaught.get());
```

### 4.5 内存

Hashtable使用简单的数据结构，本质上是一个链表数组。此数组中的每个存储桶都存储一个键值对，因此只有数组本身和链表节点的开销，没有额外的内部数据结构来管理并发级别、加载因子或其他高级功能。因此，**Hashtable总体上消耗的内存较少**。

**ConcurrentHashMap更复杂，由一个段数组组成，本质上是一个单独的Hashtable**。这允许它并发执行某些操作，但也会为这些段对象消耗额外的内存。

对于每个段，它都会维护额外的信息，例如计数、阈值、负载因子等，这会增加其内存占用。它会动态调整段的数量及其大小以容纳更多条目并减少冲突，这意味着它必须保留额外的元数据来管理这些元数据，从而导致进一步的内存消耗。

## 5. 总结

在本文中，我们了解了Hashtable和ConcurrentHashMap之间的区别。

Hashtable和ConcurrentHashMap都用于以线程安全的方式存储键值对。但是，我们看到，由于ConcurrentHashMap具有高级同步功能，因此在性能和可扩展性方面通常更胜一筹。

Hashtable仍然有用，在旧系统或明确需要方法级同步的场景中可能更可取。了解应用程序的具体需求可以帮助我们在两者之间做出更明智的决定。