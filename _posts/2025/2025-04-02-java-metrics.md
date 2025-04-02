---
layout: post
title:  使用Java监控磁盘使用情况和其他指标
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

在本快速教程中，我们将讨论如何在Java中监控关键指标。我们将重点关注**磁盘空间、内存使用和线程数据-仅使用核心Java API**。

在我们的第一个示例中，我们将使用[File](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/File.html)类来查询特定的磁盘信息。

然后，我们将深入研究[ManagementFactory](https://docs.oracle.com/en/java/javase/11/docs/api/java.management/java/lang/management/ManagementFactory.html)类来分析内存使用情况和处理器信息。

最后，我们将介绍**如何使用Java Profilers在运行时监控这些关键指标**。

## 2. File类介绍

简单地说，**File类表示文件或目录的抽象**。它可用于获取有关文件系统的关键信息并保持与文件路径相关的操作系统独立性。在本教程中，我们将使用此类来检查Windows和Linux机器上的根分区。

## 3. ManagementFactory

**Java提供ManagementFactory类作为工厂，用于获取包含有关JVM的特定信息的托管bean(MXBeans)**，我们将在以下代码示例中检查两个。

### 3.1 MemoryMXBean

**MemoryMXBean表示JVM内存系统的管理接口**，在运行时，JVM创建此接口的单个实例，我们可以使用ManagementFactory的getMemoryMXBean()方法检索该实例。

### 3.2 ThreadMXBean

与MemoryMXBean类似，ThreadMXBean是JVM线程系统的管理接口，可以使用getThreadMXBean()方法调用它并保存有关线程的关键数据。

在下面的示例中，我们将使用ThreadMXBean来获取JVM的ThreadInfo类-**它包含有关在JVM上运行的线程的特定信息**。

## 4. 监控磁盘使用情况

在此代码示例中，我们将使用 File 类来包含有关分区的关键信息。以下示例将返回 Windows 计算机上 C: 驱动器的可用空间、总空间和可用空间：

```java
File cDrive = new File("C:");
System.out.println(String.format("Total space: %.2f GB", (double)cDrive.getTotalSpace() /1073741824));
System.out.println(String.format("Free space: %.2f GB", (double)cDrive.getFreeSpace() /1073741824));
System.out.println(String.format("Usable space: %.2f GB", (double)cDrive.getUsableSpace() /1073741824));
```

同样，我们可以返回Linux机器的根目录的相同信息：

```java
File root = new File("/");
System.out.println(String.format("Total space: %.2f GB", (double)root.getTotalSpace() /1073741824));
System.out.println(String.format("Free space: %.2f GB", (double)root.getFreeSpace() /1073741824));
System.out.println(String.format("Usable space: %.2f GB", (double)root.getUsableSpace() /1073741824));
```

上述代码打印出定义文件的总空间、可用空间和可用空间。默认情况下，上述方法提供的是字节数，我们将这些字节转换为千兆字节，以使结果更易于阅读。

## 5. 监控内存使用情况

**我们现在将使用ManagementFactory类通过调用MemoryMXBean来查询JVM可用的内存**。 

在此示例中，我们将主要关注查询堆内存。需要注意的是，非堆内存也可以使用MemoryMXBean查询：

```java
MemoryMXBean memoryMXBean = ManagementFactory.getMemoryMXBean();
System.out.println(String.format("Initial memory: %.2f GB", (double)memoryMXBean.getHeapMemoryUsage().getInit() /1073741824));
System.out.println(String.format("Used heap memory: %.2f GB", (double)memoryMXBean.getHeapMemoryUsage().getUsed() /1073741824));
System.out.println(String.format("Max heap memory: %.2f GB", (double)memoryMXBean.getHeapMemoryUsage().getMax() /1073741824));
System.out.println(String.format("Committed memory: %.2f GB", (double)memoryMXBean.getHeapMemoryUsage().getCommitted() /1073741824));
```

上述示例分别返回初始、已用、最大和已提交的内存。以下是对其含义的简单[解释](https://docs.oracle.com/en/java/javase/21/docs/api/java.management/java/lang/management/MemoryUsage.html)：

-   初始内存：JVM在启动时向OS请求的初始内存
-   已使用内存：JVM当前使用的内存量
-   最大内存：JVM可用的最大内存，如果达到此限制，则可能会抛出[OutOfMemoryException](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/OutOfMemoryError.html) 
-   已提交内存：保证JVM可用的内存量

## 6. CPU使用率

接下来，我们将**使用ThreadMXBean获取ThreadInfo对象的完整列表**，并查询它们以获取有关JVM上运行的当前线程的有用信息。

```java
ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();

for(Long threadID : threadMXBean.getAllThreadIds()) {
    ThreadInfo info = threadMXBean.getThreadInfo(threadID);
    System.out.println("Thread name: " + info.getThreadName());
    System.out.println("Thread State: " + info.getThreadState());
    System.out.println(String.format("CPU time: %s ns", threadMXBean.getThreadCpuTime(threadID)));
}
```

首先，代码使用getAllThreadIds方法获取当前线程的列表。然后，针对每个线程，输出线程的名称和状态，以及线程的CPU时间(以纳秒为单位)。

## 7. 使用分析器监控指标

最后，值得一提的是，**我们无需使用任何Java代码即可监视这些关键指标**。Java Profilers密切监视JVM级别的关键构造和操作，并提供内存、线程等的实时分析。

VisualVM就是Java分析器的一个例子，自Java 6以来就与JDK捆绑在一起。许多集成开发环境(IDE)都包含插件，以便在开发新代码时利用分析器。你可以在[此处](https://www.baeldung.com/java-profilers)了解有关Java Profiler和VisualVM的更多信息。

## 8. 总结

在本文中，我们讨论了如何使用核心Java API来查询有关磁盘使用情况、内存管理和线程信息的关键信息。

我们研究了使用File和ManagementFactory类来获取这些指标的多个示例。