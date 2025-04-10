---
layout: post
title:  Java中的选择排序
category: algorithms
copyright: algorithms
excerpt: 排序
---

## 1. 简介

在本教程中，我们将学习选择排序，了解其在Java中的实现，并分析其性能。

## 2. 算法概述

选择排序从无序数组中第一个位置的元素开始，然后扫描后续元素以找到最小元素。找到后，将最小元素与第一个位置的元素交换。

然后，算法转到第二个位置的元素，并扫描后续元素以找到第二小元素的索引。找到后，将第二小元素与第二个位置的元素交换。

这个过程一直持续到我们到达数组的第n - 1个元素，并将第n - 1个最小元素放在第n - 1个位置。最后一个元素会自动落到第n - 1次迭代的位置，从而对数组进行排序。

**我们找到最大元素而不是最小元素来按降序对数组进行排序**。

我们来看一个未排序数组的例子，并按升序对其进行排序，以直观地理解该算法。

### 2.1 示例

考虑以下未排序的数组：

int[] arr= {5 ,4 ,1 ,6 ,2 }

迭代1

考虑到算法的上述工作原理，我们从位于第一位的元素5开始，扫描所有后续元素以找到最小元素1。然后，我们将最小元素与位于第一位的元素交换。

修改后的数组现在如下所示：

{1 ,4 ,5 ,6 ,2 }

总比较次数：4

迭代2

在第二次迭代中，我们转到第2个元素4，并扫描后续元素以找到第二小元素2，然后我们将第二小元素与第二个位置的元素交换。

修改后的数组现在如下所示：

{1 ,2 ,5 ,6 ,4 }

总比较次数：3

继续类似操作，我们有以下迭代：

迭代3

{1 ,2 ,4 ,6 ,5 }

总比较次数：2

迭代4

{1 ,2 ,4 ,5 ,6 }

总比较次数：1

## 3. 实现

让我们使用几个for循环来实现选择排序：
```java
public static void sortAscending(final int[] arr) {
    for (int i = 0; i < arr.length - 1; i++) {
        int minElementIndex = i;
        for (int j = i + 1; j < arr.length; j++) {
            if (arr[minElementIndex] > arr[j]) {
                minElementIndex = j;
            }
        }

        if (minElementIndex != i) {
            int temp = arr[i];
            arr[i] = arr[minElementIndex];
            arr[minElementIndex] = temp;
        }
    }
}
```

当然，为了扭转它，我们可以做一些非常类似的事情：
```java
public static void sortDescending(final int[] arr) {
    for (int i = 0; i < arr.length - 1; i++) {
        int maxElementIndex = i;
        for (int j = i + 1; j < arr.length; j++) {
            if (arr[maxElementIndex] < arr[j]) {
                maxElementIndex = j;
            }
        }

        if (maxElementIndex != i) {
            int temp = arr[i];
            arr[i] = arr[maxElementIndex];
            arr[maxElementIndex] = temp;
        }
    }
}
```

再多花点功夫，我们就可以使用[Comparator](https://www.baeldung.com/java-comparator-comparable)将它们结合起来。

## 4. 性能概览

### 4.1 时间

在我们之前看到的例子中，**选择最小元素需要总共(n - 1)次比较**，然后将其交换到第一个位置。同样，选择下一个最小元素需要总共(n - 2)次比较，然后将其交换到第二个位置，依此类推。

因此，从索引0开始，我们执行n - 1、n - 2、n - 3、n - 4....1 次比较，由于之前的迭代和交换，最后一个元素会自动落到位。

从数学上讲，前n - 1个自然数的总和将告诉我们需要多少次比较才能使用选择排序对大小为n的数组进行排序。 

**n个自然数的和的公式是n(n + 1) / 2**。

在我们的例子中，我们需要前n - 1个自然数的总和；因此，我们将上式中的n替换为n - 1得到：

(n - 1)(n - 1 + 1) / 2 = (n - 1)n / 2 = (n^2 - n) / 2

由于随着n的增长，n^2的增长非常显著，我们以n的高次幂作为性能基准，使得该算法的时间复杂度为O(n^2)。

### 4.2 空间

在辅助空间复杂度方面，选择排序需要一个额外的变量来临时保存交换的值，因此，选择排序的空间复杂度为O(1)。

## 5. 总结

选择排序是一种非常容易理解和实现的排序算法，不幸的是，**它的时间复杂度是平方级的，这使得它成为一种成本高昂的排序技术**。此外，由于算法必须扫描每个元素，因此**最佳情况、平均情况和最坏情况的时间复杂度都相同**。

其他排序技术，如[插入排序](https://www.baeldung.com/java-insertion-sort)和[希尔排序](https://www.baeldung.com/java-shell-sort)也具有平方最坏情况时间复杂度，但它们在最佳和平均情况下的表现更好。