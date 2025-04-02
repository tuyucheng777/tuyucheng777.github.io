---
layout: post
title:  JVM中的实验性垃圾收集器
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

在本教程中，我们将介绍[Java内存管理](https://www.baeldung.com/jvm-garbage-collectors)的基本问题以及不断寻找更好的方法来实现它的需要。这将主要介绍Java中引入的名为Shenandoah的新实验性垃圾回收器，以及它与其他垃圾回收器的比较。

## 2. 了解垃圾回收中的挑战

垃圾回收器是一种自动内存管理形式，其中像JVM这样的运行时管理在其上运行的用户程序的内存分配和回收。有几种算法可以实现垃圾回收器，这些包括引用计数、标记-清除、标记-压缩和复制。

### 2.1 垃圾回收器的注意事项

根据我们用于垃圾回收的算法，**它可以在用户程序挂起时运行，也可以与用户程序并发运行**。前者以长时间暂停(也称为Stop-the-world暂停)导致的高延迟为代价实现了更高的吞吐量。后者旨在实现更好的延迟，但会牺牲吞吐量。

事实上，大多数现代收集器都使用混合策略，他们同时应用Stop-the-world和并发方法。它通常**通过将堆空间划分为年轻代和老年代来工作**。然后，分代收集器在年轻代中使用Stop-the-world收集，在老代中使用并发收集，可能以增量方式减少暂停。

尽管如此，**最佳点实际上是找到一个运行时停顿最少、吞吐量高的垃圾收集器**-所有这些都具有可预测的堆大小行为，可以从小到大不等。这是一场持续不断的斗争，它从早期就一直保持着Java垃圾回收创新的步伐。

### 2.2 Java中现有的垃圾回收器

**一些传统的[垃圾回收器](https://www.baeldung.com/jvm-garbage-collectors)包括串行和并行收集器，他们是分代收集器**，在年轻代中使用复制，在老年代中使用标记压缩：

![](/assets/images/2025/javajvm/jvmexperimentalgarbagecollectors01.png)

虽然它们提供了良好的吞吐量，但它们却**存在长时间Stop-the-world暂停的问题**。

Java 1.4中引入的[Concurrent Mark Sweep(CMS)收集器](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/cms.html)是一种分代、并发、低暂停收集器，它适用于年轻代中的复制和老年代中的标记清除：

![](/assets/images/2025/javajvm/jvmexperimentalgarbagecollectors02.png)

它试图通过与用户程序同时完成大部分工作来最小化暂停时间，尽管如此，**它仍然存在导致不可预测的暂停的问题**，需要更多的CPU时间，并且不适合大于4GB的堆。

作为CMS的长期替代品，[G1回收器](https://docs.oracle.com/javase/9/gctuning/garbage-first-garbage-collector.htm)在Java 7中被引入。G1是一个分代、并行、并发、增量压缩的低暂停收集器，它适用于年轻代中的复制和老年代中的标记压缩：

![](/assets/images/2025/javajvm/jvmexperimentalgarbagecollectors03.png)

然而，G1也是一个区域化的收集器，并将堆区域划分为更小的区域。这给它带来了更可预测的暂停的好处，G1针对具有大量内存的多处理器机器，因此也无法避免暂停。

因此，寻找更好的垃圾回收器的旅程仍在继续，尤其是能够进一步减少暂停时间的垃圾收集器。JVM最近推出了一系列实验性收集器，如Z、Epsilon和Shenandoah。除此之外，G1继续得到更多改进。

目标实际上是尽可能接近无暂停的Java。

## 3. Shenandoah垃圾回收器

**[Shenandoah](https://wiki.openjdk.java.net/display/shenandoah/Main)是Java 12中引入的实验性收集器，定位为延迟专家**，它试图通过与用户程序同时执行更多的垃圾回收工作来减少暂停时间。

例如，Shenandoah尝试同时执行对象重定位和压缩，这实质上意味着Shenandoah中的暂停时间不再与堆大小成正比。因此，无论堆大小如何，它都可以**提供一致的低暂停行为**。

### 3.1 堆结构

Shenandoah和G1一样，是一个区域化的收集器。这意味着**它将堆区域划分为大小相等的区域集合**，区域基本上是内存分配或回收的单位：

![](/assets/images/2025/javajvm/jvmexperimentalgarbagecollectors04.png)

但是，与G1和其他分代收集器不同，Shenandoah不将堆区域划分为分代。因此，它必须在每个周期标记大部分存活对象，而分代收集器可以避免这种情况。

### 3.2 对象布局

在Java中，内存中的对象不仅包括数据字段-它们还携带一些额外的信息。这些额外信息包括标头(包含指向对象类的指针)和标记字。标记字有多种用途，如转发指针、年龄位、锁和哈希：

![](/assets/images/2025/javajvm/jvmexperimentalgarbagecollectors05.png)

**Shenandoah为该对象布局添加了一个额外的字**，这用作间接指针并允许Shenandoah移动对象而无需更新对它们的所有引用，这也**称为Brooks指针**。

### 3.3 屏障

在Stop-the-world模式下执行收集周期更简单，但是当我们与用户程序同时执行时，复杂性就会飙升。它对收集阶段(如并发标记和压缩)提出了不同的挑战。

解决方案在于**通过我们所说的屏障拦截所有堆访问**，Shenandoah和G1等其他并发收集器使用屏障来确保堆的一致性。然而，障碍是昂贵的操作并且通常会降低收集器的吞吐量。

例如，对象的读写操作可能会被收集器使用屏障拦截：

![](/assets/images/2025/javajvm/jvmexperimentalgarbagecollectors06.png)

**Shenandoah在不同阶段使用多个屏障，如SATB屏障、读屏障和写屏障**，我们将在后面的部分中看到这些屏障的用途。

### 3.4 模式、启发式和故障模式

**模式定义了Shenandoah运行的方式**，比如它使用的屏障，它们还定义了它的性能特征。有三种模式可用：normal/SATB、iu和passive，normal/SATB模式是默认模式。

**启发式确定收集应何时开始以及应包括哪些区域**，这些包括自适应、静态、紧凑和激进，其中自适应作为默认启发式。例如，它可能会选择具有60%或更多垃圾的区域，并在75%的区域已分配时开始收集周期。

Shenandoah需要比分配它的用户程序更快地收集堆，但是，**有时它可能会落后，从而导致故障模式之一**。这些故障模式包括步调、退化收集，以及在最坏情况下的完整收集。

## 4. Shenandoah收集阶段

Shenandoah的收集周期主要包括三个阶段：标记、疏散和更新引用。尽管这些阶段中的大部分工作与用户程序同时发生，但仍有一小部分必须在Stop-the-world模式下发生。

### 4.1 标记

**标记是识别堆中所有或部分不可达对象的过程**，我们可以通过从根对象开始并遍历对象图来找到可达对象来做到这一点。在遍历过程中，我们还为每个对象分配3种颜色之一：白色、灰色或黑色：

![](/assets/images/2025/javajvm/jvmexperimentalgarbagecollectors07.png)

在Stop-the-world模式下标记更简单，但在并发模式下会变得复杂，这是因为用户程序在标记过程中并发地改变对象图。Shenandoah通过**使用开始时快照(SATB)算法解决了这个问题**。

这意味着，任何在标记开始时处于活动状态的对象或自标记开始以来已分配的对象都被视为存活对象，Shenandoah使用SATB屏障来维护堆的SATB视图。

虽然**大多数标记都是并发完成的**，但仍有一些部分是在Stop-the-world模式下完成的。在Stop-the-world模式下发生的部分是扫描根集的init-mark和清空所有待处理队列并重新扫描根集的final-mark，final-mark还准备指示要疏散区域的集合集。

### 4.2 清理和疏散

一旦标记完成，垃圾区域就可以被回收了。**垃圾区域是不存在活动对象的区域**，清理工作是同时进行的。

现在，下一步是将集合集中的活跃对象移动到其他区域。这样做是为了减少内存分配中的碎片，因此也称为压缩。疏散或压缩完全同时发生。

现在，这就是Shenandoah不同于其他回收器的地方。对象的并发重定位是棘手的，因为用户程序会继续读取和写入它们。**Shenandoah通过对对象的Brooks指针执行比较和交换操作来指向其目标空间版本**，从而成功实现了这一点：

![](/assets/images/2025/javajvm/jvmexperimentalgarbagecollectors08.png)

此外，**Shenandoah使用读写屏障来确保在并发疏散期间保持严格的“to-space”不变量**，这意味着读写必须从保证在撤离过程中幸存下来的目标空间进行。

### 4.3 引用更新

**回收周期中的这个阶段是遍历堆并更新对在疏散期间移动的对象的引用**：

![](/assets/images/2025/javajvm/jvmexperimentalgarbagecollectors09.png)

同样，更新引用阶段大部分是同时完成的。有短暂的init-update-refs周期用于初始化更新引用阶段，还有final-update-refs周期用于重新更新根集并回收集合中的区域，只有这些才需要Stop-the-world模式。

## 5. 与其他实验性回收器的比较

Shenandoah并不是最近在Java中引入的唯一实验性垃圾回收器，其他包括Z和Epsilon，让我们了解一下它们与Shenandoah的比较。

### 5.1 ZGC

**[ZGC](https://www.baeldung.com/jvm-zgc-garbage-collector)在Java 11中引入，它是一种单代、低延迟的收集器，专为非常大的堆大小而设计-这里的大我们指TB级别的堆**。ZGC与用户程序并发完成大部分工作，并利用堆引用的加载屏障。

此外，ZGC通过称为指针着色的技术利用64位指针。在这里，彩色指针存储有关堆上对象的额外信息，ZGC使用存储在指针中的额外信息重新映射对象以减少内存碎片。

总体而言，**ZGC的目标与Shenandoah的目标相似**，它们都旨在实现与堆大小不成正比的低暂停时间。但是，**与ZGC相比，Shenandoah有更多可用的调优选项**。

### 5.2 Epsilon

同样在Java 11中引入的[Epsilon](https://www.baeldung.com/jvm-epsilon-gc-garbage-collector)具有非常不同的垃圾回收方法，它基本上是一个被动或“无操作”收集器，这意味着它处理内存分配但不回收它。因此，当堆内存用完时，JVM会简单地关闭。

但是我们为什么要使用这样的收集器呢？基本上，任何垃圾回收器都会间接影响用户程序的性能，很难对应用程序进行基准测试并了解垃圾回收对其的影响。

Epsilon正是为此而生的，它只是消除了垃圾回收器的影响，让我们可以独立运行应用程序。但这要求我们对应用程序的内存需求有非常清晰的了解，因此，我们可以从应用程序中获得更好的性能。

显然，**Epsilon的目标与Shenandoah的目标截然不同**。

## 6. 总结

在本文中，我们介绍了Java中垃圾回收的基础知识以及不断改进它的必要性。我们详细讨论了Java中引入的最新实验性收集器Shenandoah，我们还讨论了它与Java中可用的其他实验性收集器的对比情况。

通用垃圾收集器的追求不会很快实现，因此，虽然G1仍然是默认回收器，但这些新增功能为我们提供了在低延迟情况下使用Java的选项。但是，我们不应该将它们视为其他高吞吐量回收器的直接替代品。