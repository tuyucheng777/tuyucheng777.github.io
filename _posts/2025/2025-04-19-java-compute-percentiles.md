---
layout: post
title:  用Java计算百分位数
category: algorithms
copyright: algorithms
excerpt: 百分位数
---

## 1. 概述

在使用Java分析数据时，计算百分位数是理解数字数据集的统计分布和特征的一项基本任务。

在本教程中，我们将介绍在Java中计算百分位数的过程，并提供代码示例和解释。

## 2. 理解百分位数

在讨论实现细节之前，让我们首先了解百分位数是什么以及它们在数据分析中的常见用途。

**百分位数是统计学中使用的一种度量，表示一定百分比的观测值等于或低于该值**。例如，第50个百分位数(也称为中位数)表示50%的数据点低于该值。

值得注意的是，**百分位数的表示单位与输入数据集的单位相同，而不是百分比**。例如，如果数据集指的是月薪，则相应的百分位数将以美元、欧元或其他货币表示。

接下来我们来看几个具体的例子：

```text
Input: A dataset with numbers 1-100 unsorted
-> sorted dataset: [1, 2, ... 49, (50), 51, 52, ..100]
-> The 50th percentile: 50

Input: [-1, 200, 30, 42, -5, 7, 8, 92]
-> sorted dataset: [-2, -1, 7, (8), 30, 42, 92, 200]
-> The 50th percentile: 8
```

百分位数通常用于理解数据分布、识别异常值以及比较不同的数据集。在处理大型数据集或简洁地概括数据集的特征时，百分位数尤其有用。

接下来，让我们看看如何在Java中计算百分位数。

## 3. 从集合中计算百分位数

现在我们了解了百分位数是什么，让我们总结一下如何逐步实现百分位数计算：

- 按升序对给定的数据集进行排序
- 计算所需百分位数的排名为(百分位数 / 100) * dataset.size
- **取排名的上限值，因为排名可以是小数**
- 最终结果是排序后数据集中索引ceiling(rank) - 1处的元素

接下来，让我们创建一个[泛型方法](https://www.baeldung.com/java-generics)来实现上述逻辑：

```java
static <T extends Comparable<T>> T getPercentile(Collection<T> input, double percentile) {
    if (input == null || input.isEmpty()) {
        throw new IllegalArgumentException("The input dataset cannot be null or empty.");
    }
    if (percentile < 0 || percentile > 100) {
        throw new IllegalArgumentException("Percentile must be between 0 and 100 inclusive.");
    }
    List<T> sortedList = input.stream()
            .sorted()
            .collect(Collectors.toList());

    int rank = percentile == 0 ? 1 : (int) Math.ceil(percentile / 100.0 * input.size());
    return sortedList.get(rank - 1);
}
```

可以看出，上面的实现非常简单，不过有几点值得一提：

- 需要验证百分位数参数(0 <= 百分位数 <= 100)
- 我们**使用[Stream API](https://www.baeldung.com/java-8-streams)对输入数据集进行排序，并将排序结果[收集](https://www.baeldung.com/java-stream-to-list-collecting)到新列表中，以避免修改原始数据集**

接下来，让我们测试getPercentile()方法。

## 4. 测试getPercentile()方法

首先，如果百分位数超出有效范围，该方法应该抛出IllegalArgumentException：

```java
assertThrows(IllegalArgumentException.class, () -> getPercentile(List.of(1, 2, 3), -1));
assertThrows(IllegalArgumentException.class, () -> getPercentile(List.of(1, 2, 3), 101));
```

我们**使用[assertThrows()](https://www.baeldung.com/junit-assert-exception)方法来验证是否引发了预期的异常**。

接下来我们以1-100的List作为输入，验证该方法是否能产生预期的结果：

```java
List<Integer> list100 = IntStream.rangeClosed(1, 100)
    .boxed()
    .collect(Collectors.toList());
Collections.shuffle(list100);
 
assertEquals(1, getPercentile(list100, 0));
assertEquals(10, getPercentile(list100, 10));
assertEquals(25, getPercentile(list100, 25));
assertEquals(50, getPercentile(list100, 50));
assertEquals(76, getPercentile(list100, 75.3));
assertEquals(100, getPercentile(list100, 100));
```

在上面的代码中，我们通过[IntStream](https://www.baeldung.com/java-intstream-convert#intstream-to-list)准备了输入列表。此外，我们使用[shuffle()](https://www.baeldung.com/java-shuffle-collection)方法对100个数字进行随机排序。

此外，让我们用另一个数据集输入来测试我们的方法：

```java
List<Integer> list8 = IntStream.of(-1, 200, 30, 42, -5, 7, 8, 92)
    .boxed()
    .collect(Collectors.toList());

assertEquals(-5, getPercentile(list8, 0));
assertEquals(-5, getPercentile(list8, 10));
assertEquals(-1, getPercentile(list8, 25));
assertEquals(8, getPercentile(list8, 50));
assertEquals(92, getPercentile(list8, 75.3));
assertEquals(200, getPercentile(list8, 100));
```

## 5. 从数组计算百分位数

有时，给定的数据集输入是一个[数组](https://www.baeldung.com/java-arrays-guide)而不是一个Collection。在这种情况下，我们可以先[将输入数组转换为List](https://www.baeldung.com/convert-array-to-list-and-list-to-array)，然后利用getPercentile()方法计算所需的百分位数。

接下来，让我们通过将long数组作为输入来演示如何实现这一点：

```java
long[] theArray = new long[] { -1, 200, 30, 42, -5, 7, 8, 92 };
 
//convert the long[] array to a List<Long>
List<Long> list8 = Arrays.stream(theArray)
    .boxed()
    .toList();
 
assertEquals(-5, getPercentile(list8, 0));
assertEquals(-5, getPercentile(list8, 10));
assertEquals(-1, getPercentile(list8, 25));
assertEquals(8, getPercentile(list8, 50));
assertEquals(92, getPercentile(list8, 75.3));
assertEquals(200, getPercentile(list8, 100));
```

如代码所示，由于我们的输入是一个[原始类型数组](https://www.baeldung.com/java-primitive-array-to-list)(long[\])，我们使用[Arrays.stream()](https://www.baeldung.com/java-primitive-array-to-list#streams)将其转换为List。然后，我们可以将转换后的List传递给getPercentile()以获得预期结果。

## 6. 总结

在本文中，我们首先讨论了百分位数的基本原理。然后，我们探讨了如何在Java中计算数据集的百分位数。