---
layout: post
title:  Java中的冒泡排序
category: algorithms
copyright: algorithms
excerpt: 排序
---

## 1. 简介

在这篇简短的文章中，我们将详细探讨冒泡排序算法，重点介绍Java实现。

这是最直接的排序算法之一；其核心思想是，如果数组中相邻元素的顺序不正确，则不断交换它们，直到集合排序完成。

当我们迭代数据结构时，较小的元素会“冒泡”到列表顶部。因此，该技术被称为冒泡排序。

由于排序是通过交换来执行的，因此我们可以说它执行就地排序。

此外，**如果两个元素具有相同的值，则结果数据的顺序将保留**-这使得它成为一种[稳定的排序](https://www.baeldung.com/cs/stable-sorting-algorithms)。

## 2. 方法论

如前所述，要对数组进行排序，我们需要迭代数组并比较相邻元素，必要时交换它们。对于大小为n的数组，我们执行n - 1次这样的迭代。

让我们举一个例子来理解该方法，我们想按升序对数组进行排序：

> 4 2 1 6 3 5

我们通过比较4和2来开始第一次迭代；它们的顺序显然不正确，交换结果如下：

> [2 4\]1 6 3 5

现在，对4和1重复相同的操作：

> 2 [1 4\]6 3 5

一直到最后：

>2 1 [4 6\]3 5
>
>2 1 4 [3 6\]5
>
>2 1 4 3 [5 6\]

如我们所见，在第一次迭代结束时，我们将最后一个元素放在了正确的位置。现在，我们需要做的就是在后续迭代中重复相同的过程，不过，我们排除了已经排序的元素。

在第二次迭代中，我们将遍历除最后一个元素之外的整个数组。同样，对于第三次迭代，我们省略最后两个元素。通常，对于第k次迭代，我们迭代到索引nk(不包括)。在n - 1次迭代结束时，我们将获得排序后的数组。

现在你已经了解了该技术，让我们深入研究其实现方法。

## 3. 实现

让我们使用Java 8方法实现我们讨论过的示例数组的排序：
```java
void bubbleSort(Integer[] arr) {
    int n = arr.length;
    IntStream.range(0, n - 1)
            .flatMap(i -> IntStream.range(1, n - i))
            .forEach(j -> {
                if (arr[j - 1] > arr[j]) {
                    int temp = arr[j];
                    arr[j] = arr[j - 1];
                    arr[j - 1] = temp;
                }
            });
}
```

对该算法进行快速的JUnit测试：
```java
@Test
void whenSortedWithBubbleSort_thenGetSortedArray() {
    Integer[] array = { 2, 1, 4, 6, 3, 5 };
    Integer[] sortedArray = { 1, 2, 3, 4, 5, 6 };
    BubbleSort bubbleSort = new BubbleSort();
    bubbleSort.bubbleSort(array);

    assertArrayEquals(array, sortedArray);
}
```

## 4. 复杂性与优化

我们可以看出，**对于平均情况和最坏情况，时间复杂度都是O(n^2)**。

此外，**即使在最坏的情况下，空间复杂度也是O(1)，因为冒泡排序算法不需要任何额外的内存**，并且排序在原始数组中进行。

通过仔细分析解决方案，我们可以看到，**如果在一次迭代中没有发现交换，我们就不需要进一步迭代**。

对于前面讨论的例子，经过第二次迭代后，我们得到：

> 1 2 3 4 5 6

在第三次迭代中，我们不需要交换任何一对相邻元素。因此，我们可以跳过所有剩余的迭代。

如果数组已排序，则第一次迭代本身就不需要交换-这意味着我们可以停止执行。这是最佳情况，**算法的时间复杂度为O(n)**。

现在，我们来实现优化的解决方案。
```java
public void optimizedBubbleSort(Integer[] arr) {
    int i = 0, n = arr.length;
    boolean swapNeeded = true;
    while (i < n - 1 && swapNeeded) {
        swapNeeded = false;
        for (int j = 1; j < n - i; j++) {
            if (arr[j - 1] > arr[j]) {
                int temp = arr[j - 1];
                arr[j - 1] = arr[j];
                arr[j] = temp;
                swapNeeded = true;
            }
        }
        if(!swapNeeded) {
            break;
        }
        i++;
    }
}
```

让我们检查优化算法的输出：
```java
@Test
void givenIntegerArray_whenSortedWithOptimizedBubbleSort_thenGetSortedArray() {
    Integer[] array = { 2, 1, 4, 6, 3, 5 };
    Integer[] sortedArray = { 1, 2, 3, 4, 5, 6 };
    BubbleSort bubbleSort = new BubbleSort();
    bubbleSort.optimizedBubbleSort(array);

    assertArrayEquals(array, sortedArray);
}
```

## 5. 总结

在本教程中，我们了解了冒泡排序的工作原理及其在Java中的实现，我们还了解了如何对其进行优化。总而言之，它是一种就地稳定算法，时间复杂度为：

- 最坏情况和平均情况：O(n*n)，当数组按相反顺序排列时
- 最佳情况：O(n)，当数组已经排序时

该算法在计算机图形学中很流行，因为它能够检测排序中的一些小错误。例如，在一个几乎排序的数组中，只需交换两个元素，即可获得完全排序的数组。冒泡排序可以在线性时间内修复此类错误(即对该数组进行排序)。