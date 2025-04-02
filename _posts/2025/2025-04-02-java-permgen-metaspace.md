---
layout: post
title:  Java中的Permgen与Metaspace
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

在本快速教程中，我们将研究Java环境中PermGen和Metaspace内存区域之间的差异。

重要的是要记住，从Java 8开始，Metaspace取代了PermGen-带来了一些实质性的变化。

## 2. PermGen

**PermGen(永久代)是一个与主内存堆分离的特殊堆空间**。

JVM跟踪PermGen中加载的类元数据。此外，JVM将所有静态内容存储在该内存部分中，这包括所有静态方法、原始变量和对静态对象的引用。

此外，**它还包含有关字节码、名称和JIT信息的数据**。在Java 7之前，字符串池也是这块内存的一部分。固定池大小的缺点在我们的[文章](https://www.baeldung.com/java-string-pool)中列出。

32位JVM的默认最大内存大小为64MB，64位版本为82MB。

但是，我们可以使用JVM选项更改默认大小：

-   -XX:PermSize=[size\]是PermGen空间的初始或最小大小
-   -XX:MaxPermSize=[size\]为最大大小

最重要的是，**Oracle在JDK 8版本中完全移除了这个内存空间**。因此，如果我们在Java 8和更新版本中使用这些调整标志，我们将收到以下警告：

```shell
>> java -XX:PermSize=100m -XX:MaxPermSize=200m -version
OpenJDK 64-Bit Server VM warning: Ignoring option PermSize; support was removed in 8.0
OpenJDK 64-Bit Server VM warning: Ignoring option MaxPermSize; support was removed in 8.0
...
```

**由于PermGen的内存大小有限，因此会产生著名的OutOfMemoryError错误**。简而言之，[类加载器](https://www.baeldung.com/java-classloaders)没有正确地进行垃圾回收，因此产生了内存泄漏。

因此，我们收到[内存空间错误](https://www.baeldung.com/java-gc-overhead-limit-exceeded)；这主要发生在开发环境中，在创建新的类加载器时。

## 3. 元空间

简单的说，Metaspace是一个新的内存空间-从Java 8版本开始；**它取代了老版本的PermGen内存空间**，最显著的区别是它如何处理内存分配。

具体来说，**这个本机内存区域默认会自动增长**。

我们还有新的标志来调整内存：

-   MetaspaceSize和MaxMetaspaceSize：可以设置元空间上限
-   MinMetaspaceFreeRatio：[垃圾回收](https://www.baeldung.com/jvm-garbage-collectors)后可用的类元数据容量的最小百分比
-   MaxMetaspaceFreeRatio：垃圾回收后类元数据容量的最大可用百分比，以避免空间减少

此外，垃圾回收过程也从这一变化中获得了一些好处。一旦类元数据使用量达到其最大元空间大小，垃圾回收器现在会自动触发对死类的清理。

因此，**通过这种改进，JVM减少了出现OutOfMemory错误的机会**。

尽管有所有这些改进，我们仍然需要监视和[调整元空间](https://www.baeldung.com/jvm-parameters)以避免内存泄漏。

## 4. 总结

在这篇简短的文章中，我们简要描述了PermGen和Metaspace内存区域。此外，我们还解释了它们之间的主要区别。

PermGen仍然存在于JDK 7和更早的版本中，但是Metaspace为我们的应用程序提供了更灵活和可靠的内存使用。