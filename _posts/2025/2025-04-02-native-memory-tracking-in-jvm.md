---
layout: post
title:  JVM中的本机内存跟踪
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

有没有想过为什么Java应用程序消耗的内存比通过众所周知的-Xms和-Xmx调整标志指定的数量多得多？由于各种原因和可能的优化，JVM可能会分配额外的本机内存，这些额外的分配最终会使消耗的内存超出-Xmx限制。

在本教程中，我们将列举JVM中本机内存分配的一些常见来源及其大小调整标志，然后学习如何使用本机内存跟踪来监视它们。

## 2. 本机分配

堆通常是Java应用程序中最大的内存消耗者，但还有其他消耗者。**除了堆之外，JVM还从本机内存中分配相当大的块来维护其类元数据、应用程序代码、JIT生成的代码、内部数据结构等**。在接下来的部分中，我们将探讨其中的一些分配。

### 2.1 元空间

**为了维护已加载类的一些元数据，JVM使用了一个专用的非堆区域，称为Metaspace**。在Java 8之前，等效项称为PermGen或Permanent Generation。Metaspace或PermGen包含有关已加载类的元数据，而不是它们的实例，后者保存在堆内。

这里重要的是堆大小配置不会影响元空间大小，**因为元空间是堆外数据区域**。为了限制元空间大小，我们使用其他调整标志：

-    -XX:MetaspaceSize和-XX:MaxMetaspaceSize设置最小和最大元空间大小
-   在Java 8之前，-XX:PermSize和-XX:MaxPermSize设置最小和最大PermGen大小

### 2.2 线程

JVM中最消耗内存的数据区域之一是栈，它与每个线程同时创建。栈存储局部变量和部分结果，在方法调用中起着重要作用。

默认线程栈大小取决于平台，但在大多数现代64位操作系统中，它约为1MB。此大小可通过-Xss调整标志进行配置。

与其他数据区域相比，**当线程数没有限制时，分配给栈的总内存实际上是无限的**。还值得一提的是，JVM本身需要一些线程来执行其内部操作，如GC或即时编译。

### 2.3 代码缓存

为了在不同的平台上运行JVM字节码，需要将其转换为机器指令，JIT编译器在程序执行时负责此编译。

**当JVM将字节码编译为汇编指令时，它会将这些指令存储在称为代码缓存的特殊非堆数据区域中**。代码缓存可以像JVM中的其他数据区域一样进行管理，-XX:InitialCodeCacheSize和-XX:ReservedCodeCacheSize调整标志确定代码缓存的初始大小和最大可能大小。

### 2.4 垃圾回收

JVM附带了一些GC算法，每个算法都适用于不同的用例。所有这些GC算法都有一个共同特征：它们需要使用一些堆外数据结构来执行它们的任务，这些内部数据结构消耗更多的本机内存。

### 2.5 符号

让我们从字符串开始，它是应用程序和库代码中最常用的数据类型之一。由于它们无处不在，因此它们通常占据堆的很大一部分。如果大量这些字符串包含相同的内容，那么堆的很大一部分将被浪费。

为了节省一些堆空间，我们可以为每个String存储一个版本，并让其他版本引用存储的版本。**这个过程称为字符串驻留**。由于JVM只能驻留编译时字符串常量，因此我们可以手动调用intern()方法来处理我们想要驻留的字符串。

