---
layout: post
title:  如何在Java中反转Map
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在这篇简短的文章中，我们将介绍**如何在Java中反转Map**。这个想法是为Map<K, V\>类型的给定Map创建Map<V, K\>的新实例。此外，我们还将了解如何处理源Map中存在重复值的情况。

请参阅我们的另一篇文章以了解有关[HashMap](https://www.baeldung.com/java-hashmap)类本身的更多信息。

## 2. 定义问题

假设我们有一个包含几个键值对的Map：

```java
Map<String, Integer> map = new HashMap<>();
map.put("first", 1);
map.put("second", 2);
```

原始Map将存储以下元素：

```text
{first=1, second=2}
```

相反，**我们希望将键转换为值，将值转换为键**。结果将是：

```text
{1=first, 2=second}
```

## 3. 使用传统的for循环

首先，让我们看看如何使用for循环反转Map：

```java
public static <V, K> Map<V, K> invertMapUsingForLoop(Map<K, V> map) {
    Map<V, K> inversedMap = new HashMap<V, K>();
    for (Entry<K, V> entry : map.entrySet()) {
        inversedMap.put(entry.getValue(), entry.getKey());
    }
    return inversedMap;
}
```

在这里，我们遍历Map对象的entrySet()。之后，我们将原来的Value作为新的Key，将原来的Key作为新的Value添加到inversedMap对象中。换句话说，**我们通过用值替换键和用键替换值来复制Map的内容**。此外，这适用于8之前的Java版本，但我们应该注意，**这种方法仅在源Map的值是唯一的情况下才有效**。

## 4. 使用Stream API反转Map

Java 8提供了来自Stream API的便捷方法，以更实用的方式反转Map。让我们来看看其中的几个。

### 4.1 Collectors.toMap()

**如果源Map中没有任何重复值，我们可以使用Collectors.toMap()**：

```java
public static <V, K> Map<V, K> invertMapUsingStreams(Map<K, V> map) {
    Map<V, K> inversedMap = map.entrySet()
        .stream()
        .collect(Collectors.toMap(Entry::getValue, Entry::getKey));
    return inversedMap;
}
```

首先，entrySet()被转换为对象流。随后，**我们使用Collectors.toMap()将Key和Value收集到inversedMap对象中**。

让我们考虑源Map包含重复值。在这种情况下，我们可以使用映射函数将自定义规则应用于输入元素：

```java
public static <K, V> Map<V, K> invertMapUsingMapper(Map<K, V> sourceMap) {
    return sourceMap.entrySet()
        .stream().collect(
            Collectors.toMap(Entry::getValue, Entry::getKey, (oldValue, newValue) -> oldValue) 
        );
}
```

在此方法中，Collectors.toMap()的最后一个参数是一个映射函数。**使用它，我们可以自定义在有重复的情况下应该添加哪个键**。在上面的示例中，如果源Map包含重复值，我们将为键保留第一个值。但是，如果值重复，我们只能保留一个键。

### 4.2 Collectors.groupingBy()

有时，即使源Map包含重复值，我们也可能需要所有键。或者，**Collectors.groupingBy()可以更好地控制重复值的处理**。

例如，假设我们有以下键值对：

```text
{first=1, second=2, two=2}
```

此处，值“2”针对不同的键重复两次。在这些情况下，我们可以使用groupingBy()方法对Value对象实现级联的“group by”操作：

```java
private static <V, K> Map<V, List<K>> invertMapUsingGroupingBy(Map<K, V> map) {
    Map<V, List<K>> inversedMap = map.entrySet()
        .stream()
        .collect(Collectors.groupingBy(Map.Entry::getValue, Collectors.mapping(Map.Entry::getKey, Collectors.toList())));
    return inversedMap;
}
```

稍微解释一下，Collectors.mapping()函数使用指定的收集器对与给定键关联的值执行归约操作。groupingBy()收集器将重复值收集到List中，从而生成MultiMap。现在的输出将是：

```text
{1=[first], 2=[two, second]}
```

## 5. 总结

在本文中，我们通过示例快速回顾了几种反转HashMap的内置方法。此外，我们还了解了在反转Map对象时如何处理重复值。

同时，一些外部库在Map接口之上提供了额外的功能。我们之前已经演示了如何使用[Google Guava BiMap](https://www.baeldung.com/apache-commons-collections-vs-guava#bimap)和[Apache BidiMap](https://baeldung.com/apache-commons-collections-vs-guava#bidimap)反转Map。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-maps-5)上获得。