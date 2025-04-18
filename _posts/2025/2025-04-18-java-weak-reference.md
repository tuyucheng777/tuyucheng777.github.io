---
layout: post
title:  Java中的弱引用
category: java
copyright: java
excerpt: Java
---

## 1. 概述

在本文中，我们将了解Java语言中弱引用的概念。

我们将解释它们是什么、它们的用途以及如何正确使用它们。

## 2. 弱引用

**当弱引用对象弱可达时，垃圾收集器会清除该对象**。

弱可达性意味着一个对象既没有强引用也没有[软引用](https://www.baeldung.com/java-soft-references)指向它，只有遍历弱引用才能到达该对象。

首先，**垃圾收集器会清除弱引用，使引用对象不再可访问**。然后，该引用会被放入一个引用队列(如果存在关联的话)，以便我们能够从中获取它。

与此同时，以前弱可达的对象也将被最终确定。

### 2.1 弱引用与软引用

有时，弱引用和软引用之间的区别并不明显。软引用本质上是一个大型的LRU缓存，也就是说，**当引用对象在不久的将来很有可能被重用时，我们会使用软引用**。

由于软引用充当了缓存的角色，即使引用对象本身不可访问，它也可能继续保持可访问状态。事实上，软引用只有在满足以下条件时才可以被回收：

-   引用对象不具有强可达性
-   软引用最近未被访问

因此，软引用在被引用对象不可达后，可能还能够保持几分钟甚至几小时的可用状态；而弱引用则只能在被引用对象仍然存在的情况下保持可用状态。

## 3. 用例

Java文档中提到，**弱引用最常用于实现规范化Map**。如果Map仅包含特定值的一个实例，则称其为规范化的。它不会创建新的对象，而是在Map中查找现有对象并使用它。

当然，**这些引用最广为人知的用途是[WeakHashMap](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/WeakHashMap.html)类**，它是[Map](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/Map.html)接口的实现，其中每个键都存储为对给定键的弱引用。当垃圾回收器删除一个键时，与该键关联的实体也会被删除。

有关更多信息，请查看我们的[WeakHashMap指南](https://www.baeldung.com/java-weakhashmap)。

另一个可以使用它们的领域是**监听者流失问题**。

发布者(或主体)持有对所有订阅者(或监听者)的强引用，用于通知它们发生的事件，**当监听者无法成功取消订阅发布者时就会出现问题**。

因此，由于发布者仍然拥有对监听器的强引用，因此监听器无法被垃圾回收。这样一来，就可能出现内存泄漏。

问题的解决方案可以是主体持有对观察者的弱引用，从而允许前者被垃圾收集而无需取消订阅(请注意，这不是一个完整的解决方案，它引入了一些本文未涉及的其他问题)。

## 4. 使用弱引用

弱引用由[java.lang.ref.WeakReference](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/ref/WeakReference.html)类表示，我们可以通过传递一个引用作为参数来初始化它。此外，我们还可以选择提供一个[java.lang.ref.ReferenceQueue](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/ref/ReferenceQueue.html)：

```java
Object referent = new Object();
ReferenceQueue<Object> referenceQueue = new ReferenceQueue<>();

WeakReference weakReference1 = new WeakReference<>(referent);
WeakReference weakReference2 = new WeakReference<>(referent, referenceQueue);
```

引用的指称可以通过[get](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/ref/Reference.html#get())方法获取，并使用[clear](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/ref/Reference.html#clear())方法手动删除：

```java
Object referent2 = weakReference1.get();
weakReference1.clear();
```

使用这种引用的安全模式与[软引用](https://www.baeldung.com/java-soft-references)相同：

```java
Object referent3 = weakReference2.get();
if (referent3 != null) {
    // GC hasn't removed the instance yet
} else {
    // GC has cleared the instance
}
```

## 5. 总结

在本快速教程中，我们了解了Java中弱引用的低级概念，并重点介绍了使用这些概念的最常见场景。