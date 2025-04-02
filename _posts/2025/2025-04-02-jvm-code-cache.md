---
layout: post
title:  JVM代码缓存简介
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

在本教程中，我们将快速浏览和了解JVM的代码[缓存内存](https://www.baeldung.com/cs/cache-memory)。

## 2. 什么是代码缓存？

简单的说，**JVM代码缓存就是JVM存放自己编译成本机代码的字节码的区域**。我们将可执行本机代码的每个块称为nmethod，nmethod可能是完整的或内联的Java方法。 

即时(JIT)编译器是代码缓存区的最大消费者，这就是为什么一些开发人员将此内存称为JIT代码缓存的原因。

## 3. 代码缓存调优 

**代码缓存具有固定大小**，一旦它已满，JVM将不会编译任何额外的代码，因为JIT编译器现在已关闭。此外，我们会收到“CodeCache is full... The compiler has been disabled”的警告消息。结果，我们的应用程序性能会下降。为避免这种情况，我们可以使用以下大小选项调整代码缓存：

-   InitialCodeCacheSize：初始代码缓存大小，默认160K
-   **ReservedCodeCacheSize：默认最大大小为48MB**
-   CodeCacheExpansionSize：代码缓存的扩展大小，32KB或64KB

增加ReservedCodeCacheSize可能是一种解决方案，但这通常只是一种临时解决方法。

幸运的是，**JVM提供了一个UseCodeCacheFlushing选项来控制代码缓存区的刷新**。它的默认值为false，当我们启用它时，**它会在满足以下条件时释放占用的区域**：

-   代码缓存已满；**如果该区域的大小超过某个阈值，则该区域将被刷新**
-   自上次清理以来经过了特定时间间隔
-   预编译代码不够热；对于每个编译的方法，JVM都会跟踪一个特殊的热度计数器，如果这个计数器的值小于计算的阈值，JVM会释放这段预编译代码

## 4. 代码缓存使用

为了监控代码缓存的使用情况，我们需要跟踪当前使用的内存大小。

**要获取有关代码缓存使用情况的信息，我们可以指定–XX:+PrintCodeCacheJVM选项**。运行我们的应用程序后，我们将看到类似的输出：

```text
CodeCache: size=32768Kb used=542Kb max_used=542Kb free=32226Kb
```

让我们看看每个值的含义：

-   输出中的size表示内存的最大大小，与ReservedCodeCacheSize相同
-   used是当前正在使用的内存的实际大小
-   max_used是已经使用的最大大小
-   free是尚未被占用的剩余内存

PrintCodeCache选项非常有用，我们可以：

-   查看何时发生冲洗
-   确定是否达到临界内存使用点

## 5. 分段代码缓存

**从[Java 9](https://openjdk.java.net/jeps/197)开始，JVM将代码缓存分为3个不同的段**，每个段包含一种特定类型的编译代码：

-   非方法段包含JVM内部相关代码，如字节码解释器。默认情况下，该段约为5MB。此外，可以通过-XX:NonNMethodCodeHeapSize调整标志配置段大小
-   分析代码段包含轻度优化的代码，生命周期可能很短。尽管默认情况下段大小约为122MB，但我们可以通过-XX:ProfiledCodeHeapSize调整标志更改它
-   非分析段包含完全优化的代码，可能具有较长的生命周期。同样，默认情况下约为122MB。当然，这个值可以通过-XX:NonProfiledCodeHeapSize调整标志进行配置

**这种新结构对不同类型的编译代码进行不同的处理，从而提高整体性能**。

例如，将短暂的编译代码与长寿命的代码分开可以提高方法清除器的性能-主要是因为它需要扫描更小的内存区域。

## 6. 总结

这篇简短的文章对JVM代码缓存进行了简单的介绍。

此外，我们还提供了一些使用和调整选项来监视和诊断此内存区域。