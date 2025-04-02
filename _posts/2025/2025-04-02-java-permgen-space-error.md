---
layout: post
title: 处理“java.lang.OutOfMemoryError:PermGen space”错误
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

PermGen(永久代)是为运行基于JVM的应用程序分配的一块特殊内存，[PermGen错误](https://www.baeldung.com/java-memory-leaks)是java.lang.OutOfMemoryError家族中的一个错误，它表示资源(内存)耗尽。

在这个快速教程中，我们将了解导致java.lang.OutOfMemoryError: Permgen space错误的原因及其解决方法。

## 2. Java内存类型

JVM使用两种类型的内存：栈和[堆](https://www.baeldung.com/cs/heap-vs-binary-search-tree)。栈仅用于存储原始类型和对象地址，而堆包含对象的值。**当我们谈论内存错误时，我们总是指堆。永久代实际上是堆内存的一部分，但与JVM的主内存是分开的，处理方式也不同**。要掌握的最重要的概念是，堆中可能剩余大量可用空间，并且仍然会耗尽永久代内存。

永久代的主要作用是存储Java应用程序运行时的静态内容：具体来说，它包含静态方法、静态变量、对静态对象的引用和类文件。

## 3. java.lang.OutOfMemoryError: PermGen

**简单来说，当为永久代分配的空间不再能够存储对象时，就会出现这个错误。发生这种情况是因为永久代不是动态分配的并且具有固定的最大容量**。对于64位版本的JVM，默认大小为82Mb，对于旧的32位JVM，默认大小为64Mb。

永久代耗尽的最常见原因之一是与类加载器相关的内存泄漏，实际上，永久代包含类文件，类加载器负责加载Java类。类加载器问题经常出现在应用服务器中，因为应用服务器会实例化多个类加载器以实现各种应用程序的独立部署。

当应用程序被取消部署并且服务器容器保留一个或多个类的引用时，就会出现问题。如果发生这种情况，则无法对类加载器本身进行垃圾回收，从而使永久代内存与它的类文件一起饱和。永久代崩溃的另一个常见原因是应用程序线程在应用程序被取消部署后继续运行，从而维护在内存中分配的多个对象。

## 4. 处理错误

### 4.1 调整正确的JVM参数

对于有限的内存空间，首先要做的是尽可能增加该空间。通过使用特定的标志，可以增加永久代空间的默认大小。具有数千个类或大量Java字符串的大型应用程序通常需要更大的永久代空间。通过使用[JVM参数](https://www.baeldung.com/jvm-parameters)–XX:MaxPermSize可以指定更大的空间分配给该内存区域。 

**既然我们提到了JVM标志，那么还值得一提的是一个可能触发此错误的不常用标志**。-Xnoclassgc JVM参数，当在JVM启动指定时，明确地从要删除的实体列表中删除类文件。在应用程序服务器和每个应用程序生命周期加载和卸载类数千次的现代框架中，这会导致永久代空间非常快地耗尽。

在旧版本的Java中，类是堆的永久组成部分，这意味着一旦加载，它们就会保留在内存中。通过指定CMSClassUnloadingEnabled(对于Java 1.5或CMSPermGenSweepingEnabled对于Java 1.6)JVM参数，可以启用类的垃圾回收。如果我们碰巧使用Java 1.6，UseConcMarkSweepGC也必须设置为true。否则，CMSClassUnloadEnabled参数将被忽略。 

### 4.2 升级到JVM 8+

修复此类错误的另一种方法是升级到更新版本的Java，从Java版本8开始，[永久代已完全被元空间取代](https://www.baeldung.com/java-permgen-metaspace)，元空间具有可自动调整大小的空间和能够清除死类的高级功能。 

### 4.3 堆分析

值得注意的是，在内存泄漏的情况下，所提供的解决方案都无法满足要求。无论内存大小如何，内存都会耗尽，即使是元空间可用的内存量也是有限的。深度堆分析有时是唯一的解决方案，可以使用VisualGC或JProfiler等工具进行。

## 5. 总结

在这篇简短的文章中，我们了解了永久代内存的用途以及与堆内存的主要区别。接下来，我们了解了java.lang.OutOfMemoryError: Permgen错误的含义以及在哪些特殊情况下会被触发。在上一节中，我们重点介绍了在尝试解决这个特殊问题时可以使用的各种解决方案。