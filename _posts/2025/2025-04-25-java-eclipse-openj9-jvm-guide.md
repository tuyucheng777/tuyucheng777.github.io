---
layout: post
title:  Eclipse OpenJ9 JVM指南
category: java-jvm
copyright: java-jvm
excerpt: OpenJ9
---

## 1. 概述

OpenJ9是一个高性能、可扩展且灵活的[Java虚拟机](https://www.baeldung.com/jvm-series)，我们可以在许多Java应用程序中使用它来替代HotSpot。IBM最初将其作为其商业JDK的一部分进行开发，现在，该JVM已获得Eclipse基金会的支持。

**我们使用OpenJ9来减少内存占用和启动时间，尤其是在云和容器环境中**。它支持Java SE规范，并且与OpenJDK完美兼容，本文将探讨如何安装OpenJ9 JVM并回顾其主要功能。

## 2. 安装

**我们可以通过下载包含OpenJ9的预构建OpenJDK二进制文件来安装它**，许多发行版(例如Eclipse Temurin)都提供了基于OpenJ9的版本，官方发行列表位于[Eclipse OpenJ9网站](https://projects.eclipse.org/projects/technology.openj9/releases)。

下载存档后，我们提取文件并设置JAVA_HOME环境变量。

### 2.1 Windows

首先，我们将ZIP压缩包解压到指定位置，例如path-to-jdk\\jdk-openj9，然后，设置JAVA_HOME环境变量并修改PATH环境变量：

```shell
$ setx JAVA_HOME "path-to-jdk\jdk-openj9" 
$ setx PATH "%JAVA_HOME%\bin;%PATH%"
```

最后我们验证安装：

```shell
$ java -version
```

### 2.2 Linux

首先，让我们解压存档：

```shell
$ tar -xzf OpenJ9-JDK.tar.gz -C /opt
```

然后我们设置JAVA_HOME并更新PATH：

```shell
$ export JAVA_HOME=/opt/jdk-openj9 
$ export PATH=$JAVA_HOME/bin:$PATH
```

我们可以将这些行添加到.bashrc或.zshrc中，以使它们永久生效，最后，我们检查安装：

```shell
$ java -version
```

现在我们准备使用OpenJ9来运行和开发Java应用程序。

## 3. 垃圾回收策略

OpenJ9提供了多种[垃圾回收策略](https://eclipse.dev/openj9/docs/gc/)，每种策略都针对不同的工作负载进行了优化，我们可以选择最符合自身性能目标的策略。

在HotSpot JVM中，我们通过选择[GC实现](https://www.baeldung.com/jvm-garbage-collectors)并调整许多参数来配置垃圾回收。**在OpenJ9中，我们使用GC策略，每种策略都是针对特定用例设计的，我们在启动时使用虚拟机选项选择一种策略**：

```shell
-Xgcpolicy:<name>
```

让我们看看OpenJ9中可用的GC策略。

### 3.1 分代并发GC策略

**这是默认策略，我们将在大多数应用程序中使用它**。使用此GC策略，我们将堆划分为两个区域：新生代(nursery)和老年代(tenure)，此策略管理两个代组：新生代(new)和老年代(older)。

它使用并发标记-清除执行全局GC，并可选地执行压缩，部分GC是一种Stop-the-World清除或并发清除。

我们使用以下标志启用它：

```shell
-Xgcpolicy:gencon
```

### 3.2 平衡策略

**当我们遇到gencon暂停时间过长的问题时，我们会将此策略用于大型堆和多线程**。此策略将堆划分为大小相等的区域，并获得多代支持。

全局GC使用增量并发标记，部分GC使用Stop-the-World以及可选的标记、清除或压缩。

我们可以使用标志添加此策略：

```shell
-Xgcpolicy:balanced
```

### 3.3 优化暂停时间策略

**我们选择此策略是为了最大限度地减少暂停时间，例如，当堆大小较大并且会影响GC暂停时长时，此策略可能很有用**。此策略使用单个平坦的堆区域，这里我们只有一代。

GC是并发标记-清除，带有可选的压缩。

要开始使用此策略，我们必须添加以下标志：

```shell
-Xgcpolicy:optavgpause
```

### 3.4 优化吞吐量策略

**当我们希望在短期应用程序中实现最大吞吐量时，我们会选择此策略**。此处我们有一个平坦的堆区域，此策略支持单代。

GC是一种具有可选压缩功能的Stop-the-World标记-清除操作。

我们可以使用以下标志启用此策略：

```shell
-Xgcpolicy:optthruput
```

### 3.5 节拍器策略

**我们将此策略用于实时和软实时应用**，它将堆划分为按大小分类的区域，这样我们就可以获得一代的支持。

节拍器GC以短暂可中断的突发方式运行，以避免长时间的Stop-the-World暂停。

要使用节拍器策略，我们可以打开标志：

```shell
-Xgcpolicy:metronome
```

### 3.6 nogc政策

**我们使用此策略进行测试或内存不回收的特殊用例**，这里我们有一个扁平堆，不执行任何GC。

我们可以使用以下标志启用它：

```shell
-Xgcpolicy:nogc
```

## 4. 类数据共享和AOT编译

OpenJ9开箱即用地支持[类数据共享](https://eclipse.dev/openj9/docs/shrc/#introduction-to-class-data-sharing)和[提前编译](https://www.baeldung.com/ahead-of-time-compilation)，我们可以利用这些特性来缩短启动时间并减少内存占用。

### 4.1 类数据共享

**类数据共享允许我们将类元数据缓存在共享内存缓存中**，我们只需创建一次共享类缓存，即可在JVM运行期间重复使用它。这可以加快类加载速度并减少内存占用，尤其是在容器和微服务环境中。

为了启用CDS，我们使用以下标志启动JVM：

```shell
-Xshareclasses
```

我们还可以命名缓存并控制其大小：

```shell
-Xshareclasses:name=myCache,cacheDir=/path/to/cache,cacheSize=100M
```

要查看缓存统计信息，我们使用：

```shell
-Xshareclasses:cacheStats
```

### 4.2 提前编译

OpenJ9中的预编译功能会在运行时之前将Java字节码编译为原生代码，这减少了JIT的预热时间，并提高了启动性能。

与需要GraalVM进行AOT编译的HotSpot不同，OpenJ9内置了对AOT的支持，**虚拟机会自动选择哪些方法应该基于AOT编译，但我们可以使用以下标志禁用此功能**：

```shell
-Xnoaot
```

## 5. 诊断数据和工具

**OpenJ9和HotSpot一样支持JMX，我们可以将[现有的监控工具](https://www.baeldung.com/java-jvm-monitor-non-heap-memory-usage)连接到它**。

此外，OpenJ9还提供了其他工具，可用于排查性能问题、崩溃或内存问题。我们可以使用转储代理来[生成堆转储](https://eclipse.dev/openj9/docs/dump_heapdump/)、[系统转储](https://eclipse.dev/openj9/docs/dump_systemdump/)和[常规JVM信息转储](https://eclipse.dev/openj9/docs/dump_javadump/)。

为了实时监控CPU、内存、GC和线程活动，我们可以利用[IBM Health Center](https://www.ibm.com/docs/en/mon-diag-tools?topic=center-introduction)。

## 6. 总结

在本教程中，我们了解了OpenJ9在减少内存使用和加快启动速度方面如何成为绝佳选择。它非常适合云工作负载、微服务和容器化应用，**但是，在生产环境中使用OpenJ9之前，我们应该根据具体工作负载进行测试**。

同样的建议也适用于GC策略的选择，每个策略针对不同的用例，我们需要衡量和比较，以选择最佳策略。

通过正确的设置，OpenJ9可以提供稳定的性能并节省资源。