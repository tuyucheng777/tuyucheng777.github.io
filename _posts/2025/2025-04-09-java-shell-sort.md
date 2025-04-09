---
layout: post
title:  Java中的希尔排序
category: algorithms
copyright: algorithms
excerpt: 排序
---

## 1. 简介

在本教程中，我们将描述Java中的希尔排序算法。

## 2. 希尔排序概述

### 2.1 描述

让我们首先描述一下希尔排序算法，这样就知道我们要实现什么。

希尔排序基于[插入排序算法](https://www.baeldung.com/java-insertion-sort)，属于非常高效的算法。通常，**该算法将原始集合分成较小的子集，然后使用插入排序对每个子集进行排序**。

但是，如何创建子集并不简单。它不像我们预期的那样选择相邻元素来构成子集，相反，希尔排序使用所谓的间隔或间隙来创建子集。例如，如果间隙为I，则意味着一个子集将包含相距I个位置的元素。

首先，该算法对彼此距离较远的元素进行排序。然后，距离缩小，并比较距离较近的元素。这样，一些位置不正确的元素可以比我们用相邻元素创建子集更快地定位。

### 2.2 示例

让我们在示例中看一下间隙为3和1且未排序列表包含9个元素的情况：

![](/assets/images/2025/algorithms/javashellsort01.png)

如果我们遵循上述描述，在第一次迭代中，我们将有3个包含3个元素的子集(以相同的颜色突出显示)：

![](/assets/images/2025/algorithms/javashellsort02.png)

在第一次迭代中对每个子集进行排序后，列表将如下所示：

![](/assets/images/2025/algorithms/javashellsort03.png)

我们可以注意到，虽然我们还没有排序列表，但元素现在已经更接近所需的位置。

最后，我们需要再进行一次增量为1的排序，这实际上是一个基本的插入排序。现在，对列表进行排序所需执行的移位操作次数比不进行第一次迭代的情况要少：

![](/assets/images/2025/algorithms/javashellsort04.png)

### 2.3 选择间隙序列

正如我们提到的，希尔排序有一种独特的选择间隙序列的方法。这是一项艰巨的任务，我们应该小心不要选择太少或太多的间隙。更多详细信息可以在[最推荐的间隙序列列表](https://en.wikipedia.org/wiki/Shellsort#Gap_sequences)中找到。

## 3. 实现

现在让我们看一下实现，我们将使用希尔的原始序列作为间隔增量：
```text
N/2, N/4, ..., 1 (continuously dividing by 2)
```

实现本身并不太复杂：
```java
public void sort(int[] arrayToSort) {
    int n = arrayToSort.length;

    for (int gap = n / 2; gap > 0; gap /= 2) {
        for (int i = gap; i < n; i++) {
            int key = arrayToSort[i];
            int j = i;
            while (j >= gap && arrayToSort[j - gap] > key) {
                arrayToSort[j] = arrayToSort[j - gap];
                j -= gap;
            }
            arrayToSort[j] = key;
        }
    }
}
```

我们首先用for循环创建一个间隙序列，然后对每个间隙大小进行插入排序。

现在，我们可以轻松地测试我们的方法：
```java
@Test
void givenUnsortedArray_whenShellSort_thenSortedAsc() {
    int[] input = {41, 15, 82, 5, 65, 19, 32, 43, 8};
    ShellSort.sort(input);
    int[] expected = {5, 8, 15, 19, 32, 41, 43, 65, 82};
    assertArrayEquals("the two arrays are not equal", expected, input);
}
```

## 4. 复杂度

通常，**希尔排序算法对于中等大小的列表非常有效**，[复杂度](https://www.baeldung.com/cs/shellsort-complexity)很难确定，因为它很大程度上取决于间隙序列，但时间复杂度在O(N)和O(N^2)之间变化。

最坏情况的空间复杂度为O(N)，辅助空间为O(1)。

## 5. 总结

在本教程中，我们描述了希尔排序并说明了如何在Java中实现它。