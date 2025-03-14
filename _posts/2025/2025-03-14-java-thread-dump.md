---
layout: post
title:  捕获Java线程转储
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 概述

在本教程中，我们将讨论捕获Java应用程序的线程转储的各种方法。

**线程转储是Java进程中所有线程状态的快照**，每个线程的状态都通过堆栈跟踪来呈现，显示线程堆栈的内容。线程转储可用于诊断问题，因为它可以显示线程的活动。**线程转储以纯文本形式编写，因此我们可以将其内容保存到文件中，稍后在文本编辑器中查看**。

在接下来的部分中，我们将介绍多种工具和方法来生成线程转储。

## 2. 使用JDK实用程序

JDK提供了几个可以捕获Java应用程序的线程转储的实用程序，**所有实用程序都位于JDK主目录内的bin文件夹下**。因此，只要此目录位于我们的系统路径中，我们就可以从命令行执行这些实用程序。

### 2.1 jstack

[jstack](https://docs.oracle.com/en/java/javase/11/tools/jstack.html)是一个命令行JDK实用程序，我们可以使用它来捕获线程转储。它获取进程的pid并在控制台中显示线程转储，或者，我们可以将其输出重定向到文件。

**让我们看一下使用jstack捕获线程转储的基本命令语法**：

```shell
jstack [-F] [-l] [-m] <pid>
```

所有标志都是可选的，让我们看看它们的含义：

- -F选项强制线程转储；当jstack pid没有响应(进程挂起)时很方便使用
- -l选项指示实用程序在堆和锁中查找可拥有的同步器
- -m选项除了打印Java堆栈帧外，还打印本机堆栈帧(C和C++)

让我们通过捕获线程转储并将结果重定向到文件来利用这些知识：

```shell
jstack 17264 > /tmp/threaddump.txt
```

记住，我们可以使用[jps](https://docs.oracle.com/en/java/javase/11/tools/jps.html)命令轻松获取Java进程的pid。

### 2.2 Java Mission Control

**[Java Mission Control](https://docs.oracle.com/javacomponents/jmc-5-5/jmc-user-guide/intro.htm#JMCCI109)(JMC)是一个GUI工具，用于收集和分析来自Java应用程序的数据**。启动JMC后，它会显示本地计算机上运行的Java进程列表。我们还可以通过JMC连接到远程Java进程。

我们可以右键单击该进程，然后单击“Start Flight Recording”选项。此后，“Threads”选项卡将显示线程转储：

![](/assets/images/2025/javaconcurrency/javathreaddump01.png)

### 2.3 jvisualvm

**[jvisualvm](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jvisualvm.html)是一个具有图形用户界面的工具，可以让我们监控、排除故障和分析Java应用程序**。GUI很简单，但非常直观且易于使用。

它的众多选项之一允许我们捕获线程转储。如果我们右键单击Java进程并选择“Thread Dump”选项，该工具将创建一个线程转储并在新选项卡中打开它：

![](/assets/images/2025/javaconcurrency/javathreaddump02.png)

从JDK 9开始，Visual VM不包含在Oracle JDK和Open JDK发行版中，因此，如果我们使用的是Java 9或更新版本，我们可以从Visual VM开源[项目网站](https://visualvm.github.io/)获取JVisualVM。

### 2.4 jcmd

[jcmd](https://docs.oracle.com/en/java/javase/11/tools/jcmd.html)是一个通过向JVM发送命令请求来工作的工具，虽然功能强大，但**它不包含任何远程功能**；我们必须在运行Java进程的同一台机器上使用它。

**它的许多命令之一是Thread.print**，我们可以使用它通过指定进程的pid来获取线程转储：

```shell
jcmd 17264 Thread.print
```

### 2.5 jconsole

[jconsole](https://docs.oracle.com/en/java/javase/11/management/using-jconsole.html)允许我们检查每个线程的堆栈跟踪，如果我们打开jconsole并连接到正在运行的Java进程，**我们可以导航到“Threads”选项卡并找到每个线程的堆栈跟踪**：

![](/assets/images/2025/javaconcurrency/javathreaddump03.png)

### 2.6 总结

事实证明，使用JDK实用程序捕获线程转储的方法有很多种。让我们花点时间总结一下每种方法，并概述它们的优缺点：

- jstack：提供最快捷、最简单的方法来捕获线程转储；但是，从Java 8开始，有更好的替代方案可用
- jmc：增强的JDK分析和诊断工具，它最大限度地减少了分析工具通常存在的性能开销问题
- jvisualvm：轻量级开源分析工具，具有出色的GUI控制台
- jcmd：功能极其强大，推荐用于Java 8及更高版本。一个可以满足多种用途的工具：捕获线程转储(jstack)、堆转储(jmap)、系统属性和命令行参数(jinfo)
- jconsole：允许我们检查线程堆栈跟踪信息

## 3. 从命令行

在企业应用服务器中，出于安全原因，只安装了JRE。因此，我们不能使用上述实用程序，因为它们是JDK的一部分。但是，有各种命令行替代方案可让我们轻松捕获线程转储。

### 3.1 kill -3命令(Linux/Unix)

在类Unix系统中捕获线程转储的最简单方法是通过[kill](https://linux.die.net/man/3/kill)命令，我们可以使用该命令通过kill()系统调用向进程发送信号。在此用例中，我们将向其发送-3信号。

使用与前面示例中相同的pid，让我们看一下如何使用kill来捕获线程转储：

```shell
kill -3 17264
```

这样，接收信号的Java进程将在标准输出上打印线程转储。

如果我们使用以下调整标志组合来运行Java进程，那么它还会将线程转储重定向到给定的文件：

```bash
-XX:+UnlockDiagnosticVMOptions -XX:+LogVMOutput -XX:LogFile=~/jvm.log
```

现在，如果我们发送-3信号，除了标准输出之外，转储还将在~/jvm.log文件中提供。

### 3.2 Ctrl + Break(Windows)

**在Windows操作系统中，我们可以使用CTRL和Break组合键来捕获线程转储**。要获取线程转储，请导航到用于启动Java应用程序的控制台，然后同时按下CTRL和Break键。

值得注意的是，在某些键盘上，Break键不可用。因此，在这种情况下，可以同时使用CTRL、SHIFT和Pause键来捕获线程转储。

这两个命令都会将线程转储打印到控制台。

## 4. 以编程方式使用ThreadMxBean

本文要讨论的最后一个方法是使用[JMX](https://www.baeldung.com/java-management-extensions)，**我们将使用ThreadMxBean来捕获线程转储**。让我们在代码中看一下：

```java
private static String threadDump(boolean lockedMonitors, boolean lockedSynchronizers) {
    StringBuffer threadDump = new StringBuffer(System.lineSeparator());
    ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();
    for(ThreadInfo threadInfo : threadMXBean.dumpAllThreads(lockedMonitors, lockedSynchronizers)) {
        threadDump.append(threadInfo.toString());
    }
    return threadDump.toString();
}
```

在上面的程序中，我们执行了几个步骤：

1. 首先初始化一个空的StringBuffer来保存各个线程的堆栈信息。
2. 然后我们使用ManagementFactory类来获取ThreadMxBean的实例，ManagementFactory是用于获取Java平台托管bean的工厂类。此外，ThreadMxBean是JVM线程系统的管理接口。
3. 将fixedMonitors和fixedSynchronizers值设置为true表示在线程转储中捕获可拥有的同步器和所有锁定的监视器。

## 5. 总结

在本文中，我们学习了多种捕获线程转储的方法。

首先，我们讨论了各种JDK实用程序，然后讨论了命令行替代方案。最后，我们总结了使用JMX的编程方法。