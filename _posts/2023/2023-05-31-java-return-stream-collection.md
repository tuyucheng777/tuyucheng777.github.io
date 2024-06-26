---
layout: post
title:  返回Stream vs Collection
category: java-stream
copyright: java-stream
excerpt: Java Stream
---

## 1. 概述

[Java Stream API](https://www.baeldung.com/java-8-streams-introduction)提供了一种替代[Java Collection](https://www.baeldung.com/java-collections)的高效方法来呈现或处理结果集。但是，决定何时使用哪一个是一个常见的难题。

在本文中，我们将探讨Stream和Collection，并讨论适合它们各自用途的各种场景。

## 2. Collection与Stream

Java Collection通过提供[List](https://www.baeldung.com/java-linkedlist)、[Set](https://www.baeldung.com/java-hashset)和[Map](https://www.baeldung.com/java-hashmap)等数据结构来提供存储和处理数据的有效机制。

但是，Stream API对于在不需要中间存储的情况下对数据执行各种操作非常有用。因此，Stream的工作方式类似于直接从底层存储(如集合和[I/O资源](https://www.baeldung.com/java-io))访问数据。

此外，集合主要关注提供对数据的访问和修改数据的方法。另一方面，流关注的是高效地传输数据。

尽管Java允许从Collection到Stream的轻松转换，反之亦然，但了解哪种是呈现/处理结果集的最佳机制还是很方便的。

例如，我们可以使用stream和parallelStream方法将Collection转换为Stream：

```java
public Stream<String> userNames() {
    ArrayList<String> userNameSource = new ArrayList<>();
    userNameSource.add("john");
    userNameSource.add("smith");
    userNameSource.add("tom");
    return userNames.stream();
}
```

同样，我们可以[使用Stream API的collect方法将Stream转换为Collection](https://www.baeldung.com/java-8-collectors#Collect)：

```java
public List<String> userNameList() {
    return userNames().collect(Collectors.toList());
}
```

在这里，我们[使用Collectors.toList()方法将Stream转换为List](https://www.baeldung.com/java-8-collectors#1-collectorstolist)。同样，我们可以将[Stream转换为Set](https://www.baeldung.com/java-8-collectors#2-collectorstoset)或[Map](https://www.baeldung.com/java-8-collectors#4-collectorstomap)：

```java
public static Set<String> userNameSet() {
    return userNames().collect(Collectors.toSet());
}

public static Map<String, String> userNameMap() {
    return userNames().collect(Collectors.toMap(u1 -> u1.toString(), u1 -> u1.toString()));
}
```

## 3. 什么时候返回流？

### 3.1 物化成本高

Stream API提供延迟执行和移动中的结果过滤，这是降低物化成本的最有效方法。

例如，[Java NIO Files类中的readAllLines](https://www.baeldung.com/reading-file-in-java#read-file-with-path-readalllines)方法呈现文件的所有行，为此JVM必须将整个文件内容保存在内存中。因此，此方法在返回行列表时具有很高的具体化成本。

但是，[Files类还提供了返回Stream的lines方法](https://www.baeldung.com/reading-file-in-java#%20id=)，我们可以使用它来呈现所有行，甚至可以使用limit方法更好地限制结果集的大小-两者都具有延迟执行：

```java
Files.lines(path).limit(10).collect(toList());
```

此外，Stream不会执行中间操作，直到我们对其调用诸如[forEach之类的终端操作](https://www.baeldung.com/java-collection-stream-foreach)：

```java
userNames().filter(i -> i.length() >= 4).forEach(System.out::println);
```

因此，Stream避免了与过早实现相关的成本。

### 3.2 大或无限结果

Stream旨在获得更大或无限结果的更好性能。因此，在这种用例中使用Stream始终是一个好主意。

此外，在无限结果的情况下，我们通常不会处理整个结果集。因此，事实证明，Stream API的内置功能(如filter和limit)在处理所需结果集时非常方便，使Stream成为更可取的选择。

### 3.3 灵活性

流非常灵活，允许以任何形式或顺序处理结果。

当我们不想向使用者强制执行一致的结果集时，Stream是一个显而易见的选择。此外，当我们想要为使用者提供急需的灵活性时，Stream是一个不错的选择。

例如，我们可以使用Stream API上可用的各种操作来过滤/排序/限制结果：

```java
public static Stream<String> filterUserNames() {
    return userNames().filter(i -> i.length() >= 4);
}

public static Stream<String> sortUserNames() {
    return userNames().sorted();
}

public static Stream<String> limitUserNames() {
    return userNames().limit(3);
}
```

### 3.4 函数式行为

Stream是函数式的。当以不同方式处理时，它不允许对源进行任何修改。因此，呈现不可变结果集是首选。

例如，让我们过滤并限制从主Stream接收到的一组结果：

```java
userNames().filter(i -> i.length() >= 4).limit(3).forEach(System.out::println);
```

在这里，对Stream的filter和limit等操作每次都会返回一个新的Stream，并且不会修改userNames方法提供的源Stream。

## 4. 什么时候返回集合？

### 4.1. 物化成本低

在渲染或处理涉及低物化成本的结果时，我们可以选择集合而不是流。

换句话说，Java通过在开始时计算所有元素来急切地构造一个Collection。因此，具有大结果集的Collection会给物化中的堆内存带来很大压力。

因此，我们应该考虑使用Collection来呈现一个结果集，该结果集不会为其物化对堆内存造成太大压力。

### 4.2 固定格式

我们可以使用Collection为用户强制执行一致的结果集。例如，像[TreeSet](https://www.baeldung.com/java-tree-set)和[TreeMap](https://www.baeldung.com/java-treemap)这样的Collection返回自然排序的结果。

换句话说，通过使用Collection，我们可以确保每个使用者以相同的顺序接收和处理相同的结果集。

### 4.3 可重复使用的结果

当结果以集合的形式返回时，它可以很容易地被多次遍历。但是，[Stream一旦被遍历就被视为已消耗，并且在重用时抛出IllegalStateException](https://www.baeldung.com/java-stream-operated-upon-or-closed-exception)：

```java
public static void tryStreamTraversal() {
    Stream<String> userNameStream = userNames();
    userNameStream.forEach(System.out::println);
    
    try {
        userNameStream.forEach(System.out::println);
    } catch(IllegalStateException e) {
        System.out.println("stream has already been operated upon or closed");
    }
}
```

因此，当使用者显然会多次遍历结果时，返回Collection是更好的选择。

### 4.4 修改

与Stream不同，集合允许修改元素，例如从结果源中添加或删除元素。因此，我们可以考虑使用集合返回结果集以允许使用者修改。

例如，我们可以使用add/remove方法修改ArrayList：

```java
userNameList().add("bob");
userNameList().add("pepper");
userNameList().remove(2);
```

同样，像put和remove这样的方法允许在Map上进行修改：

```java
Map<String, String> userNameMap = userNameMap();
userNameMap.put("bob", "bob");
userNameMap.remove("alfred");
```

### 4.5 内存中结果

此外，当集合形式的具体化结果已存在于内存中时，使用集合是一个显而易见的选择。

## 5. 总结

在本文中，我们比较了Stream与Collection，并研究了适合它们的各种场景。

我们可以得出总结，Stream是呈现大型或无限结果集的理想选择，具有延迟初始化、急需的灵活性和函数式行为等优点。

但是，当我们需要一致形式的结果时，或者当涉及低具体化时，我们应该选择Collection而不是Stream。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-streams-3)上获得。
