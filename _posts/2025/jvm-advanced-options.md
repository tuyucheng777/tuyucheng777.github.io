---
layout: post
title:  探索高级JVM选项
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

Java虚拟机(JVM)是驱动Java应用程序的强大引擎，它高度可定制，通过标准选项提供基本配置，通过非标准选项提供常规性能调优，并通过高级选项提供精确控制。

高级选项允许开发人员微调性能、诊断问题并试验尖端功能。

**在本教程中，我们将探讨最著名的高级JVM选项及其使用方法**，从而让我们能够更细粒度地控制JVM行为。

## 2. JVM选项的分类

JVM参数可以分为三类：

- **标准选项**(-version、-help)
- **非标准选项**(-X:option)
- **高级选项**(-XX:option)

## 3. 了解高级JVM选项

高级选项超越了基本配置，可以设置JVM的底层属性。**这些选项允许我们调整性能关键参数，例如垃圾回收、内存管理和运行时诊断**。

其中一些高级选项也是最常用的[重要JVM参数之一](https://www.baeldung.com/jvm-parameters)，了解如何[在IntelliJ中设置这些JVM选项](https://www.baeldung.com/intellij-idea-set-jvm-arguments)，对于在开发和调试过程中调优JVM大有裨益。

**然而，由于它们针对特定的应用场景进行了微调，我们必须谨慎使用它们**。在没有清晰理解应用程序行为的情况下进行过度定制可能会导致性能不佳、崩溃或意外行为。

**此外，高级JVM选项并非保证所有JVM实现都支持，并且可能会发生变化**。因此，由于这些选项会随着JVM更新而演变，某些选项可能会在新版本中被弃用或行为发生变化。

例如，Java并发标记和清除垃圾回收算法就曾出现过这种情况，该算法在Java 9中已弃用，并[在Java 14中被移除](https://openjdk.org/jeps/363)，通过关注文档，我们可以在任何变更发生之前及时了解情况。

现在让我们通过不同的类别探索各种高级JVM选项。

## 4. 垃圾回收调优

垃圾回收对于[内存管理](https://www.baeldung.com/gc-release-memory)至关重要，但可能会造成影响性能的暂停。**高级选项可控制垃圾回收行为，确保应用程序运行更顺畅**。

自Java 9以来，默认设置是G1，旨在平衡吞吐量和延迟。

为了克服G1的延迟限制，JDK 12引入了[Shenandoah GC](https://www.baeldung.com/jvm-experimental-garbage-collectors)，可以通过
**-XX:+UseShenandoahGC**选项启用，Shenandoah的[可用性](https://wiki.openjdk.org/display/shenandoah/Main)取决于JDK供应商和版本。

可以根据具体工作负载使用[其他实现](https://www.baeldung.com/jvm-garbage-collectors)，[Epsilon垃圾回收器](https://www.baeldung.com/jvm-epsilon-gc-garbage-collector)也非常适合性能调优，可以检查垃圾回收是否影响程序性能。

前面引用的主题探讨了用于垃圾回收的各种有用的高级JVM选项。

## 5. 内存管理

正如我们上面讨论的，垃圾回收是内存管理的重要组成部分，但它只是JVM中更大的内存管理生态系统的一部分。

**为了实现最佳性能，配置内存分配、管理堆大小以及了解堆外内存的工作原理同样重要**，特别是对于内存密集型应用程序或具有特定性能要求的系统。

现在，让我们回顾一下与内存管理相关的一些高级JVM选项：

- **-XX:InitialHeapSize**和**-XX:MaxHeapSize**：这些选项定义初始和最大堆大小(以字节为单位)。
- **-XX:MetaspaceSize**和**-XX:MaxMetaspaceSize**：这些选项定义元空间区域的初始大小和最大大小。
- **-XX:InitialRAMPercentage**和**-XX:MaxRAMPercentage**：这些选项将初始和最大堆大小定义为系统可用内存的百分比，这些设置允许JVM动态扩展其内存使用量，从而提供更好的适应性。
- **-XX:MinHeapFreeRatio**和**-XX:MaxHeapFreeRatio**：这些选项定义GC周期后堆中保留的可用空间的最小和最大百分比。
- **-XX:+AlwaysPreTouch**：通过在JVM初始化期间预触碰Java堆来减少延迟，因此，每个堆页面都会在JVM启动时初始化，而不是在应用程序执行期间逐步初始化。
- **-XX:MaxDirectMemorySize**：定义可为直接字节缓冲区保留的内存量。
- **-XX:CompressedClassSpaceSize**：定义使用压缩类指针(-XX:-UseCompressedClassPointers)时在元空间中存储类元数据所分配的最大内存。

## 6. 即时编译

[即时(JIT)编译](https://www.baeldung.com/graal-java-jit-compiler)是JVM的一个关键组件，它可以在运行时将字节码编译为本机机器码，从而提高Java应用程序的性能。JIT编译器默认启用，除非为了调查JIT编译问题，否则不建议禁用它。

让我们回顾一下用于配置和调整JIT编译的高级JVM选项：

- **-XX:CICompilerCount**：定义用于JIT编译的编译器线程数，默认值与可用CPU和内存数量成正比。
- **-XX:ReservedCodeCacheSize**：定义用于存储JIT编译的本机代码的内存区域的最大大小。
- **-XX:CompileThreshold**：定义方法首次编译之前的方法调用次数。
- **-XX:MaxInlineSize**：定义JIT编译器可以内联的方法的最大允许大小(以字节为单位)。

## 7. 诊断和调试

**诊断和调试对于识别和解决Java应用程序中的问题(例如性能瓶颈、内存泄漏和意外行为)至关重要**。

让我们回顾一下与诊断和调试相关的高级JVM选项，这些选项可以帮助我们更深入地了解应用程序的行为和性能：

- **-XX:+HeapDumpOnOutOfMemoryError**：发生OutOfMemoryError时生成堆转储
- **-XX:HeapDumpPath**：定义保存堆转储的文件路径
- **-XX:+PrintCompilation**：记录JIT编译
- **-XX:+LogCompilation**：将详细的JIT编译日志写入文件
- **-XX:+UnlockDiagnosticVMOptions**：解锁默认情况下不可用的诊断JVM选项
- **-XX:+ExitOnOutOfMemoryError**：强制JVM在遇到OutOfMemoryError时立即退出
- **-XX:+CrashOnOutOfMemoryError**：强制JVM生成核心转储并在发生OutOfMemoryError时崩溃
- **-XX:ErrorFile**：定义发生不可恢复的错误时保存错误数据的文件路径
- **-XX:NativeMemoryTracking**：定义跟踪JVM本机内存使用情况的模式(关闭/摘要/详细)
- **-XX:+PrintNMTStatistics**：在JVM退出时启用打印收集的本机内存跟踪数据，此功能仅在启用本机内存跟踪(-XX:NativeMemoryTracking)时使用。

用于记录GC信息的高级JVM选项-XX:+PrintGC和-XX:+PrintGCDetails自Java 9以来已被弃用，应替换为统一日志记录选项-Xlog。

## 8. 人体工程学

我们探索了许多强大的高级JVM选项，这些选项可能会让我们在配置和调优JVM以满足特定需求时感到不知所措，**JVM通过人体工程学设计提供了一种解决方案，它可以根据底层硬件和运行时条件自动调整其行为，从而提升应用程序的性能**。

我们列出环境中的所有人体工程学默认值：

```shell
java -XX:+PrintFlagsFinal -version
```

然而，尽管人体工程学旨在设置合理的默认值，但它们可能并不总是符合我们应用程序的需求。

## 9. 总结

在本文中，我们全面探讨了高级JVM选项，结合了本文引用的讨论和所提供的分析的见解。

我们观察了高级JVM参数以及它们如何增强垃圾回收、内存管理和运行时性能。虽然配置范围可能让人感到不知所措，但我们提到JVM人体工程学是一个有用的解决方案，它可以简化调优过程并优化应用程序性能。