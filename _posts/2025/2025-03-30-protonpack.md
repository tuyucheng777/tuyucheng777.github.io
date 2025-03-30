---
layout: post
title:  Protonpack简介
category: libraries
copyright: libraries
excerpt: Protonpack
---

## 1. 概述

在本教程中，我们将了解[Protonpack](https://github.com/poetix/protonpack)的主要功能，它是一个通过添加一些补充功能来扩展标准[Stream API](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Stream.html#takeWhile(java.util.function.Predicate))的库。

请参阅[此处的文章](https://www.baeldung.com/java-8-streams-introduction)来了解Java Stream API的基础知识。

## 2. Maven依赖

要使用Protonpack库，我们需要在pom.xml文件中添加依赖：

```xml
<dependency>
    <groupId>com.codepoetics</groupId>
    <artifactId>protonpack</artifactId>
    <version>1.15</version>
</dependency>
```

在[Maven Central](https://mvnrepository.com/artifact/com.codepoetics/protonpack)上检查最新版本。

## 3. StreamUtils

这是扩展Java标准Stream API的主要类。

这里讨论的所有方法都是[中间操作](https://www.baeldung.com/java-8-streams-introduction#operations)，这意味着它们修改Stream但不会触发其处理。

### 3.1 takeWhile()和takeUntil()

takeWhile()**只要满足提供的条件**，就会从源流中获取值：

```java
Stream<Integer> streamOfInt = Stream
    .iterate(1, i -> i + 1);
List<Integer> result = StreamUtils
    .takeWhile(streamOfInt, i -> i < 5)
    .collect(Collectors.toList());
assertThat(result).contains(1, 2, 3, 4);
```

相反，takeUntil()会**接收值直到某个值满足提供的条件**然后停止：

```java
Stream<Integer> streamOfInt = Stream
    .iterate(1, i -> i + 1);
List<Integer> result = StreamUtils
    .takeUntil(streamOfInt, i -> i >= 5)
    .collect(Collectors.toList());
assertThat(result).containsExactly(1, 2, 3, 4);
```

从Java 9开始，takeWhile()是标准[Stream API](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Stream.html#takeWhile(java.util.function.Predicate))的一部分。

### 3.2 zip()

zip()接收2个或3个流作为输入和一个组合器函数，**该方法从每个流的相同位置获取一个值并将其传递给组合器**。

它会一直这样做，直到其中一个流的值用完：

```java
String[] clubs = {"Juventus", "Barcelona", "Liverpool", "PSG"};
String[] players = {"Ronaldo", "Messi", "Salah"};
Set<String> zippedFrom2Sources = StreamUtils
    .zip(stream(clubs), stream(players), (club, player) -> club + " " + player)
    .collect(Collectors.toSet());
 
assertThat(zippedFrom2Sources)
    .contains("Juventus Ronaldo", "Barcelona Messi", "Liverpool Salah");
```

类似地，重载的zip()需要3个源流：

```java
String[] leagues = { "Serie A", "La Liga", "Premier League" };
Set<String> zippedFrom3Sources = StreamUtils
    .zip(stream(clubs), stream(players), stream(leagues), 
        (club, player, league) -> club + " " + player + " " + league)
    .collect(Collectors.toSet());
 
assertThat(zippedFrom3Sources).contains(
    "Juventus Ronaldo Serie A", 
    "Barcelona Messi La Liga", 
    "Liverpool Salah Premier League");
```

### 3.3 zipWithIndex()

zipWithIndex()**获取值并将每个值与其索引压缩以创建索引值流**：

```java
Stream<String> streamOfClubs = Stream
    .of("Juventus", "Barcelona", "Liverpool");
Set<Indexed<String>> zipsWithIndex = StreamUtils
    .zipWithIndex(streamOfClubs)
    .collect(Collectors.toSet());
assertThat(zipsWithIndex)
    .contains(Indexed.index(0, "Juventus"), Indexed.index(1, "Barcelona"), 
        Indexed.index(2, "Liverpool"));
```

### 3.4 merge()

merge()与多个源流和一个合并器一起工作，**它从每个源流中获取相同索引位置的值并将其传递给合并器**。

该方法的工作原理是从种子值开始，依次从每个流中的相同索引中获取1个值。

然后将值传递给组合器，并将得到的组合值反馈给组合器以创建下一个值：

```java
Stream<String> streamOfClubs = Stream
    .of("Juventus", "Barcelona", "Liverpool", "PSG");
Stream<String> streamOfPlayers = Stream
    .of("Ronaldo", "Messi", "Salah");
Stream<String> streamOfLeagues = Stream
    .of("Serie A", "La Liga", "Premier League");

Set<String> merged = StreamUtils.merge(
    () ->  "",
    (valOne, valTwo) -> valOne + " " + valTwo,
    streamOfClubs,
    streamOfPlayers,
    streamOfLeagues)
    .collect(Collectors.toSet());

assertThat(merged)
    .contains("Juventus Ronaldo Serie A", "Barcelona Messi La Liga", "Liverpool Salah Premier League", "PSG");
```

### 3.5 mergeToList()

mergeToList()接收多个流作为输入，**它将每个流中相同索引的值组合成一个List**：

```java
Stream<String> streamOfClubs = Stream
    .of("Juventus", "Barcelona", "PSG");
Stream<String> streamOfPlayers = Stream
    .of("Ronaldo", "Messi");

Stream<List<String>> mergedStreamOfList = StreamUtils
    .mergeToList(streamOfClubs, streamOfPlayers);
List<List<String>> mergedListOfList = mergedStreamOfList
    .collect(Collectors.toList());

assertThat(mergedListOfList.get(0))
    .containsExactly("Juventus", "Ronaldo");
assertThat(mergedListOfList.get(1))
    .containsExactly("Barcelona", "Messi");
assertThat(mergedListOfList.get(2))
    .containsExactly("PSG");
```

### 3.6 interleave()

interleave()**使用选择器从多个流中获取创建替代值**。

该方法将包含来自每个流的一个值的集合提供给选择器，然后选择器将选择一个值。

然后，所选值将从集合中移除，并替换为所选值的下一个值。此迭代持续进行，直到所有源的值都用完为止。

下一个示例使用interleave()以循环策略创建交替值：

```java
Stream<String> streamOfClubs = Stream
    .of("Juventus", "Barcelona", "Liverpool");
Stream<String> streamOfPlayers = Stream
    .of("Ronaldo", "Messi");
Stream<String> streamOfLeagues = Stream
    .of("Serie A", "La Liga");

List<String> interleavedList = StreamUtils
    .interleave(Selectors.roundRobin(), streamOfClubs, streamOfPlayers, streamOfLeagues)
    .collect(Collectors.toList());
  
assertThat(interleavedList)
    .hasSize(7)
    .containsExactly("Juventus", "Ronaldo", "Serie A", "Barcelona", "Messi", "La Liga", "Liverpool");
```

请注意，上述代码仅用于教程目的，因为循环选择器由库以[Selectors.roundRobin()](https://github.com/poetix/protonpack/blob/master/src/main/java/com/codepoetics/protonpack/selectors/Selectors.java)的形式提供。

### 3.7 skipUntil()和skipWhile()

skipUntil()**跳过值直到某个值满足条件**：

```java
Integer[] numbers = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
List skippedUntilGreaterThan5 = StreamUtils
    .skipUntil(stream(numbers), i -> i > 5)
    .collect(Collectors.toList());
 
assertThat(skippedUntilGreaterThan5).containsExactly(6, 7, 8, 9, 10);
```

相反，skipWhile()**会在值满足条件时跳过该值**：

```java
Integer[] numbers = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
List skippedWhileLessThanEquals5 = StreamUtils
    .skipWhile(stream(numbers), i -> i <= 5 || )
    .collect(Collectors.toList());
 
assertThat(skippedWhileLessThanEquals5).containsExactly(6, 7, 8, 9, 10);
```

skipWhile()的一个重要特点是，它在找到第一个不满足条件的值后将继续流式传输：

```java
List skippedWhileGreaterThan5 = StreamUtils
    .skipWhile(stream(numbers), i -> i > 5)
    .collect(Collectors.toList());
assertThat(skippedWhileGreaterThan5).containsExactly(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
```

从Java 9开始，标准Stream API中的[dropWhile()](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Stream.html#dropWhile(java.util.function.Predicate))提供与skipWhile()相同的功能。

### 3.8 unfold()

unfold()通过将自定义生成器应用于种子值，然后应用于每个生成的值来生成可能无限的流-可以通过返回Optional.empty()来终止该流：

```java
Stream<Integer> unfolded = StreamUtils
    .unfold(2, i -> (i < 100) 
        ? Optional.of(i * i) : Optional.empty());

assertThat(unfolded.collect(Collectors.toList()))
    .containsExactly(2, 4, 16, 256);
```

### 3.9 windowed()

windowed()**将源流的多个子集创建为List流**。该方法以源流、窗口大小和跳过值作为参数。

列表长度等于窗口大小，而跳过值决定了子集相对于前一个子集的开始位置： 

```java
Integer[] numbers = { 1, 2, 3, 4, 5, 6, 7, 8 };

List<List> windowedWithSkip1 = StreamUtils
    .windowed(stream(numbers), 3, 1)
    .collect(Collectors.toList());
assertThat(windowedWithSkip1)
    .containsExactly(asList(1, 2, 3), asList(2, 3, 4), asList(3, 4, 5), asList(4, 5, 6), asList(5, 6, 7));
```

此外，最后一个窗口保证具有所需的大小，如下例所示：

```java
List<List> windowedWithSkip2 = StreamUtils.windowed(stream(numbers), 3, 2).collect(Collectors.toList());
assertThat(windowedWithSkip2).containsExactly(asList(1, 2, 3), asList(3, 4, 5), asList(5, 6, 7));
```

### 3.10 aggregate()

有两种aggregate()方法，它们的工作方式截然不同。

**第一个aggregate()根据给定的谓词将相等值的元素分组在一起**：

```java
Integer[] numbers = { 1, 2, 2, 3, 4, 4, 4, 5 };
List<List> aggregated = StreamUtils
    .aggregate(Arrays.stream(numbers), (int1, int2) -> int1.compareTo(int2) == 0)
    .collect(Collectors.toList());
assertThat(aggregated).containsExactly(asList(1), asList(2, 2), asList(3), asList(4, 4, 4), asList(5));
```

谓词以连续的方式接收值。因此，如果数字无序，上述操作将给出不同的结果。

**另一方面，第二个aggregate()只是用于将源流中的元素分组为所需大小的组**：

```java
List<List> aggregatedFixSize = StreamUtils
    .aggregate(stream(numbers), 5)
    .collect(Collectors.toList());
assertThat(aggregatedFixSize).containsExactly(asList(1, 2, 2, 3, 4), asList(4, 4, 5));
```

### 3.11 aggregateOnListCondition()

**aggregateOnListCondition()根据谓词和当前活动组对值进行分组**，谓词将当前活动组作为列表和下一个值给出，然后它必须确定该组是否应继续或开始一个新组。

下面的示例解决了将连续的整数值分组到一个组中的要求，其中每组中值的总和不得大于5：

```java
Integer[] numbers = { 1, 1, 2, 3, 4, 4, 5 };
Stream<List<Integer>> aggregated = StreamUtils
    .aggregateOnListCondition(stream(numbers), 
        (currentList, nextInt) -> currentList.stream().mapToInt(Integer::intValue).sum() + nextInt <= 5);
assertThat(aggregated)
    .containsExactly(asList(1, 1, 2), asList(3), asList(4), asList(4), asList(5));
```

## 4. Streamable<T\>

Stream的实例不可重用，因此，**Streamable通过包装和公开与Stream相同的方法来提供可重用的流**：

```java
Streamable<String> s = Streamable.of("a", "b", "c", "d");
List<String> collected1 = s.collect(Collectors.toList());
List<String> collected2 = s.collect(Collectors.toList());
assertThat(collected1).hasSize(4);
assertThat(collected2).hasSize(4);
```

## 5. CollectorUtils

CollectorUtils通过添加几个有用的收集器方法来补充标准Collectors。

### 5.1 maxBy()和minBy()

maxBy()**使用提供的投影逻辑在流中查找最大值**：

```java
Stream<String> clubs = Stream.of("Juventus", "Barcelona", "PSG");
Optional<String> longestName = clubs.collect(CollectorUtils.maxBy(String::length));
assertThat(longestName).contains("Barcelona");
```

相反，minBy()**使用提供的投影逻辑查找最小值**。

### 5.2 unique()

unique()收集器做的事情非常简单：**如果给定的流恰好有1个元素，它就返回唯一的值**：

```java
Stream<Integer> singleElement = Stream.of(1);
Optional<Integer> unique = singleElement.collect(CollectorUtils.unique());
assertThat(unique).contains(1);
```

否则，unique()将引发异常：

```java
Stream multipleElement = Stream.of(1, 2, 3);
assertThatExceptionOfType(NonUniqueValueException.class).isThrownBy(() -> {
    multipleElement.collect(CollectorUtils.unique());
});
```

## 6. 总结

在本文中，我们了解了Protonpack库如何扩展Java Stream API以使其更易于使用，它添加了我们可能常用但标准API中缺少的有用方法。

从Java 9开始，Protonpack提供的一些功能也可在标准Stream API中可用。