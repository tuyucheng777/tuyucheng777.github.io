---
layout: post
title:  Java中的强引用、弱引用、软引用和虚引用
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

当我们用Java编程时，我们经常使用硬引用，通常甚至都没有考虑过-这是有充分理由的，因为它们是大多数情况下的最佳选择。然而，有时我们需要更多地控制对象可用于垃圾回收器清除的时间。

在本文中，我们将探讨硬引用类型和各种非硬引用类型之间的区别以及何时可以使用它们。

## 2. 硬引用

硬(或强)引用是默认的引用类型，大多数时候，我们甚至可能没有想过引用的对象何时以及如何被垃圾回收。**如果可以通过任何强引用访问该对象，则该对象不能被垃圾回收**。假设我们创建了一个ArrayList对象并将其分配给list变量：

```java
List<String> list = new ArrayList<>;
```

垃圾回收器无法收集这个列表，因为我们在list变量中持有对它的强引用。但如果我们随后使变量无效：

```java
list = null;
```

现在，也只是现在，可以收集ArrayList对象，因为没有任何对象持有对它的引用。

## 3. 超越硬引用

硬引用是默认设置，这是有原因的。它们让垃圾回收器按预期工作，因此我们不必担心管理内存分配。尽管如此，在某些情况下，即使我们仍然持有对这些对象的引用，我们仍希望收集对象并释放内存。

## 4. 软引用

[软引用](https://www.baeldung.com/java-soft-references)告诉垃圾回收器可以根据收集器的判断收集引用的对象。对象可以在内存中保留一段时间，直到收集器决定需要回收它。这种情况会发生，尤其是当JVM面临内存不足的风险时。**在抛出OutOfMemoryError异常之前，应清除所有对只能通过软引用访问的对象的软引用**。

我们可以通过将其包装在我们的对象周围来轻松地使用软引用：

```java
SoftReference<List<String>> listReference = new SoftReference<List<String>>(new ArrayList<String>());
```

如果我们想要检索引用对象，我们可以使用get方法。因为该对象可能已经被清除，所以我们需要检查它：

```java
List<String> list = listReference.get();
if (list == null) {
    // object was already cleared
}
```

### 4.1 用例

**软引用可用于使我们的代码对与内存不足相关的错误更有弹性**。例如，我们可以创建一个内存敏感的缓存，当内存不足时自动清除对象。我们不需要手动管理内存，因为垃圾回收器会为我们做这件事。

## 5. 弱引用

**不会阻止收集仅由[弱引用](https://www.baeldung.com/java-weak-reference)引用的对象**，从垃圾回收的角度来看，它们根本不可能存在。如果应该保护弱引用对象不被清除，那么它也应该被某种硬引用引用。

### 5.1 用例

弱引用最常用于创建规范化Map，这些Map仅映射可访问的对象。一个很好的例子是WeakHashMap，它像普通的HashMap一样工作，但它的键是弱引用的，当引用被清除时它们会自动删除。

使用WeakHashMap，我们可以创建一个短期缓存，用于清除代码其他部分不再使用的对象。如果我们使用普通的HashMap，那么只要键存在于Map中，垃圾收集器就不会清除它。

## 6. 虚引用

与弱引用类似，[虚引用](https://www.baeldung.com/java-phantom-reference)不会禁止垃圾回收器将对象排入队列以进行清除。不同之处在于**虚引用必须在最终确定之前从引用队列中手动轮询**，这意味着我们可以在它们被清除之前决定我们想做什么。

### 6.1 用例

如果我们需要实现一些终结逻辑，虚引用就非常有用，而且它们比[finalize](https://www.baeldung.com/java-finalize)方法更可靠、更灵活。让我们编写一个简单的方法，它将遍历引用队列并对所有引用执行清理：

```java
private static void clearReferences(ReferenceQueue queue) {
    while (true) {
        Reference reference = queue.poll();
        if (reference == null) {
            break; // no references to clear
        }
        cleanup(reference);
    }
}
```

虚引用不允许我们使用get方法检索它们的引用对象。因此，通常的做法是使用我们自己的类来扩展PhantomReference类，其中包含对清理逻辑很重要的信息。

虚引用的其他重要用例是调试和内存泄漏检测，即使我们不需要执行任何终结操作，我们也可以使用虚引用来观察哪些对象正在被释放以及何时被释放。

## 7. 总结

在本文中，我们探讨了硬引用和不同类型的非硬引用及其用例。我们了解到，软引用可用于内存保护，弱引用可用于规范化映射，虚引用可用于细粒度终结。