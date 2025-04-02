---
layout: post
title:  Java中的虚引用
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

在本文中，我们将了解Java语言中的虚引用的概念。

## 2. 虚引用

虚引用与[软引用](https://www.baeldung.com/java-soft-references)和[弱引用](https://www.baeldung.com/java-weak-reference)有两个主要区别。

**我们无法获得虚引用的引用对象**，引用对象永远无法通过API直接访问，这就是为什么我们需要一个引用队列来处理这种类型的引用。

**垃圾回收器在其引用对象的finalize方法执行后将虚引用添加到引用队列**，这意味着该实例仍在内存中。

## 3. 用例

它们有两种常见用途。

第一种技术是**确定对象何时从内存中移除**，这有助于调度内存敏感型任务。例如，我们可以等待一个大对象被移除，然后再加载另一个。

第二种做法是**避免使用finalize方法**，改进finalization过程。

### 3.1 例子

现在，让我们实现第二个用例，实际弄清楚这种引用是如何工作的。

首先，我们需要[PhantomReference](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/ref/PhantomReference.html)类的子类来定义清除资源的方法：

```java
public class LargeObjectFinalizer extends PhantomReference<Object> {

    public LargeObjectFinalizer(
            Object referent, ReferenceQueue<? super Object> q) {
        super(referent, q);
    }

    public void finalizeResources() {
        // free resources
        System.out.println("clearing ...");
    }
}
```

现在我们要编写一个增强的细粒度终结：

```java
ReferenceQueue<Object> referenceQueue = new ReferenceQueue<>();
List<LargeObjectFinalizer> references = new ArrayList<>();
List<Object> largeObjects = new ArrayList<>();

for (int i = 0; i < 10; ++i) {
    Object largeObject = new Object();
    largeObjects.add(largeObject);
    references.add(new LargeObjectFinalizer(largeObject, referenceQueue));
}

largeObjects = null;
System.gc();

Reference<?> referenceFromQueue;
for (PhantomReference<Object> reference : references) {
    System.out.println(reference.isEnqueued());
}

while ((referenceFromQueue = referenceQueue.poll()) != null) {
    ((LargeObjectFinalizer)referenceFromQueue).finalizeResources();
    referenceFromQueue.clear();
}
```

首先，我们初始化所有必要的对象：referenceQueue-跟踪排队的引用，references-之后执行清理工作，largeObjects-模拟大型数据结构。

接下来，我们使用Object和LargeObjectFinalizer类创建这些对象。

在调用垃圾回收器之前，我们通过取消引用largeObjects列表来手动释放大量数据。请注意，我们使用Runtime.getRuntime().gc()语句的快捷方式来调用垃圾回收器。

重要的是要知道System.gc()不会立即触发垃圾回收-它只是提示JVM触发该过程。

for循环演示了如何确保所有引用都已入队-它将为每个引用打印出true。

最后，我们使用while循环来轮询排队的引用并对它们中的每一个进行清理工作。

## 4. 总结

在这个快速教程中，我们介绍了Java的虚引用。

我们通过一些简单且切中要点的例子了解了这些是什么以及它们如何有用。