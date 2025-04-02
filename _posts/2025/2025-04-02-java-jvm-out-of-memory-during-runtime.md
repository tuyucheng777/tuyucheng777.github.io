---
layout: post
title: 当JVM在运行时分配的内存不足时会发生什么？
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

为JVM应用程序定义适当的堆大小是一个关键步骤，这可能有助于我们的应用程序进行内存分配和处理高负载。**但是，低效的堆大小(太小或太大)都可能会影响其性能**。

在本教程中，我们将了解[OutOfMemoryErrors](https://www.baeldung.com/java-gc-overhead-limit-exceeded)的原因及其与堆大小的关系。此外，我们将检查我们可以对此错误采取哪些措施以及如何调查根本原因。

## 2. –Xmx和–Xms

**我们可以使用两个专用的[JVM标志](https://www.baeldung.com/jvm-parameters#explicit-heap-memory---xms-and-xmx-options)来控制堆分配**。第一个-Xms帮助我们设置堆的初始大小和最小大小，另一个-Xmx设置最大堆大小。其他几个标志可以帮助更动态地分配，但它们总体上执行类似的工作。

让我们检查一下这些标志如何相互关联以及OutOfMemoryError如何导致或阻止它。首先，让我们澄清一个显而易见的事情：**-Xms不能大于-Xmx**。如果我们不遵循此规则，JVM将在启动时使应用程序失败：

```shell
$ java -Xms6g -Xmx4g
Error occurred during initialization of VM
Initial heap size set to a larger value than the maximum heap size
```

现在，让我们考虑一个更有趣的场景。**如果我们尝试分配比物理RAM更多的内存，会发生什么**？这取决于JVM版本、架构、操作系统等。某些操作系统(如Linux)允许[过度使用](https://www.baeldung.com/linux/overcommit-modes)并直接配置过度使用。其他操作允许过度使用，但在其内部启发式方法上这样做：

![](/assets/images/2024/javajvm/javajvmoutofmemoryduringruntime01.png)

同时，由于高度碎片化，即使我们有足够的物理内存，我们也可能无法启动应用程序。假设我们有4GB物理RAM，其中可用空间约为3GB。**分配2GB的堆可能是不可能的，因为RAM中没有这种大小的连续段**：

![](/assets/images/2024/javajvm/javajvmoutofmemoryduringruntime02.png)

某些版本的JVM，尤其是较新的版本，没有这样的要求。但是，它可能会影响运行时的对象分配。

## 3. 运行时OutOfMemoryError

假设我们启动应用程序时没有任何问题。由于多种原因，我们仍然有机会遇到OutOfMemoryError。

### 3.1 耗尽堆空间

内存消耗的增加可能是由自然原因引起的，例如，节日期间我们网上商店的活动增加。**此外，这也可能是由于[内存泄漏](https://www.baeldung.com/java-memory-leaks)而发生**。我们通常可以通过检查GC活动来区分这两种情况。同时，可能存在更复杂的情况，例如[完成延迟](https://www.baeldung.com/java-memory-leaks#5-through-finalize-methods)或垃圾回收线程速度慢。

### 3.2 过量使用

由于存在[交换空间](https://www.baeldung.com/cs/virtual-memory-vs-swap-space#introduction-to-swap-space)，过量使用是可能的。**我们可以通过将一些数据转储到光盘上来扩展RAM**，这可能会导致速度显著降低，但同时应用程序不会失败。但是，这可能不是解决此问题的最佳或理想的解决方案。此外，交换内存的极端情况是[抖动](https://www.baeldung.com/cs/virtual-memory-thrashing)，这可能会冻结系统。

我们可以将超额承诺视为部分准备金银行业务。RAM没有向应用程序承诺的所有所需内存。但是，当应用程序开始索取其承诺的内存时，操作系统可能会开始[杀死](https://www.baeldung.com/linux/memory-overcommitment-oom-killer)不重要的应用程序，以确保其余应用程序不会失败：

![](/assets/images/2024/javajvm/javajvmoutofmemoryduringruntime03.png)

### 3.3 收缩堆

此问题与过度使用有关，但罪魁祸首是试图[最小化占用空间](https://www.baeldung.com/gc-release-memory)的垃圾收集启发式方法。**即使应用程序在生命周期的某个时刻成功声明了最大堆大小，也不意味着下次就能获得它**。

垃圾收集器可能会从堆中返回一些未使用的内存，操作系统可以将其重新用于不同的目的。**同时，当应用程序尝试取回它时，RAM可能已经分配给其他某个应用程序**。

我们可以通过将-Xms和-Xmx设置为相同的值来控制它，这样，我们可以获得更可预测的内存消耗并避免堆收缩。但是，这可能会影响资源利用率；因此，应谨慎使用。**此外，不同的JVM版本和垃圾收集器在堆收缩方面的行为可能有所不同**。

## 4. OutOfMemoryError

并非所有OutOfMemoryErrors都是相同的。我们有很多口味，了解它们之间的差异可能有助于我们找出根本原因。我们只会考虑那些与前面描述的场景相关的内容。

### 4.1 Java堆空间

我们可以在日志中看到以下消息：java.lang.OutOfMemoryError: Java heap space。这清楚地描述了问题：堆中没有空间。造成这种情况的原因可能是内存泄漏或应用程序负载增加，创建率和删除率的显著差异也可能导致此问题。

### 4.2 GC开销超出限制

有时，应用程序可能会失败，并显示：[java.lang.OutOfMemoryError: GC Overhead limit exceeded](https://www.baeldung.com/java-gc-overhead-limit-exceeded)。**当应用程序将98%的时间用于垃圾收集时，就会发生这种情况，这意味着吞吐量仅为2%**。这种情况描述了垃圾收集抖动：应用程序处于活动状态，但没有有用的工作。

### 4.3 交换空间不足

另一种类型的OutOfMemoryError是：java.lang.OutOfMemoryError: request size bytes for reason. Out of swap space？**这通常是操作系统端过度使用的指标**。在这种情况下，我们堆中仍然有容量，但操作系统无法为我们提供更多内存。

## 5. 根本原因

当我们遇到OutOfMemoryError时，我们在应用程序中几乎无能为力。尽管不建议捕获错误，但在某些情况下，出于清理或记录目的可能是合理的。有时，我们可以看到处理try-catch块来处理条件逻辑的代码。**这是一种相当昂贵且不可靠的黑客行为，在大多数情况下应避免使用**。

### 5.1 垃圾收集日志

虽然OutOfMemoryError提供了有关问题的信息，但不足以进行更深入的分析。最简单的方法是使用[垃圾收集日志](https://www.baeldung.com/java-verbose-gc)，它不会产生太多开销，同时提供有关正在运行的应用程序的基本信息。

### 5.2 堆转储

堆转储是浏览应用程序的另一种方式，虽然我们可以定期捕获它，但这可能会影响应用程序的性能。最便宜的使用方法是在OutOfMemoryError上自动执行堆转储。幸运的是，JVM允许我们使用-XX:+HeapDumpOnOutOfMemoryError进行设置。此外，我们还可以使用-XX:HeapDumpPath标志设置堆转储的路径。

### 5.3 在发生OutOfMemoryError时运行脚本

为了增强OutOfMemoryError的体验，我们可以使用-XX:OnOutOfMemoryError并将其定向到应用程序内存不足时将运行的[脚本](https://www.baeldung.com/jvm-parameters#handling-out-of-memory)。这可用于实现通知系统、将堆转储发送到某些分析工具或重新启动应用程序。

## 6. 总结

在本文中，我们讨论了OutOfMemoryError，它表示应用程序外部的问题，就像其他错误一样。处理这些错误可能会产生更多问题，并使我们的应用程序不一致，因此处理这种情况的最佳方法是从一开始就防止它发生。

仔细的内存管理和JVM配置可以帮助我们解决这个问题。此外，分析垃圾收集日志可以帮助我们找出问题的原因。在不了解根本问题的情况下向应用程序分配更多内存或使用其他技术来确保其保持活动状态并不是正确的解决方案，并且可能会导致更多问题。