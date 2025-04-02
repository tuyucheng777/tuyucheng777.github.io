---
layout: post
title:  如何分析Java线程转储
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

应用程序有时会挂起或运行缓慢，而找出根本原因并不总是一件容易的事。**线程转储提供了正在运行的Java进程的当前状态的快照**。但是，生成的数据包括多个长文件。因此，我们需要分析Java线程转储并在大量不相关的信息中挖掘问题。 

在本教程中，我们将了解如何过滤这些数据以有效诊断性能问题。此外，我们还将学习如何检测瓶颈甚至简单的错误。

## 2. JVM中的线程

JVM使用线程来执行每个内部和外部操作。众所周知，垃圾收集进程有自己的线程，而且Java应用程序内部的任务也会创建自己的线程。

在其生命周期中，线程会经历[多种状态](https://www.baeldung.com/java-thread-lifecycle)，每个线程都有一个跟踪当前操作的执行栈。此外，JVM还存储了之前调用成功的所有方法。因此，可以分析完整的堆栈以研究出现问题时应用程序发生了什么。

为了展示本教程的主题，我们将使用一个简单的[发送方-接收方应用程序(NetworkDriver)](https://www.baeldung.com/java-wait-notify#sender-receiver-synchronization-problem)作为示例。Java程序发送和接收数据包，以便我们能够分析幕后发生的事情。

### 2.1 捕获Java线程转储

应用程序运行后，有多种方法可以生成用于诊断的[Java线程转储](https://www.baeldung.com/java-thread-dump)。在本教程中，我们将使用JDK 7+安装中包含的两个实用程序。首先，我们将执行[JVM进程状态(jps)](https://docs.oracle.com/en/java/javase/11/tools/jps.html)命令来发现应用程序的PID进程：

```shell
$ jps 
80661 NetworkDriver
33751 Launcher
80665 Jps
80664 Launcher
57113 Application
```

其次，我们获取应用程序的PID，在本例中为NetworkDriver旁边的PID。然后，我们将使用[jstack](https://docs.oracle.com/en/java/javase/11/tools/jstack.html)捕获线程转储。最后，我们将结果存储在一个文本文件中：

```shell
$ jstack -l 80661 > sender-receiver-thread-dump.txt
```

### 2.2 示例转储的结构

让我们看一下生成的线程转储。第一行显示时间戳，第二行显示有关JVM的信息：

```text
2021-01-04 12:59:29
Full thread dump OpenJDK 64-Bit Server VM (15.0.1+9-18 mixed mode, sharing):
```

下一部分显示安全内存回收(SMR)和非JVM内部线程：

```text
Threads class SMR info:
_java_thread_list=0x00007fd7a7a12cd0, length=13, elements={
0x00007fd7aa808200, 0x00007fd7a7012c00, 0x00007fd7aa809800, 0x00007fd7a6009200,
0x00007fd7ac008200, 0x00007fd7a6830c00, 0x00007fd7ab00a400, 0x00007fd7aa847800,
0x00007fd7a6896200, 0x00007fd7a60c6800, 0x00007fd7a8858c00, 0x00007fd7ad054c00,
0x00007fd7a7018800
}
```

然后，转储显示线程列表，每个线程包含以下信息：

-   **名称**：如果开发人员包含有意义的线程名称，它可以提供有用的信息
-   **优先级**(prior)：线程的优先级
-  **Java ID**(tid)：JVM给定的唯一ID
-   **本机ID**(nid)：操作系统提供的唯一ID，有助于提取与CPU或内存处理的相关性
-   **状态**：线程的[实际状态](https://www.baeldung.com/java-thread-lifecycle)
-   **堆栈跟踪**：解释应用程序正在发生的事情的最重要信息来源

我们可以从上到下看到快照时不同线程在做什么，让我们只关注堆栈中等待使用消息的有趣部分：

```text
"Monitor Ctrl-Break" #12 daemon prio=5 os_prio=31 cpu=17.42ms elapsed=11.42s tid=0x00007fd7a6896200 nid=0x6603 runnable  [0x000070000dcc5000]
   java.lang.Thread.State: RUNNABLE
	at sun.nio.ch.SocketDispatcher.read0(java.base@15.0.1/Native Method)
	at sun.nio.ch.SocketDispatcher.read(java.base@15.0.1/SocketDispatcher.java:47)
	at sun.nio.ch.NioSocketImpl.tryRead(java.base@15.0.1/NioSocketImpl.java:261)
	at sun.nio.ch.NioSocketImpl.implRead(java.base@15.0.1/NioSocketImpl.java:312)
	at sun.nio.ch.NioSocketImpl.read(java.base@15.0.1/NioSocketImpl.java:350)
	at sun.nio.ch.NioSocketImpl$1.read(java.base@15.0.1/NioSocketImpl.java:803)
	at java.net.Socket$SocketInputStream.read(java.base@15.0.1/Socket.java:981)
	at sun.nio.cs.StreamDecoder.readBytes(java.base@15.0.1/StreamDecoder.java:297)
	at sun.nio.cs.StreamDecoder.implRead(java.base@15.0.1/StreamDecoder.java:339)
	at sun.nio.cs.StreamDecoder.read(java.base@15.0.1/StreamDecoder.java:188)
	- locked <0x000000070fc949b0> (a java.io.InputStreamReader)
	at java.io.InputStreamReader.read(java.base@15.0.1/InputStreamReader.java:181)
	at java.io.BufferedReader.fill(java.base@15.0.1/BufferedReader.java:161)
	at java.io.BufferedReader.readLine(java.base@15.0.1/BufferedReader.java:326)
	- locked <0x000000070fc949b0> (a java.io.InputStreamReader)
	at java.io.BufferedReader.readLine(java.base@15.0.1/BufferedReader.java:392)
	at com.intellij.rt.execution.application.AppMainV2$1.run(AppMainV2.java:61)

   Locked ownable synchronizers:
	- <0x000000070fc8a668> (a java.util.concurrent.locks.ReentrantLock$NonfairSync)
```

乍一看，我们看到主堆栈跟踪正在执行java.io.BufferedReader.readLine，这是预期的行为。如果我们进一步向下看，**我们将看到应用程序在后台执行的所有JVM方法**。因此，我们可以通过查看源代码或其他JVM内部处理来确定问题的根源。

在转储结束时，我们会注意到**有几个额外的线程在执行后台操作，例如垃圾收集(GC)或对象终止**：

```text
"VM Thread" os_prio=31 cpu=1.85ms elapsed=11.50s tid=0x00007fd7a7a0c170 nid=0x3603 runnable  
"GC Thread#0" os_prio=31 cpu=0.21ms elapsed=11.51s tid=0x00007fd7a5d12990 nid=0x4d03 runnable  
"G1 Main Marker" os_prio=31 cpu=0.06ms elapsed=11.51s tid=0x00007fd7a7a04a90 nid=0x3103 runnable  
"G1 Conc#0" os_prio=31 cpu=0.05ms elapsed=11.51s tid=0x00007fd7a5c10040 nid=0x3303 runnable  
"G1 Refine#0" os_prio=31 cpu=0.06ms elapsed=11.50s tid=0x00007fd7a5c2d080 nid=0x3403 runnable  
"G1 Young RemSet Sampling" os_prio=31 cpu=1.23ms elapsed=11.50s tid=0x00007fd7a9804220 nid=0x4603 runnable  
"VM Periodic Task Thread" os_prio=31 cpu=5.82ms elapsed=11.42s tid=0x00007fd7a5c35fd0 nid=0x9903 waiting on condition
```

最后，转储显示Java本机接口(JNI)引用。当发生内存泄漏时，我们应该特别注意这一点，因为它们不会自动被垃圾收集：

```text
JNI global refs: 15, weak refs: 0
```

线程转储的结构非常相似，但我们希望摆脱为我们的用例生成的不重要数据。另一方面，我们需要保留和分组堆栈跟踪生成的大量日志中的重要信息，让我们看看如何做到这一点。

## 3. 分析线程转储的建议

为了了解应用程序发生了什么，我们需要有效地分析生成的快照。我们将获得大量信息，其中包含转储时所有线程的精确数据。但是，我们需要整理日志文件，进行一些过滤和分组以从堆栈跟踪中提取有用的提示。准备好转储后，我们将能够使用不同的工具分析问题，让我们看看如何解读示例转储的内容。

### 3.1 同步问题

过滤堆栈跟踪的一个有趣技巧是线程的状态，我们将**主要关注RUNNABLE或BLOCKED线程，最后关注TIMED_WAITING线程**。这些状态将向我们指出两个或多个线程之间发生冲突的方向：

-   **在死锁情况下，多个线程运行在共享对象上持有一个同步块**
-   **在线程争用中，当一个线程被阻塞等待其他线程完成时，比如上一节生成的转储**

### 3.2 执行问题

根据经验，**对于异常高的CPU使用率，我们只需要查看RUNNABLE线程**。我们将使用线程转储和其他命令来获取更多信息，其中一个命令是[top](https://www.baeldung.com/linux/top-command) -H -p PID，它显示哪些线程正在消耗该特定进程中的操作系统资源。我们还需要查看内部JVM线程(例如GC)，以防万一。另一方面，**当处理性能异常低下时，我们将查看BLOCKED线程**。

在这些情况下，单一的转储肯定不足以了解正在发生的事情。**我们需要在很短的时间间隔内进行多次转储**，以便比较不同时间相同线程的堆栈。一方面，一个快照并不总是足以找出问题的根源。另一方面，我们需要避免快照之间的噪音(信息太多)。

为了了解线程随时间的演变，**推荐的最佳做法是至少进行3次转储，每10秒一次**。另一个有用的技巧是将转储分成小块以避免加载文件时崩溃。

### 3.3 建议

为了有效地破译问题的根源，我们需要整理堆栈跟踪中的大量信息。因此，我们将考虑以下建议：

-   对于执行问题，**以10秒的间隔捕获多个快照将有助于专注于实际问题**。如果需要，还建议拆分文件以避免加载崩溃
-   **在创建新线程时使用命名**以更好地识别源代码
-   根据问题，**忽略内部JVM进程**(例如GC)
-   当出现CPU或内存使用异常时，**重点关注长时间运行或阻塞的线程**
-   **使用top -H -p PID将线程的堆栈与CPU处理相关联**
-   最重要的是，**使用分析器工具**

手动分析Java线程转储可能是一项繁琐的工作，对于简单的应用程序，可以确定产生问题的线程。另一方面，对于复杂的情况，我们需要工具来简化这项任务。我们将在下一节中展示如何使用这些工具，并使用为示例线程争用生成的转储。

## 4. 在线工具

有几种在线工具可用，在使用这类软件时我们需要考虑到安全问题。请记住，我们可能会与第三方实体共享日志。

### 4.1 FastThread

[FastThread](https://fastthread.io/)可能是分析生产环境线程转储的最佳在线工具，它提供了非常漂亮的图形用户界面。它还包括多种功能，例如线程的CPU使用率、堆栈长度以及最常用和最复杂的方法：

![](/assets/images/2025/javajvm/javaanalyzethreaddumps01.png)

FastThread集成了REST API功能来自动分析线程转储，使用简单的cURL命令，可以立即发送结果。主要缺点是安全性，因为它将堆栈跟踪存储在云中。

### 4.2 JStack Review

[JStack Review](https://jstack.review/)是一个在线工具，用于在浏览器中分析转储。它只是客户端，因此不会在计算机之外存储任何数据。从安全角度来看，这是使用它的一大优势。它提供所有线程的图形概览，显示正在运行的方法，并按状态对它们进行分组。JStack Review将产生堆栈的线程与其余线程分开，这对于忽略内部进程等非常重要。最后，它还包括同步器和忽略的行：

![](/assets/images/2025/javajvm/javaanalyzethreaddumps02.png)

### 4.3 Spotify在线Java线程转储分析器

[Spotify Online Java Thread Dump Analyzer](https://spotify.github.io/threaddump-analyzer/)是一个用JavaScript编写的在线开源工具，它以纯文本显示结果，区分有堆栈和无堆栈的线程。它还显示正在运行的线程中最常用的方法：

![](/assets/images/2025/javajvm/javaanalyzethreaddumps03.png)

## 5. 独立应用

还有几个我们可以在本地使用的独立应用程序。

### 5.1 JProfiler

[JProfiler](https://www.ej-technologies.com/products/jprofiler/overview.html)是市场上最强大的工具，在Java开发人员社区中广为人知，可以使用10天试用许可证来测试功能。JProfiler允许创建配置文件并将正在运行的应用程序附加到它们。它包括多种功能，可在现场识别问题，例如CPU和内存使用情况以及数据库分析。它还支持与IDE集成：

![](/assets/images/2025/javajvm/javaanalyzethreaddumps04.png)

### 5.2 IBM Java线程监控和转储分析器 (TMDA)

[IBM TMDA](https://www.ibm.com/support/pages/ibm-thread-and-monitor-dump-analyzer-java-tmda)可用于识别线程争用、死锁和瓶颈，它是免费分发和维护的，但不提供IBM的任何保证或支持：

![](/assets/images/2025/javajvm/javaanalyzethreaddumps05.png)

### 5.3 Irockel线程转储分析器(TDA)

[Irockel TDA](https://github.com/irockel/tda)是一款独立的开源工具，最新版本(v2.4)于2020年8月发布，因此维护良好。它将线程转储显示为一棵树，还提供一些统计信息以简化导航：

![](/assets/images/2025/javajvm/javaanalyzethreaddumps06.png)

最后，IDE支持线程转储的基本分析，因此可以在开发期间调试应用程序。

## 5. 总结

在本文中，我们演示了Java线程转储分析如何帮助我们查明同步或执行问题。

最重要的是，我们回顾了如何正确分析它们，包括组织快照中嵌入的大量信息的建议。