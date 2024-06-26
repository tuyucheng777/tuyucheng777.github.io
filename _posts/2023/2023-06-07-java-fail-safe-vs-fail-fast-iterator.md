---
layout: post
title:  安全失败迭代器与快速失败迭代器
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在本文中，我们将介绍快速失败和安全失败迭代器的概念。

**快速失败系统尽可能快地中止操作，立即暴露故障并停止整个操作**。

**而安全失败系统在发生故障时不会中止操作，这样的系统会尽量避免引发故障**。

## 2. 快速失败迭代器

当基础集合被修改时，Java中的快速失败迭代器不会发挥作用。

集合维护一个名为modCount的内部计数器。每次从Collection添加或删除元素时，此计数器都会递增。

迭代时，在每次调用next()时，会将modCount的当前值与初始值进行比较。如果不匹配，它会抛出ConcurrentModificationException，从而中止整个操作。

**来自java.util包的集合(例如ArrayList、HashMap等)的默认迭代器是快速失败的**。

```java
ArrayList<Integer> numbers = // ...

Iterator<Integer> iterator = numbers.iterator();
while (iterator.hasNext()) {
    Integer number = iterator.next();
    numbers.add(50);
}
```

在上面的代码片段中，ConcurrentModificationException在执行修改后的下一个迭代周期开始时被抛出。

快速失败行为不能保证在所有情况下都会发生，因为在并发修改的情况下无法预测行为。**这些迭代器在尽力而为的基础上抛出ConcurrentModificationException**。

**如果在集合的迭代过程中，使用Iterator的remove()方法删除了一个元素，那是完全安全的并且不会抛出异常**。

但是，如果Collection的remove()方法用于删除元素，则会抛出异常：

```java
ArrayList<Integer> numbers = // ...

Iterator<Integer> iterator = numbers.iterator();
while (iterator.hasNext()) {
    if (iterator.next() == 30) {
        iterator.remove(); // ok!
    }
}

iterator = numbers.iterator();
while (iterator.hasNext()) {
    if (iterator.next() == 40) {
        numbers.remove(2); // exception
    }
}
```

## 3. 安全失败迭代器

安全失败迭代器倾向于避免失败而不是异常处理的不便。

这些迭代器创建实际集合的克隆并对其进行迭代。如果在创建迭代器后发生任何修改，副本仍然保持不变。因此，这些迭代器会继续遍历集合，即使它已被修改。

但是，请务必记住，真正的安全失败迭代器是不存在的。正确的术语是弱一致性。

这意味着，**如果一个Collection在被迭代时被修改，那么Iterator看到的是弱保证**。此行为对于不同的Collection可能不同，并且记录在每个此类Collection的Javadoc中。

不过，安全失败迭代器有一些缺点。**一个缺点是Iterator不能保证从Collection返回更新的数据**，因为它处理克隆而不是实际集合。

另一个缺点是创建集合副本的开销，包括时间和内存。

**来自java.util.concurrent包的集合的迭代器，如ConcurrentHashMap、CopyOnWriteArrayList等，本质上是安全失败的**。

```java
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();

map.put("First", 10);
map.put("Second", 20);
map.put("Third", 30);
map.put("Fourth", 40);

Iterator<String> iterator = map.keySet().iterator();

while (iterator.hasNext()) {
    String key = iterator.next();
    map.put("Fifth", 50);
}
```

在上面的代码片段中，我们使用了安全失败迭代器。因此，即使在迭代期间将新元素添加到集合中，它也不会抛出异常。

ConcurrentHashMap的默认迭代器是弱一致性的。这意味着此迭代器可以容忍并发修改，遍历构造Iterator时存在的元素，并且可能(但不保证)在构造Iterator后反映对Collection的修改。

因此，在上面的代码片段中，迭代循环了五次，这意味着**它确实检测到新添加到集合中的元素**。

## 4. 总结

在本教程中，我们了解了快速失败和安全失败迭代器的含义以及它们是如何在Java中实现的。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-3)上获得。