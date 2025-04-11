---
layout: post
title:  从Java中的整数列表中查找最接近给定值的数字
category: algorithms
copyright: algorithms
excerpt: 算法
---

## 1. 概述

当我们在Java中使用整数List时，我们可能会遇到的一个常见问题是在List中找到最接近给定值的数字。

在本教程中，我们将探索使用Java解决此问题的不同方法。

## 2. 问题介绍

我们先来看一下问题的定义：

**给定一个整数列表和一个目标值，我们需要从列表中找到最接近目标的数字。如果多个数字同样接近目标值，我们选择索引最小的数字**。

接下来我们来快速了解一下数字与给定目标之间的“距离”。

### 2.1 两个数字之间的距离

在这种情况下，**数字a和b之间的距离是a – b的绝对值：距离 = abs(a – b)**

例如，给定target = 42，以下是target与不同数字(n)之间的距离(D)：

- n = 42 → D = abs(42 – 42) = 0
- n = 100 → D = abs(42 - 100) = 58
- n = 44 → D = abs(42 – 44) = 2
- n = 40 →  D = abs(42 - 40) = 2

我们可以看到，给定target = 42，数字44和40与target的距离相同。

### 2.2 理解问题

因此，**列表中最接近目标的数字是所有元素中与目标距离最小的**。例如，在列表中：
```text
[ 8, 100, 6, 89, -23, 77 ]
```

假设target = 70，列表中最接近它的整数是77。但如果target = 7，那么8(索引0)和6(索引2)与其距离相等(1)。**在这种情况下，我们选择索引最小的数字**。因此，8是预期结果。

为简单起见，**我们假设给定的List不为null或为空**，我们将探索解决此问题的各种方法。我们还将以此整数List为例，并利用单元测试断言来验证每个解决方案的结果。

另外，为了有效地解决问题，我们将根据输入列表是否排序选择不同的算法来解决问题。

接下来我们设计一些测试用例来覆盖所有场景：

- 当target(-100)小于列表中的最小整数(-23)时：结果 = -23
- 当target(500)大于列表中的最大整数(100)时：结果 = 100
- 当target(89)存在于列表中时：结果 = 89
- 当target(70)在列表中不存在时：结果 = 77 
- 如果列表中的多个整数(8和6)与target(7)的距离相同(1)：结果 = 8

现在，让我们深入研究代码。

## 3. 当我们不知道列表是否已排序时

通常，我们不知道给定的List是否已排序，因此，我们创建一个List变量来保存数字：
```java
static final List<Integer> UNSORTED_LIST = List.of(8, 100, 6, 89, -23, 77);
```

显然，这不是一个排序的列表。

接下来，我们将使用两种不同的方法解决这个问题。

### 3.1 使用循环

解决这个问题的第一个想法是**遍历列表并检查每个数字和目标之间的距离**：
```java
static int findClosestByLoop(List<Integer> numbers, int target) {
    int closest = numbers.get(0);
    int minDistance = Math.abs(target - closest);
    for (int num : numbers) {
        int distance = Math.abs(target - num);
        if (distance < minDistance) {
            closest = num;
            minDistance = distance;
        }
    }
    return closest;
}
```

如代码所示，我们从列表中的第一个数字开始作为最近的数字，并**在另一个变量minDistance中保存目标与它之间的距离，该变量跟踪最小距离**。

然后，我们循环遍历List。对于List中的每个数字，我们计算其与目标的距离并将其与minDistance进行比较，如果它小于minDistance，我们将更新nearest和minDistance。

遍历整个List之后，closest保存最终结果。

接下来，让我们创建一个测试来检查此方法是否按预期工作：
```java
assertEquals(-23, findClosestByLoop(UNSORTED_LIST, -100));
assertEquals(100, findClosestByLoop(UNSORTED_LIST, 500));
assertEquals(89, findClosestByLoop(UNSORTED_LIST, 89));
assertEquals(77, findClosestByLoop(UNSORTED_LIST, 70));
assertEquals(8, findClosestByLoop(UNSORTED_LIST, 7));
```

我们可以看到，这个基于循环的解决方案通过了我们所有的测试用例。

### 3.2 使用Stream API

