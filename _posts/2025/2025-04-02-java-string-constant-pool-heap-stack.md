---
layout: post
title: Java的字符串常量池位于哪里，堆还是栈？
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

每当我们声明一个变量或创建一个对象时，它都存储在内存中。从高层次上讲，Java将内存分为两个块：[栈和堆](https://www.baeldung.com/java-stack-heap)。**这两种内存都存储特定类型的数据，并且具有不同的存储和访问模式**。

在本教程中，我们将研究不同的参数并了解哪个区域最适合存储字符串常量池。

## 2. 字符串常量池

[字符串常量池](https://www.baeldung.com/jvm-constant-pool)是一块特殊的内存区域，**当我们声明一个String字面量时，[JVM](https://www.baeldung.com/jvm-parameters)在池中创建对象并将其引用存储在堆栈中**。在内存中创建每个String对象之前，JVM会执行一些步骤来减少内存开销。

字符串常量池在其实现中使用了一个[Hashmap](https://www.baeldung.com/java-hashmap)，Hashmap的每个桶都包含一个具有相同哈希码的String列表。在早期版本的Java中，池的存储区域是固定大小的，经常会导致“[Could not reserve enough space for object heap](https://www.baeldung.com/java-heap-memory-error)”错误。

**当系统加载类时，所有类的字符串字面量都进入应用程序级池，这是因为不同类的相等String字面量必须是相同的Object**。在这些情况下，池中的数据应该可供每个类使用，而没有任何依赖关系。

通常，堆栈存储的是短期数据。它包括本地原始变量、堆对象的引用和执行中的方法。堆允许动态内存分配，在运行时存储Java对象和JRE类。

堆允许全局访问，并且在应用程序的生命周期内，所有线程都可以访问堆中的数据存储，而栈中的数据存储具有私有范围，只有所有者线程可以访问它们。

栈将数据存储在连续的内存块中并允许随机访问，如果一个类需要池中的随机字符串，由于堆栈的LIFO(后进先出)规则，该字符串可能无法使用。相比之下，堆动态分配内存并允许我们以任何方式访问数据。

假设我们有一个由不同类型的变量组成的代码片段，栈将存储int字面量的值以及String和Demo对象的引用。任何对象的值都将存储在堆中，所有String字面量都进入堆内的池中：

![](/assets/images/2025/javajvm/javastringconstantpoolheapstack01.png)

线程执行完成后，栈上创建的变量会被立即释放。相反，[垃圾收集器](https://www.baeldung.com/jvm-garbage-collectors)回收堆中的资源。同样，垃圾收集器从池中收集未引用的项。

**池的默认大小在不同平台上可能不同**。无论如何，它仍然比可用堆栈大小大得多。在JDK 7之前，池是permgen空间的一部分，从JDK 7到现在，它是主堆内存的一部分。

## 3. 总结

在这篇简短的文章中，我们了解了字符串常量池的存储区域。栈和堆在存储和访问数据方面具有不同的特性，从内存分配到访问和可用性，堆是最适合存放字符串常量池的区域。

**事实上，池从来都不是栈内存的一部分**。