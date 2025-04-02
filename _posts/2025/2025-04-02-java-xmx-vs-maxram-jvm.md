---
layout: post
title: Xmx和MaxRAM JVM参数之间的差异
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

[堆](https://www.baeldung.com/java-stack-heap#heap-space-in-java)大小是Java应用程序的一个重要参数，它直接影响我们可以使用的内存量，并间接影响应用程序的性能。**例如[压缩指针](https://www.baeldung.com/jvm-compressed-oops)的使用、[垃圾回收](https://www.baeldung.com/jvm-garbage-collectors)周期的数量和持续时间等**。

在本教程中，我们将学习如何使用–XX:MaxRAM标志为堆大小计算提供更多优化机会，在容器内或不同主机上运行应用程序时，这一点尤其重要。

## 2. 堆大小计算

用于配置堆的标志可以协同工作，也可以相互覆盖。**了解它们的关系对于更深入地了解它们的目的非常重要**。

### 2.1 使用-Xmx

控制堆大小的主要方法是[-Xmx和-Xms](https://www.baeldung.com/jvm-parameters#explicit-heap-memory---xms-and-xmx-options)标志，分别控制最大大小和初始大小。**这是一个功能强大的工具，但不考虑计算机或容器上的可用空间**。假设我们在各种主机上运行应用程序，其中可用RAM从4GB到64GB不等。

如果没有-Xmx，[JVM](https://www.baeldung.com/jvm-vs-jre-vs-jdk#jvm)会自动为应用程序堆分配大约25%的可用RAM。**但是，一般来说，JVM分配的初始堆大小取决于各种参数：系统架构、JVM版本、平台等**。

在某些情况下，此行为可能是不可取的。根据可用的RAM，它可能会分配截然不同的堆。让我们检查一下JVM在24GB RAM的机器上默认分配了多少内存：

```shell
$ java -XX:+PrintFlagsFinal -version |\
grep -e '\bMaxHeapSize\|\bMinHeapSize\|\bInitialHeapSize' 
   size_t InitialHeapSize   = 402653184    {product} {ergonomic}
   size_t MaxHeapSize       = 6442450944   {product} {ergonomic}
   size_t MinHeapSize       = 8388608      {product} {ergonomic}
```

**JVM分配了大约6GB或25%，这对于我们的应用程序来说可能太多了**。将最大堆设置为特定值也可能会产生问题，如果我们使用-Xmx4g，则对于可用内存不足的主机，它可能会失败，而且我们也不会获得可以拥有的额外内存：

```shell
$ java -XX:+PrintFlagsFinal -Xmx4g -version |\
grep -e '\bMaxHeapSize\|\bMinHeapSize\|\bInitialHeapSize'
   size_t InitialHeapSize   = 402653184    {product} {ergonomic}
   size_t MaxHeapSize       = 4294967296   {product} {command line}
   size_t MinHeapSize       = 8388608      {product} {ergonomic}
```

在某些情况下，可以通过使用脚本动态计算-Xmx来解决此问题。但是，它绕过了JVM启发式方法，而JVM启发式方法可能更精确地满足应用程序需求。

### 2.2 使用-XX:MaxRAM

标志-XX:MaxRAM旨在解决上述问题。**首先，它可以防止JVM在具有大量RAM的系统上过度分配内存**。我们可以将此标志视为“运行应用程序，但假装你最多拥有X数量的RAM。”

此外，-XX:MaxRAM允许JVM对堆大小使用标准启发式。让我们回顾一下前面的示例，但使用-XX:MaxRAM:

```shell
$ java -XX:+PrintFlagsFinal -XX:MaxRAM=6g -version |\
grep -e '\bMaxHeapSize\|\bMinHeapSize\|\bInitialHeapSize'
   size_t InitialHeapSize   = 100663296    {product} {ergonomic}
   size_t MaxHeapSize       = 1610612736   {product} {ergonomic}
   size_t MinHeapSize       = 8388608      {product} {ergonomic}
```

**在这种情况下，JVM会计算最大堆大小，但假设我们只有6GB RAM**。请注意，我们不应该将-Xmx与-XX:MaxRAM一起使用，因为-Xmx更具体，所以它会覆盖-XX:MaxRAM：

```shell
$ java -XX:+PrintFlagsFinal -XX:MaxRAM=6g -Xmx6g -version |\ 
grep -e '\bMaxHeapSize\|\bMinHeapSize\|\bInitialHeapSize'
   size_t InitialHeapSize   = 100663296    {product} {ergonomic}
   size_t MaxHeapSize       = 6442450944   {product} {command line}
   size_t MinHeapSize       = 8388608      {product} {ergonomic}
```

**此标志可以提高资源利用率和堆分配**。但是，我们仍然无法控制应将多少RAM分配给堆。

### 2.3 使用-XX:MaxRAMPercentage和-XX:MinRAMPercentage

现在我们掌握了控制权，可以告诉JVM它应该考虑多少RAM。让我们定义分配堆的策略，-XX:MaxRAM标志适用于[-XX:MaxRAMPercentage](https://www.baeldung.com/java-jvm-parameters-rampercentage#-xxmaxrampercentage)和[-XX:MinRAMPercentage](https://www.baeldung.com/java-jvm-parameters-rampercentage#-xxminrampercentage)。**它们提供了更大的灵活性，尤其是在容器化环境中**。让我们尝试将其与-XX:MaxRAM -XX:MaxRAM一起使用，并将堆设置为可用RAM的50%：

```shell
$ java -XX:+PrintFlagsFinal -XX:MaxRAM=6g -XX:MaxRAMPercentage=50 -version |\
grep -e '\bMaxHeapSize\|\bMinHeapSize\|\bInitialHeapSize'
   size_t InitialHeapSize   = 100663296    {product} {ergonomic}
   size_t MaxHeapSize       = 3221225472   {product} {ergonomic}
   size_t MinHeapSize       = 8388608      {product} {ergonomic}
```

**人们对-XX:MinRAMPercentage有一个常见的混淆**，它的行为并不像-Xms。不过，可以合理地假设它设置了最小堆大小。让我们检查以下设置：

```shell
$ java -XX:+PrintFlagsFinal -XX:MaxRAM=16g -XX:MaxRAMPercentage=10 -XX:MinRAMPercentage=50 -version |\
grep -e '\bMaxHeapSize\|\bMinHeapSize\|\bInitialHeapSize'
   size_t InitialHeapSize   = 268435456    {product} {ergonomic}
   size_t MaxHeapSize       = 1719664640   {product} {ergonomic}
   size_t MinHeapSize       = 8388608      {product} {ergonomic}
```

我们设置了-XX:MaxRAMPercentage和-XX:MinRAMPercentage，但很明显只有-XX:MaxRAMPercentage有效，**我们将16GB RAM的10%分配给堆**。但是，如果我们将可用RAM减少到200MB，我们会得到不同的行为：

```shell
$ java -XX:+PrintFlagsFinal -XX:MaxRAM=200m -XX:MaxRAMPercentage=10 -XX:MinRAMPercentage=50 -version |\
grep -e '\bMaxHeapSize\|\bMinHeapSize\|\bInitialHeapSize' 
   size_t InitialHeapSize   = 8388608      {product} {ergonomic}
   size_t MaxHeapSize       = 109051904    {product} {ergonomic}
   size_t MinHeapSize       = 8388608      {product} {ergonomic}
```

在这种情况下，堆大小由-XX:MinRAMPercentage控制。当可用RAM降至低于200MB时，此标志就会启动。现在，我们可以将堆增加到75%：

```shell
$ java -XX:+PrintFlagsFinal -XX:MaxRAM=200m -XX:MaxRAMPercentage=10 -XX:MinRAMPercentage=75 -version |\
grep -e '\bMaxHeapSize\|\bMinHeapSize\|\bInitialHeapSize'
   size_t InitialHeapSize   = 8388608      {product} {ergonomic}
   size_t MaxHeapSize       = 134217728    {product} {ergonomic}
   size_t MinHeapSize       = 8388608      {product} {ergonomic}
```

如果我们继续对如此小的堆应用-XX:MaxRAMPercentage，我们将获得20MB的堆，这可能不足以满足我们的目的。这就是为什么我们对小堆和大堆有不同的标志。-XX:MaxRAM标志与它们都可以很好地配合，并为我们提供了更多的控制权。

## 3. 总结

控制堆大小对于Java应用程序至关重要，分配更多内存并不一定是好事；同时，分配不足的内存也是不好的。

使用-Xmx、-XX:MaxRAM、-XX:MaxRAMPercentage和-XX:MinRAMPercentage可以帮助我们更好地调整应用程序并提高性能。