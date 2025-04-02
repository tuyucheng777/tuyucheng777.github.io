---
layout: post
title:  JVM中的压缩OOP
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

JVM为我们管理内存，这减轻了开发人员的内存管理负担，因此我们不需要手动操作对象指针，这已被证明是耗时且容易出错的。

在幕后，JVM结合了许多巧妙的技巧来优化内存管理过程。**其中一个技巧是使用压缩指针**，我们将在本文中对其进行评估。首先，让我们看看JVM在运行时如何表示对象。

## 2. 运行时对象表示

HotSpot JVM使用称为[oop](https://www.baeldung.com/java-memory-layout)或普通对象指针的数据结构来表示对象。这些oops等同于原生C指针，**[instanceOop](https://github.com/openjdk/jdk15/blob/master/src/hotspot/share/oops/instanceOop.hpp)是一种特殊的oop，它表示Java中的对象实例**。此外，JVM还支持保存在[OpenJDK源代码树](http://hg.openjdk.java.net/jdk8/jdk8/hotspot/file/87ee5ee27509/src/share/vm/oops/)中的一些其他oops。

让我们看看JVM如何在内存中布局instanceOop。

### 2.1 对象内存布局

instanceOop的内存布局很简单：它只是对象头，后面紧跟着0个或多个对实例字段的引用。

对象头的JVM表示包括：

-   **一个标记词**有多种用途，例如偏向锁定、身份哈希值和GC。它不是oop，但由于历史原因，它位于[OpenJDK的oop](http://hg.openjdk.java.net/jdk8/jdk8/hotspot/file/87ee5ee27509/src/share/vm/oops/markOop.hpp)源代码树中。此外，标记字状态仅包含一个[uintptr_t](https://github.com/openjdk/jdk15/blob/e208d9aa1f185c11734a07db399bab0be77ef15f/src/hotspot/share/oops/markWord.hpp#L96)，因此，**其大小在32位和64位架构中分别在4和8字节之间变化**。
-   **一个可能是压缩的Klass字**，表示指向类元数据的指针。在Java 7之前，它们指向[永久代](https://www.baeldung.com/native-memory-tracking-in-jvm)，但从Java 8开始，它们指向[元空间](https://www.baeldung.com/native-memory-tracking-in-jvm)。
-   **强制对象对齐的32位间隙**，这使得布局对硬件更加友好，我们稍后会看到。

**在标头之后，将有0个或多个对实例字段的引用**。在这种情况下，一个字是一个本地机器字，因此在传统的32位机器上是32位，在更现代的系统上是64位。

数组的对象头，除了mark和klass字之外，还有一个[32位字](https://github.com/openjdk/jdk15/blob/e208d9aa1f185c11734a07db399bab0be77ef15f/src/hotspot/share/oops/arrayOop.hpp#L35)来表示它的长度。

### 2.2 废物剖析

假设我们要从传统的32位架构切换到更现代的64位机器，起初，我们可能希望立即获得性能提升。但是，当涉及到JVM时，情况并非总是如此。

**造成这种性能下降的主要原因是64位对象引用**，64位引用占用的空间是32位引用的两倍，因此这通常会导致更多的内存消耗和更频繁的GC周期。专用于GC周期的时间越多，应用程序线程的CPU执行片就越少。

那么，我们是否应该切换回去并再次使用那些32位架构？即使这是一个选项，如果不做更多的工作，我们也不可能在32位进程空间中拥有超过4GB的堆空间。

## 3. 压缩OOP

事实证明，JVM可以通过压缩对象指针或oops来避免内存浪费，因此我们可以两全其美：**在64位机器中允许超过4GB的堆空间和32位引用**。

### 3.1 基本优化

正如我们之前看到的，JVM向对象添加了填充，以便它们的大小是8字节的倍数。**有了这些填充，oops中的最后3位始终为0**，这是因为8的倍数在二进制中总是以000结尾。

![](/assets/images/2025/javajvm/jvmcompressedoops01.png)

由于JVM已经知道最后3位始终为0，因此将这些无关紧要的0存储在堆中毫无意义。相反，它假设它们在那里并存储3个我们以前无法放入32位的更重要的位。现在，我们有一个带有3个右移0的32位地址，因此我们将一个35位指针压缩为一个32位指针。**这意味着我们最多可以使用32GB(2<sup>32+3</sup> = 2<sup>35</sup> = 32GB)的堆空间，而无需使用64位引用**。

为了使这种优化发挥作用，当JVM需要在内存中找到一个对象时，它会**将指针向左移动3位**(基本上是将那些3个0加回到末尾)。另一方面，当加载指向堆的指针时，JVM将指针向右移动3位以丢弃那些先前添加的0。**基本上，JVM会执行更多的计算以节省一些空间**。幸运的是，对于大多数CPU来说，位移是一项非常微不足道的操作。

要启用oop压缩，我们可以使用-XX:+UseCompressedOops调整标志。oop压缩是从Java 7开始的默认行为，只要最大堆大小小于32GB。**当最大堆大小超过32GB时，JVM将自动关闭oop压缩**。因此，需要以不同方式管理超过32GB堆大小的内存利用率。

### 3.2 超过32GB

当Java堆大小大于32GB时，也可以使用压缩指针。**尽管默认的对象对齐方式是8字节，但可以使用-XX:ObjectAlignmentInBytes调整标志配置该值，指定的值应该是2的幂并且必须在8和256的范围内**。

我们可以使用压缩指针计算最大可能的堆大小，如下所示：

```text
4 GB * ObjectAlignmentInBytes
```

例如，当对象对齐为16字节时，我们最多可以使用64GB的堆空间和压缩指针。

请注意，随着对齐值的增加，对象之间未使用的空间也可能增加。因此，我们可能无法从使用具有较大Java堆大小的压缩指针中获益。

### 3.3 未来的GC

[ZGC](https://www.baeldung.com/jvm-zgc-garbage-collector)是[Java 11](https://openjdk.java.net/jeps/333)中的新增功能，它是一种实验性和可扩展的低延迟垃圾回收器。

它可以处理不同范围的堆大小，同时将GC暂停时间保持在10毫秒以下。由于ZGC需要使用[64位彩色指针](https://youtu.be/kF_r3GE3zOo?t=643)，因此**它不支持压缩引用**。因此，使用像ZGC这样的超低延迟GC必须与使用更多内存进行权衡。

从Java 15开始，ZGC支持压缩类指针，但仍然缺乏对压缩OOP的支持。

但是，所有新的GC算法都不会为了低延迟而牺牲内存。例如，[Shenandoah GC](https://openjdk.java.net/jeps/379)除了具有较低的暂停时间外，还支持压缩引用。

此外，Shenandoah和ZGC都已在[Java 15](https://openjdk.java.net/projects/jdk/15/)中完成。

## 4. 总结

在本文中，我们描述了64位架构中的JVM内存管理问题。我们研究了压缩指针和对象对齐，并且了解了JVM如何解决这些问题，从而允许我们使用更大的堆大小、更少浪费的指针和最少的额外计算。

有关压缩引用的更详细讨论，强烈建议查看[Aleksey Shipilëv](https://shipilev.net/jvm/anatomy-quarks/23-compressed-references/)的另一篇精彩文章。另外，要了解对象分配在HotSpot JVM中是如何工作的，请查看[Java中对象的内存布局](https://www.baeldung.com/java-memory-layout)一文。