---
layout: post
title:  Java原始类型Map
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

##   1. 概述

在本教程中，我们将学习**如何使用原始类型键和值构建Map**。

正如我们所知，核心Java [Map](https://www.baeldung.com/java-hashmap)不允许存储原始类型键或值。这就是为什么我们会引入一些提供原始类型Map实现的外部第三方库。

## 2. Eclipse Collections

**[Eclipse Collections](https://www.baeldung.com/eclipse-collections)是Java的高性能集合框架**。它提供了改进的实现以及一些额外的数据结构，包括几个原始集合。

### 2.1 可变和不可变Map

让我们创建一个空Map，其中键和值都是原始int。为此，我们将使用IntIntMaps工厂类：

```java
MutableIntIntMap mutableIntIntMap = IntIntMaps.mutable.empty();
```

**IntIntMaps工厂类是创建原始Map最方便的方法**。它允许我们创建所需Map类型的可变和不可变实例。在我们的示例中，我们创建了IntIntMap的可变实例。类似地，我们可以通过简单地将IntIntMaps.mutable静态工厂调用替换为IntIntMaps.immutable来创建不可变实例：

```java
ImmutableIntIntMap immutableIntIntMap = IntIntMaps.immutable.empty();
```

因此，让我们向可变Map中添加一个键值对：

```java
mutableIntIntMap.addToValue(1, 1);
```

同样，**我们可以创建具有引用和原始类型键值对的混合Map**。让我们创建一个包含String键和double值的Map：

```java
MutableObjectDoubleMap dObject = ObjectDoubleMaps.mutable.empty();
```

在这里，我们使用ObjectDoubleMaps工厂类为MutableObjectDoubleMap创建一个可变实例。

现在让我们添加一些条目：

```java
dObject.addToValue("price", 150.5);
dObject.addToValue("quality", 4.4);
dObject.addToValue("stability", 0.8);
```

### 2.2 原始类型API

在Eclipse Collections中，有一个名为PrimitiveIterable的基本接口。这是库中每个原始类型容器的基本接口。所有这些都被命名为PrimitiveTypeIterable，其中PrimitiveType可以是Int、Long、Short、Byte、Char、Float、Double或Boolean。

反过来，所有这些基础接口都有它们的XYMap实现树，**它根据Map是可变的还是不可变的来划分**。例如，对于IntIntMap，我们有MutableIntIntMap和ImmutableIntIntMap。

最后，正如我们在上面看到的，**我们有接口来涵盖原始类型值和对象值的键和值的所有类型组合**。因此，例如，我们可以将IntObjectMap<K\>用于具有Object值的原始类型键，或者将ObjectIntMap<K\>用于相反的情况。

## 3. HPPC

[HPPC](https://github.com/carrotsearch/hppc)是一个面向高性能和内存效率的库。这意味着该库的抽象程度低于其他库。但是，这样做的好处是可以将内部结构暴露给有用的低级操作。它同时提供Map和集合。

### 3.1 一个简单的例子

让我们首先创建一个具有int键和long值的Map，使用这个是非常熟悉的：

```java
IntLongHashMap intLongHashMap = new IntLongHashMap();
intLongHashMap.put(25, 1L);
intLongHashMap.put(150, Long.MAX_VALUE);
intLongHashMap.put(1, 0L);
        
intLongHashMap.get(150);
```

HPPC为键和值的所有组合提供Map：

-   原始类型键和原始类型值
-   原始类型键和对象类型值
-   对象类型键和原始类型值
-   对象类型键和对象类型值

对象类型Map支持泛型：

```java
IntObjectOpenHashMap<BigDecimal>
ObjectIntOpenHashMap<LocalDate>
```

第一个Map具有原始int键和BigDecimal值，第二个Map的键是LocalDate，值是int

### 3.2 HashMap与ScatterMap

由于键哈希和分布函数的传统实现方式，我们在哈希键时可能会发生冲突。根据键的分布方式，这可能会导致大型Map上的性能问题。默认情况下，HPPC实施了避免此问题的解决方案。

但是，具有更简单分布函数的Map仍然有一席之地。**如果Map用作查找表或用于计数，或者如果它们在加载后不需要大量写入操作，这将很有用**。HPPC提供ScatterMap以进一步提高性能。

所有ScatterMap类都保持与Map相同的命名约定，但改用单词Scatter：

-   IntScatterSet
-   IntIntScatterMap
-   IntObjectScatterMap<BigDecimal\>

## 4. Fastutil

**[Fastutil](https://search.maven.org/search?q=g:it.unimi.dsia:fastutil)是一个快速而紧凑的框架**，它提供特定于类型的集合，包括原始类型Map。

### 4.1 快速示例

类似于Eclipse Collections和HPPC。Fastutil也提供原始类型到原始类型和原始类型到对象类型的关联映射。

让我们创建一个int到boolean Map：

```java
Int2BooleanMap int2BooleanMap = new Int2BooleanOpenHashMap();
```

现在，让我们添加一些条目：

```java
int2BooleanMap.put(1, true);
int2BooleanMap.put(7, false);
int2BooleanMap.put(4, true);
```

然后，我们可以从中检索值：

```java
boolean value = int2BooleanMap.get(1);
```

### 4.2 就地迭代

实现Iterable接口的标准JVM集合通常在每个迭代步骤创建一个新的临时迭代器对象。对于庞大的集合，这可能会产生垃圾回收问题。

Fastutil提供了一种替代方案，可以极大地缓解这种情况：

```java
Int2FloatMap map = new Int2FloatMap();
//Add keys here
for(Int2FloatMap.Entry e : Fastutil.fastIterable(map)) {
    //e will be reused on each iteration, so it will be only one object
}
```

Fastutil还提供了fastForeach方法。这将接收Consumer[函数接口](https://www.baeldung.com/java-8-functional-interfaces)并为每个循环执行Lambda表达式：

```java
Int2FloatMap map = new Int2FloatMap();
//Add keys here
Int2FloatMaps.fastForEach(map , e ->  {
    // e is also reused across iterations
});
```

这与标准的Java foreach结构非常相似：

```java
Int2FloatMap map = new Int2FloatMap();
//Add keys here
map.forEach((key,value) -> {
    // use each key/value entry   
});
```

## 5. 总结

在本文中，我们学习了如何使用Eclipse Collections、HPPC和Fastutil在Java中创建原始Map。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-maps-2)上获得。