[Java Stream API](https://www.baeldung.com/java-8-streams/)可以让我们方便简洁地处理集合，接下来让我们使用Stream API来解决这个问题：
```java
static int findClosestByStream(List<Integer> numbers, int target) {
    return numbers.stream()
        .min(Comparator.comparingInt(o -> Math.abs(o - target)))
        .get();
}
```

首先，numbers.stream()将List<Integer\>转换为Stream<Integer\>。

然后，min()根据自定义[Comparator](https://www.baeldung.com/java-comparator-comparable#comparator)查找Stream中的最小元素，在这里，**我们使用[Comparator.comparingInt()](https://www.baeldung.com/java-8-comparator-comparing#4-using-comparatorcomparingint)和[Lambda表达式](https://www.baeldung.com/java-8-lambda-expressions-tips/)来创建自定义Comparator，该Comparator根据Lambda生成的值比较整数**。Lambda o -> Math.abs(o – target)有效地测量了target和List中每个数字之间的距离。

min()操作返回一个[Optional](https://www.baeldung.com/java-optional/)，由于我们假设List不为空，因此Optional始终[存在](https://www.baeldung.com/java-optional#is-present)。因此，我们直接调用get()来从Optional对象中提取最接近的数字。

接下来，让我们检查基于流的方法是否正常工作：
```java
assertEquals(-23, findClosestByStream(UNSORTED_LIST, -100));
assertEquals(100, findClosestByStream(UNSORTED_LIST, 500));
assertEquals(89, findClosestByStream(UNSORTED_LIST, 89));
assertEquals(77, findClosestByStream(UNSORTED_LIST, 70));
assertEquals(8, findClosestByStream(UNSORTED_LIST, 7));
```

如果我们运行测试，它就会通过。因此，这个解决方案简洁而有效地完成了工作。

### 3.3 性能

无论是直接使用循环还是使用紧凑的流，我们都必须遍历整个List一次才能得到结果。因此，**两种解决方案的[时间复杂度](https://www.baeldung.com/cs/time-vs-space-complexity)相同：O(n)**。

由于这两种解决方案都没有考虑列表顺序，因此无论给定的列表是否排序，它们都可以起作用。 

但是，如果我们知道输入列表是有序的，我们可以应用不同的算法来获得更好的性能。接下来，让我们看看如何做到这一点。

## 4. 当列表排序后

**如果列表已排序，我们可以使用[复杂度](https://www.baeldung.com/cs/binary-search-complexity)为O(log n)的[二分查找](https://www.baeldung.com/java-binary-search/)来提高性能**。 

首先，让我们创建一个包含前面示例中相同数字的排序列表：
```java
static final List<Integer> SORTED_LIST = List.of(-23, 6, 8, 77, 89, 100);
```

然后，我们可以实现基于二分查找的方法来解决这个问题：
```java
public static int findClosestByBiSearch(List<Integer> sortedNumbers, int target) {
    int first = sortedNumbers.get(0);
    if (target <= first) {
        return first;
    }

    int last = sortedNumbers.get(sortedNumbers.size() - 1);
    if (target >= last) {
        return last;
    }

    int pos = Collections.binarySearch(sortedNumbers, target);
    if (pos > 0) {
        return sortedNumbers.get(pos);
    }
    int insertPos = -(pos + 1);
    int pre = sortedNumbers.get(insertPos - 1);
    int after = sortedNumbers.get(insertPos);

    return Math.abs(pre - target) <= Math.abs(after - target) ? pre : after;
}
```

现在，让我们了解代码的工作原理。

首先，我们处理两种特殊情况：

- target <= List中的最小数字，即第一个元素-返回第一个元素
- target >= 列表中的最大数字，即最后一个元素-返回最后一个元素

然后，**我们调用[Collections.binarySearch()](https://www.baeldung.com/java-binary-search#4-using-collectionsbinarysearch)来在排序后的List中找到target的位置(pos)**，binarySearch()返回：

- pos > 0：完全匹配，即在列表中找到target
- pos < 0：未找到target，**负pos是插入点的负数(target将插入到该位置以保持排序顺序)减1：pos = (-insertPos – 1)**

**如果找到完全匹配，我们直接采用匹配，因为它是与target最接近的数字(距离 = 0)**。

否则，我们需要在插入点的前后两个数字中找出最接近的数字(insertionPos = -(pos - 1))，当然，距离较小的数字获胜。

该方法通过了相同的测试用例：
```java
assertEquals(-23, findClosestByBiSearch(SORTED_LIST, -100));
assertEquals(100, findClosestByBiSearch(SORTED_LIST, 500));
assertEquals(89, findClosestByBiSearch(SORTED_LIST, 89));
assertEquals(77, findClosestByBiSearch(SORTED_LIST, 70));
assertEquals(6, findClosestByBiSearch(SORTED_LIST, 7));
```

值得注意的是，给定target = 7，最后一个断言中的预期数字是6而不是8，这是因为在排序后的List中6的索引(1)小于8的索引(2)。

## 5. 总结

在本文中，我们探讨了两种从整数列表中查找最接近给定值的数字的方法。此外，如果列表已排序，我们可以使用二分搜索高效地找到最接近的数字，时间复杂度为O(log n)。