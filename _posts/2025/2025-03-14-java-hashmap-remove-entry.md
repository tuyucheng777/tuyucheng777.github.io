---
layout: post
title:  从Java HashMap中删除一个Entry
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 概述

在本文中，我们将讨论从Java HashMap中删除条目的不同方法。

## 2. 简介

HashMap使用具有唯一键的(Key, Value)对来存储条目。因此，一种想法是**使用键作为标识符来从Map中删除相关条目**。

我们可以使用java.util.Map接口提供的方法，以键作为输入来删除条目。

### 2.1 使用方法remove(Object key)

让我们用一个简单的例子来尝试一下，我们有一个将食物与食物类型关联的Map：

```java
HashMap<String, String> foodItemTypeMap = new HashMap<>();
foodItemTypeMap.put("Apple", "Fruit");
foodItemTypeMap.put("Grape", "Fruit");
foodItemTypeMap.put("Mango", "Fruit");
foodItemTypeMap.put("Carrot", "Vegetable");
foodItemTypeMap.put("Potato", "Vegetable");
foodItemTypeMap.put("Spinach", "Vegetable");
```

让我们删除键为“Apple”的条目：

```java
foodItemTypeMap.remove("Apple");
// Current Map Status: {Potato=Vegetable, Carrot=Vegetable, Grape=Fruit, Mango=Fruit, Spinach=Vegetable}
```

### 2.2 使用方法remove(Object key, Object value)

这是第一种方法的变体，接收键和值作为输入。如果我们只想**在键映射到特定值时删除条目**，则使用此方法。

在foodItemTypeMap中，键“Grape”未与值“Vegetable”映射。

因此，以下操作不会导致任何更新：

```java
foodItemTypeMap.remove("Grape", "Vegetable");
// Current Map Status: {Potato=Vegetable, Carrot=Vegetable, Grape=Fruit, Mango=Fruit, Spinach=Vegetable}
```

现在，让我们探索HashMap中删除条目的其他场景。

## 3. 迭代时删除条目

**HashMap类是不同步的**，如果我们尝试同时添加或删除一个条目，则可能导致ConcurrentModificationException。因此，我们**需要在外部同步remove操作**。

### 3.1 外部对象同步

一种方法是**在封装HashMap的对象上进行同步**，例如，我们可以使用java.util.Map接口的entrySet()方法来获取HashMap中的条目集合，返回的Set由关联的Map支持。

**因此，Set的任何结构修改都会导致Map的更新**。 

让我们使用此方法从foodItemTypeMap中删除一个条目：

```java
Iterator<Entry<String, String>> iterator = foodItemTypeMap.entrySet().iterator();
while (iterator.hasNext()) {
    if (iterator.next().getKey().equals("Carrot"))
        iterator.remove();
}
```

除非我们使用迭代器自己的方法进行更新，否则可能不支持对Map进行结构修改。如上代码片段所示，我们**调用迭代器对象上的remove()方法，而不是Map上的**，这提供了线程安全的删除操作。

**我们可以在Java 8或更高版本中使用removeIf操作实现相同的结果**：

```java
foodItemTypeMap.entrySet()
  .removeIf(entry -> entry.getKey().equals("Grape"));
```

### 3.2 使用ConcurrentHashMap<K, V\>

**java.util.concurrent.ConcurrentHashMap类提供线程安全操作**，ConcurrentHashMap的迭代器每次只使用一个线程。因此，它们为并发操作提供了确定性行为。

我们可以使用ConcurrencyLevel指定允许的并发线程操作数。

让我们使用基本的remove方法来删除ConcurrentHashMap中的条目：

```java
ConcurrentHashMap<String, String> foodItemTypeConcMap = new ConcurrentHashMap<>();
foodItemTypeConcMap.put("Apple", "Fruit");
foodItemTypeConcMap.put("Carrot", "Vegetable");
foodItemTypeConcMap.put("Potato", "Vegetable");

for (Entry<String, String> item : foodItemTypeConcMap.entrySet()) {
    if (item.getKey() != null && item.getKey().equals("Potato")) {
        foodItemTypeConcMap.remove(item.getKey());
    }
}
```

## 4. 总结

本文中探讨了Java HashMap中条目删除的不同场景，如果不进行迭代，我们可以安全地使用java.util.Map接口提供的标准条目删除方法。

如果我们要在迭代过程中更新Map，则必须在封装对象上使用remove方法。此外，我们还分析了替代类ConcurrentHashMap，该类支持对Map进行线程安全的更新操作。