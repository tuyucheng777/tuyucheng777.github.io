---
layout: post
title:  Eclipse Collections中的原始集合
category: libraries
copyright: libraries
excerpt: Eclipse Collections
---

## 1. 简介

在本教程中，我们将讨论Java中的基本类型集合以及[Eclipse Collections](https://www.baeldung.com/eclipse-collections)如何提供帮助。

## 2. 动机

假设我们要创建一个简单的整数列表：

```java
List<Integer> myList = new ArrayList<>; 
int one = 1; 
myList.add(one);
```

由于集合只能保存对象引用，因此在底层，该过程会将对象转换为Integer。当然，**装箱和拆箱不是免费的，因此，此过程中存在[性能损失](https://www.baeldung.com/java-list-primitive-performance)**。

因此，首先，使用Eclipse Collections中的原始集合可以提高速度。

其次，它减少了内存占用。下图比较了传统ArrayList和Eclipse Collections中的IntArrayList之间的内存使用情况：

![](/assets/images/2025/libraries/javaeclipseprimitivecollections01.png)

图片摘自https://www.eclipse.org/collections/#concept

当然，我们不要忘记，实现的多样性是Eclipse Collections的一大卖点。

还要注意的是，Java到目前为止还不支持原始集合。不过，**Project Valhalla计划通过[JEP 218](https://openjdk.java.net/jeps/218)添加该功能**。

## 3. 依赖

我们将使用[Maven](https://mvnrepository.com/artifact/org.eclipse.collections)来包含所需的依赖：

```xml
<dependency>
    <groupId>org.eclipse.collections</groupId>
    <artifactId>eclipse-collections-api</artifactId>
    <version>10.0.0</version>
</dependency>
<dependency>
    <groupId>org.eclipse.collections</groupId>
    <artifactId>eclipse-collections</artifactId>
    <version>10.0.0</version>
</dependency>
```

## 4. long列表

**Eclipse Collections为所有[基本类型](https://www.baeldung.com/java-primitives)提供了内存优化的列表、集合、堆栈、Map和Bag**，让我们来看几个例子。

首先，我们来看一个long列表：

```java
@Test
public void whenListOfLongHasOneTwoThree_thenSumIsSix() {
    MutableLongList longList = LongLists.mutable.of(1L, 2L, 3L);
    assertEquals(6, longList.sum());
}
```

## 5. int列表

同样，我们可以创建一个不可变的int列表：

```java
@Test
public void whenListOfIntHasOneTwoThree_thenMaxIsThree() {
    ImmutableIntList intList = IntLists.immutable.of(1, 2, 3);
    assertEquals(3, intList.max());
}
```

## 6. Map

除了Map接口方法之外，Eclipse Collections还为每个原始类型配对提供了新方法：

```java
@Test
public void testOperationsOnIntIntMap() {
    MutableIntIntMap map = new IntIntHashMap();
    assertEquals(5, map.addToValue(0, 5));
    assertEquals(5, map.get(0));
    assertEquals(3, map.getIfAbsentPut(1, 3));
}
```

## 7. 从Iterable到原始集合

此外，Eclipse Collections还可以与Iterable配合使用：

```java
@Test
public void whenConvertFromIterableToPrimitive_thenValuesAreEqual() {
    Iterable<Integer> iterable = Interval.oneTo(3);
    MutableIntSet intSet = IntSets.mutable.withAll(iterable);
    IntInterval intInterval = IntInterval.oneTo(3);
    assertEquals(intInterval.toSet(), intSet);
}
```

此外，我们可以从Iterable创建一个原始Map：

```java
@Test
public void whenCreateMapFromStream_thenValuesMustMatch() {
    Iterable<Integer> integers = Interval.oneTo(3);
    MutableIntIntMap map =
            IntIntMaps.mutable.from(
                    integers,
                    key -> key,
                    value -> value * value);
    MutableIntIntMap expected = IntIntMaps.mutable.empty()
            .withKeyValue(1, 1)
            .withKeyValue(2, 4)
            .withKeyValue(3, 9);
    assertEquals(expected, map);
}
```

## 8. 原始类型流

由于Java已经带有原始类型流，并且Eclipse Collections可以与它们很好地集成：

```java
@Test
public void whenCreateDoubleStream_thenAverageIsThree() {
    DoubleStream doubleStream = DoubleLists
            .mutable.with(1.0, 2.0, 3.0, 4.0, 5.0)
            .primitiveStream();
    assertEquals(3, doubleStream.average().getAsDouble(), 0.001);
}
```

## 9. 总结

本教程介绍了Eclipse Collections中的原始类型集合，我们展示了使用它的理由，并展示了如何轻松地将其添加到我们的应用程序中。