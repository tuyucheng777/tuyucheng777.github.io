---
layout: post
title:  如何在Java中合并两个已排序数组
category: algorithms
copyright: algorithms
excerpt: 数组
---

## 1. 简介

在本教程中，我们将学习如何将两个已排序组合并为一已排序数组。

## 2. 问题

让我们理解一下这个问题，我们有两个排序好的数组，想将它们合并为一个。

![](/assets/images/2025/algorithms/javamergesortedarrays01.png)

## 3. 算法

当我们分析这个问题时，**很容易发现我们可以通过使用[归并排序](https://www.baeldung.com/java-merge-sort)的合并操作来解决这个问题**。

假设我们有两个已排序数组foo和bar，长度分别为fooLength和barLength。接下来，我们可以声明另一个大小为fooLength + barLength的合并数组。

然后，我们应该在同一个循环中遍历这两个数组。我们将为每个数组维护一个当前索引值，即fooPosition和barPosition。在循环的给定迭代中，**我们取索引处值较小的元素，并推进该索引**，该元素将占据合并数组中的下一个位置。

最后，一旦我们从一个数组传输了所有元素，我们就会将另一个数组中的剩余元素复制到合并数组中。

现在让我们通过图片来看一下这个过程，以便更好地理解该算法。

步骤1：

我们首先比较两个数组中的元素，然后选择较小的一个。

![](/assets/images/2025/algorithms/javamergesortedarrays02.png)

然后我们增加第一个数组中的位置。

步骤2：

![](/assets/images/2025/algorithms/javamergesortedarrays03.png)

这里我们增加第二个数组中的位置并移动到下一个元素8。

步骤3：

![](/assets/images/2025/algorithms/javamergesortedarrays04.png)

在这次迭代结束时，我们遍历了第一个数组的所有元素。

步骤4：

在此步骤中，我们只需将第二个数组中所有剩余的元素复制到结果中。

![](/assets/images/2025/algorithms/javamergesortedarrays05.png)

## 4. 实现

现在我们看看如何实现它：
```java
public static int[] merge(int[] foo, int[] bar) {
    int fooLength = foo.length;
    int barLength = bar.length;

    int[] merged = new int[fooLength + barLength];

    int fooPosition, barPosition, mergedPosition;
    fooPosition = barPosition = mergedPosition = 0;

    while(fooPosition < fooLength && barPosition < barLength) {
        if (foo[fooPosition] < bar[barPosition]) {
            merged[mergedPosition++] = foo[fooPosition++];
        } else {
            merged[mergedPosition++] = bar[barPosition++];
        }
    }

    while (fooPosition < fooLength) {
        merged[mergedPosition++] = foo[fooPosition++];
    }

    while (barPosition < barLength) {
        merged[mergedPosition++] = bar[barPosition++];
    }

    return merged;
}
```

让我们进行一个简单的测试：
```java
@Test
void givenTwoSortedArrays_whenMerged_thenReturnMergedSortedArray() {

    int[] foo = { 3, 7 };
    int[] bar = { 4, 8, 11 };
    int[] merged = { 3, 4, 7, 8, 11 };

    assertArrayEquals(merged, SortedArrays.merge(foo, bar));
}
```

## 5. 复杂度

我们遍历两个数组并选择较小的元素，最后，我们从foo或bar数组中复制其余元素，**因此时间复杂度变为O(fooLength + barLength)**。我们使用了一个辅助数组来获取结果，**因此空间复杂度也是O(fooLength + barLength)**。

## 6. 总结

在本教程中，我们学习了如何将两个已排序的数组合并为一个。