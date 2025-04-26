---
layout: post
title:  Java中的软引用
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

在这篇简短的文章中，我们将讨论Java中的软引用。

我们将解释它们是什么、为什么需要它们以及如何创建它们。

## 2. 什么是软引用？

软[引用](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/ref/Reference.html)对象(或软可达对象)可能会被垃圾回收器清除，以响应内存需求，**软可达对象没有指向它的强引用**。

当垃圾回收器被调用时，它会开始遍历堆中的所有元素，GC将引用类型的对象存储在一个特殊的队列中。

检查完堆中的所有对象后，GC会通过从上面提到的队列中删除对象来确定应该删除哪些实例。

这些规则因JVM实现而异，但文档指出，在JVM抛出OutOfMemoryError之前，**保证清除对软可达对象的所有软引用**。

但是，无法保证软引用何时被清除，也无法保证对不同对象的一组引用的清除顺序。

通常，JVM实现会在清理最近创建的引用或最近使用的引用之间进行选择。

软可达对象在最后一次被引用后会保留一段时间，默认值为堆中每兆空闲空间的生存期为一秒，此值可以使用[-XX:SoftRefLRUPolicyMSPerMB](http://www.oracle.com/technetwork/java/hotspotfaq-138619.html#gc_softrefs)标志进行调整。

例如，要将值更改为2.5秒(2500毫秒)，我们可以使用：

```shell
-XX:SoftRefLRUPolicyMSPerMB=2500
```

与弱引用相比，软引用具有更长的生命周期，因为它们会持续存在，直到需要额外的内存为止。

因此，如果我们需要尽可能长时间地将对象保存在内存中，它们是更好的选择。

## 3. 软引用的用例

**软引用可用于实现内存敏感的缓存**，其中内存管理是一个非常重要的因素。

只要软引用的指涉对象是强可达的，即确实正在使用中，该引用就不会被清除。

例如，缓存可以通过保留对这些条目的强引用来防止其最近使用的条目被丢弃，而其余条目则由垃圾回收器自行丢弃。

## 4. 使用软引用

在Java中，软引用由[java.lang.ref.SoftReference](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/ref/SoftReference.html)类表示。

我们有两个选项来初始化它，第一种方法是仅传递一个引用：

```java
StringBuilder builder = new StringBuilder();
SoftReference<StringBuilder> reference1 = new SoftReference<>(builder);
```

第二种选择意味着传递一个指向[java.lang.ref.ReferenceQueue](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/ref/ReferenceQueue.html)的引用以及一个指向引用对象的引用，**引用队列的设计目的是让我们了解垃圾回收器执行的操作**，当垃圾回收器决定移除一个引用对象的引用对象时，它会将引用对象附加到引用队列中。

下面展示了如何使用ReferenceQueue初始化SoftReference：

```java
ReferenceQueue<StringBuilder> referenceQueue = new ReferenceQueue<>();
SoftReference<StringBuilder> reference2
 = new SoftReference<>(builder, referenceQueue);
```

根据[java.lang.ref.Reference](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/ref/Reference.html)，它包含方法[get](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/ref/Reference.html#get())和[clear](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/ref/Reference.html#clear())，分别用于获取和重置引用：

```java
StringBuilder builder1 = reference2.get();
reference2.clear();
StringBuilder builder2 = reference2.get(); // null
```

每次使用这种引用时，我们都需要确保get返回的引用存在：

```java
StringBuilder builder3 = reference2.get();
if (builder3 != null) {
    // GC hasn't removed the instance yet
} else {
    // GC has cleared the instance
}
```

## 5. 总结

在本教程中，我们熟悉了软引用的概念及其用例。

此外，我们还学习了如何创建一个并以编程方式使用它。