---
layout: post
title:  使用自定义Supplier将List转换为Map
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在本教程中，我们将把List<E\>转换为Map<K, List<E\>>。**我们将使用[Java的Stream API](https://www.baeldung.com/java-8-streams)和[Supplier函数接口](https://www.baeldung.com/java-8-functional-interfaces)来实现这一点**。

## 2. JDK 8中的Supplier

Supplier通常用作工厂。一个方法可以将Supplier作为输入并使用有界通配符类型来约束类型，然后客户端可以传入一个工厂来创建给定类型的任何子类型。

除此之外，Supplier可以执行[延迟生成值](https://www.baeldung.com/guava-memoizer)。

## 3. List转Map

Stream API提供对List操作的支持，**Stream#collect方法就是这样的一个例子**。但是，在Stream API方法中没有办法直接向Supplier提供下游参数。

在本教程中，我们将通过示例代码片段了解[Collectors.groupingBy](https://www.baeldung.com/java-groupingby-collector)、[Collectors.toMap](https://www.baeldung.com/java-collectors-tomap)和[Stream.collect](https://www.baeldung.com/java-8-collectors)方法。**我们将重点介绍允许我们使用自定义Supplier的方法**。

在本教程中，我们将在以下示例中处理字符串列表集合：

```java
List source = Arrays.asList("List", "Map", "Set", "Tree");
```

我们将把上面的列表聚合成一个Map，其键是字符串的长度。完成后，我们将得到一个如下所示的Map：

```text
{
    3: ["Map", "Set"],
    4: ["List", "Tree"]
}
```

### 3.1 Collectors.groupingBy()

使用Collectors.groupingBy，我们可以将Collection转换为具有特定分类器的Map。分类器是一个元素的属性，我们将使用此属性将元素合并到不同的组中：

```java
public Map<Integer, List> groupingByStringLength(List source, 
    Supplier<Map<Integer, List>> mapSupplier, 
    Supplier<List> listSupplier) {
    return source.stream()
        .collect(Collectors.groupingBy(String::length, mapSupplier, Collectors.toCollection(listSupplier)));
}
```

我们可以验证它是否适用于：

```java
Map<Integer, List> convertedMap = converter.groupingByStringLength(source, HashMap::new, ArrayList::new);
assertTrue(convertedMap.get(3).contains("Map"));
```

### 3.2 Collectors.toMap()

Collectors.toMap方法将流中的元素归约为Map。

我们首先使用源字符串以及List和Map Supplier定义方法：

```java
public Map<Integer, List> collectorToMapByStringLength(List source, 
    Supplier<Map<Integer, List>> mapSupplier, 
    Supplier<List> listSupplier)
```

然后我们定义如何从元素中获取键和值。为此，我们使用了两个新函数：

```java
Function<String, Integer> keyMapper = String::length;

Function<String, List> valueMapper = (element) -> {
    List collection = listSupplier.get();
    collection.add(element);
    return collection;
};
```

最后，我们定义一个在键冲突时调用的函数。在这种情况下，我们希望结合两者的内容：

```java
BinaryOperator<List> mergeFunction = (existing, replacement) -> {
    existing.addAll(replacement);
    return existing;
};
```

将所有内容放在一起，我们得到：

```java
source.stream().collect(Collectors.toMap(keyMapper, valueMapper, mergeFunction, mapSupplier))
```

请注意，大多数时候我们定义的函数是方法参数列表中的匿名内联函数。

让我们测试一下：

```java
Map<Integer, List> convertedMap = converter.collectorToMapByStringLength(source, HashMap::new, ArrayList::new);
assertTrue(convertedMap.get(3).contains("Map"));
```

### 3.3 Stream.collect()

Stream.collect方法可用于将流中的元素归约为任何类型的Collection。

为此，我们还需要为List和Map Supplier定义一个方法，一旦需要一个新的集合就会调用该方法：

```java
public Map<Integer, List> streamCollectByStringLength(List source, 
    Supplier<Map<Integer, List>> mapSupplier, 
    Supplier<List> listSupplier)
```

然后我们定义一个累加器accumulator，给定元素的键，获取现有列表或创建一个新列表，并将元素添加到响应中：

```java
BiConsumer<Map<Integer, List>, String> accumulator = (response, element) -> {
    Integer key = element.length();
    List values = response.getOrDefault(key, listSupplier.get());
    values.add(element);
    response.put(key, values);
};
```

最后，我们将累加器函数生成的值组合在一起：

```java
BiConsumer<Map<Integer, List>, Map<Integer, List>> combiner = (res1, res2) -> {
    res1.putAll(res2);
};
```

将所有内容放在一起，然后我们只需在元素流上调用collect方法：

```java
source.stream().collect(mapSupplier, accumulator, combiner);
```

请注意，大多数时候我们定义的函数是方法参数列表中的匿名内联函数。

测试结果将与其他两种方法相同：

```java
Map<Integer, List> convertedMap = converter.streamCollectByStringLength(source, HashMap::new, ArrayList::new);
assertTrue(convertedMap.get(3).contains("Map"));
```

## 4. 总结

在本教程中，我们演示了如何使用带有自定义Supplier的Java 8 Stream API将List<E\>转换为Map<K, List<E\>>。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-conversions-2)上获得。