**JVM将驻留字符串存储在一个特殊的本机固定大小哈希表中，称为字符串表，也称为[字符串池](https://www.baeldung.com/java-string-pool)**。我们可以通过-XX:StringTableSize调整标志配置表大小(即桶的数量)。

除了字符串表之外，还有另一个本地数据区域，称为运行时常量池。JVM使用此池来存储常量，例如必须在运行时解析的编译时数字字面量或方法和字段引用。

### 2.6 本机字节缓冲区

JVM通常是大量本机分配的罪魁祸首，但有时开发人员也可以直接分配本机内存，最常见的方法是JNI的malloc调用和NIO的直接ByteBuffers。

### 2.7 额外的调整标志

在本节中，我们针对不同的优化场景使用了一些JVM调优标志。使用以下提示，我们可以找到与特定概念相关的几乎所有调优标志：

```shell
$ java -XX:+PrintFlagsFinal -version | grep <concept>
```

PrintFlagsFinal打印JVM中的所有–XX选项。例如，要查找所有与元空间相关的标志：

```shell
$ java -XX:+PrintFlagsFinal -version | grep Metaspace
      // truncated
      uintx MaxMetaspaceSize                          = 18446744073709547520                    {product}
      uintx MetaspaceSize                             = 21807104                                {pd product}
      // truncated
```

## 3. 本机内存跟踪(NMT)

现在我们知道了JVM中本机内存分配的常见来源，是时候了解如何监控它们了。**首先，我们应该使用另一个JVM调优标志启用本机内存跟踪：-XX:NativeMemoryTracking=off|sumary|detail**。默认情况下，NMT处于关闭状态，但我们可以启用它来查看其观察结果的摘要或详细视图。

假设我们要跟踪典型Spring Boot应用程序的本机分配：

```shell
$ java -XX:NativeMemoryTracking=summary -Xms300m -Xmx300m -XX:+UseG1GC -jar app.jar
```

在这里，我们启用NMT，同时分配300MB的堆空间，使用G1作为我们的GC算法。

### 3.1 即时快照

当启用NMT时，我们可以随时使用jcmd命令获取本机内存信息：

```shell
$ jcmd <pid> VM.native_memory
```

为了找到JVM应用程序的PID，我们可以使用jps命令：

```shell
$ jps -l                    
7858 app.jar // This is our app
7899 sun.tools.jps.Jps
```

现在，如果我们将jcmd与适当的pid一起使用，VM.native_memory会使JVM打印出有关本机分配的信息：

```shell
$ jcmd 7858 VM.native_memory
```

让我们逐段分析一下NMT输出。

### 3.2 总分配

NMT报告总保留和已提交的内存如下：

```text
Native Memory Tracking:
Total: reserved=1731124KB, committed=448152KB
```

**预留内存代表我们的应用程序可能使用的内存总量，相反，已提交内存等于我们的应用当前正在使用的内存量**。

尽管分配了300MB的堆，但我们应用程序的总保留内存接近1.7GB，远多于此。同样，已提交内存约为440MB，这同样比300MB多得多。

在总计部分之后，NMT报告每个分配源的内存分配。因此，让我们深入探讨每个来源。

### 3.3 堆

NMT报告了我们预期的堆分配：

```text
Java Heap (reserved=307200KB, committed=307200KB)
          (mmap: reserved=307200KB, committed=307200KB)
```

保留和已提交的内存均为300MB，与我们的堆大小设置相匹配。

### 3.4 元空间

以下是NMT对已加载类的类元数据的描述：

```text
Class (reserved=1091407KB, committed=45815KB)
      (classes #6566)
      (malloc=10063KB #8519) 
      (mmap: reserved=1081344KB, committed=35752KB)
```

将近1GB的保留空间和45MB的空间用于加载6566个类。

### 3.5 线程

以下是有关线程分配的NMT报告：

```text
Thread (reserved=37018KB, committed=37018KB)
       (thread #37)
       (stack: reserved=36864KB, committed=36864KB)
       (malloc=112KB #190) 
       (arena=42KB #72)
```

总共为37个线程分配了36MB的内存-每个栈几乎1MB。JVM在创建时将内存分配给线程，因此保留分配和提交分配是相等的。

### 3.6 代码缓存

让我们看看NMT对JIT生成和缓存的汇编指令的看法：

```text
Code (reserved=251549KB, committed=14169KB)
     (malloc=1949KB #3424) 
     (mmap: reserved=249600KB, committed=12220KB)
```

目前，将近13MB的代码被缓存，并且这个数量可能会增加到大约245MB。

### 3.7 GC

以下是有关G1 GC内存使用情况的NMT报告：

```text
GC (reserved=61771KB, committed=61771KB)
   (malloc=17603KB #4501) 
   (mmap: reserved=44168KB, committed=44168KB)
```

我们可以看到，几乎有60MB被保留并用于帮助G1。

让我们看看更简单的GC(比如串行GC)的内存使用情况：

```shell
$ java -XX:NativeMemoryTracking=summary -Xms300m -Xmx300m -XX:+UseSerialGC -jar app.jar
```

串行GC几乎只使用1MB：

```text
GC (reserved=1034KB, committed=1034KB)
   (malloc=26KB #158) 
   (mmap: reserved=1008KB, committed=1008KB)
```

显然，我们不应该仅仅因为内存使用情况而选择GC算法，因为串行GC的Stop-the-World特性可能会导致性能下降。然而，有[几种GC](https://www.baeldung.com/jvm-garbage-collectors)可供选择，它们各自以不同方式平衡内存和性能。

### 3.8 符号

以下是有关符号分配(例如字符串表和常量池)的NMT报告：

```text
Symbol (reserved=10148KB, committed=10148KB)
       (malloc=7295KB #66194) 
       (arena=2853KB #1)
```

将近10MB分配给符号。

### 3.9 随时间推移

**NMT允许我们跟踪内存分配随时间的变化情况**，首先，我们应该将应用程序的当前状态标记为基线：

```shell
$ jcmd <pid> VM.native_memory baseline
Baseline succeeded
```

然后，过一会儿，我们可以将当前内存使用情况与该基线进行比较：

```shell
$ jcmd <pid> VM.native_memory summary.diff
```

NMT使用+和–符号来告诉我们内存使用情况在这段时间内是如何变化的：

```text
Total: reserved=1771487KB +3373KB, committed=491491KB +6873KB
-             Java Heap (reserved=307200KB, committed=307200KB)
                        (mmap: reserved=307200KB, committed=307200KB)
 
-             Class (reserved=1084300KB +2103KB, committed=39356KB +2871KB)
// Truncated
```

总保留内存和已提交内存分别增加了3MB和6MB，可以很容易地发现内存分配中的其他波动。

### 3.10 详细NMT

NMT可以提供关于整个内存空间映射的非常详细的信息，要启用此详细报告，我们应该使用-XX:NativeMemoryTracking=detail调整标志。

## 4. 总结

在本文中，我们列举了JVM中本机内存分配的不同因素。然后，我们学习了如何检查正在运行的应用程序以监控其本机分配。有了这些见解，我们可以更有效地调整应用程序并调整运行时环境的大小。