---
layout: post
title:  无法为对象堆保留足够的空间
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

在本教程中，我们将了解“Could not reserve enough space for object heap”错误的原因，同时介绍一些可能出现的情况。

## 2. 问题

“Could not reserve enough space for object heap”是一个特定的JVM错误，**当Java进程由于运行系统遇到内存限制而无法创建虚拟机时会引发该错误**：

```shell
java -Xms4G -Xmx4G -jar HelloWorld.jar

Error occurred during initialization of VM
Could not reserve enough space for object heap
Error: Could not create the Java Virtual Machine.
Error: A fatal exception has occurred. Program will exit.
```

一般来说，我们遇到错误时可能有两种情况。首先，当我们启动一个带有最大堆大小限制参数(-Xmx)的Java进程时，该值**大于该进程在操作系统上可以拥有的值**。

堆大小限制因以下几个约束而异：

-   硬件架构(32/64位)
-   JVM位版本(32/64位)
-   我们使用的操作系统

其次，当Java进程由于同一系统上运行的其他应用程序消耗内存而无法保留指定数量的内存时。

## 3. 堆大小

**Java堆空间是Java运行时程序的内存分配池**，由JVM自己管理。默认情况下，分配池被限制为初始大小和最大大小。要了解有关Java中堆空间的更多信息，请查看[此处](https://www.baeldung.com/java-stack-heap)的文章。

让我们看看不同环境中的最大堆大小是多少，以及我们如何设置限制。

### 3.1 最大堆大小

32位和64位JVM的最大理论堆限制很容易通过查看可用内存空间来确定，32位JVM为2^32(4GB)，64位JVM为2^64(16EB)。

实际上，由于各种限制，该限制可能会低得多，并且会因操作系统而异。例如，**在32位Windows系统上，最大堆大小范围在1.4GB到1.6GB之间**。相比之下，在32位Linux系统上，最大堆大小可以扩展到3GB。

因此，**如果应用程序需要较大的堆，则应使用64位JVM**。但是，如果堆较大，垃圾回收器将有更多工作要做，因此在堆大小和性能之间找到良好的平衡很重要。

### 3.2 如何控制堆大小限制？

我们有两个选项来控制JVM的堆大小限制。

首先，通过在每次JVM初始化时使用Java命令行参数：

```text
-Xms<size>    Sets initial Java heap size. This value must be a multiple of 1024 and greater than 1 MB.
-Xmx<size>    Sets maximum Java heap size. This value must be a multiple of 1024 and greater than 2 MB.
-Xmn<size>    Sets the initial and maximum size (in bytes) of the heap for the young generation.
```

对于大小值，我们可以附加字母k或K、m或M和g或G分别表示千字节、兆字节和千兆字节。如果没有指定字母，则使用默认单位(字节)。

```text
-Xmn2g
-Xmn2048m
-Xmn2097152k
-Xmn2147483648
```

其次，通过使用环境变量JAVA_OPTS全局配置上述Java命令行参数。因此，系统上的每个JVM初始化都会自动使用环境变量中设置的配置。

```text
JAVA_OPTS="-Xms256m -Xmx512m"
```

要有关更多信息，请查看我们全面的[JVM参数](https://www.baeldung.com/jvm-parameters)指南。

## 4. 总结

在本教程中，我们讨论了JVM无法为对象堆保留足够空间时的两种可能情况，我们还学习了如何控制堆大小限制以缓解此错误。