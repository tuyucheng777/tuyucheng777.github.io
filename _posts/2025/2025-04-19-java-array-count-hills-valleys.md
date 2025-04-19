---
layout: post
title:  计算整数数组中的山丘和山谷数量
category: algorithms
copyright: algorithms
excerpt: 数组
---

## 1. 简介

在解决数据结构和算法(DSA)问题时，统计序列中不同的模式可能非常有趣。在上一篇文章中，我们讨论了[在整数列表中发现峰值元素的各种方法](https://www.baeldung.com/java-list-find-peak)，这些方法的时间复杂度各不相同，例如O(log n)或O(n)。

在本文中，我们将解决一个常见的问题，即识别并计算整数数组中的“山丘”和“山谷”，这个问题是磨练Java数组遍历和条件检查技能的绝佳途径。我们将逐步探讨这个问题，并提供解决方案、代码示例和解释。

## 2. 问题陈述

给定一个整数数组，我们需要识别并计算所有的山丘和山谷。

定义：

- **山丘**：相邻元素的序列，该序列以较低的数字开始，以较高的数字达到峰值，然后以较低的数字结束。
- **山谷**：相反的模式，序列从高处开始，下降到较低的值，然后回升。

简单来说，**数组中的山峰是一个两侧有较低元素的山峰，而山谷是一个两侧有较高元素的谷底**。

此外，我们应该注意，具有相同值的相邻指数属于同一座山丘或山谷。

最后，要将一个索引归类为山丘或山谷的一部分，它必须在左右两侧都有不相等的邻居，这也意味着我们将一次关注三个索引来构成山丘或山谷。

### 2.1 示例

为了更好地理解山丘和山谷的概念，让我们看下面的例子。

```text
Input array: [4, 5, 6, 5, 4, 5, 4]
```

让我们一步一步地通过数组来根据我们的定义识别山丘和山谷。

**索引0和6**：由于第一个和最后一个索引分别没有左右相邻，它们无法构成任何山丘或山谷。因此，我们将忽略它们。

**索引1(5)**：5的最近不相等邻居是4(左)和6(右)，5不大于两个邻居(因此它不是山丘)，也不小于两个邻居(因此它不是山谷)。

**索引2(6)**：它的邻居是5(左)和5(右)，6大于两个邻居，因此形成一座山[5,6,5\]。

**索引3(5)**：其邻居是6(左)和4(右)，由于5小于6且大于4，因此它既不是山丘也不是山谷。

**索引4(4)**：它的邻居是5(左)和5(右)，由于4小于两个邻居，因此显然形成了一个谷值[5,4,5\]。

**索引5(5)**：邻居是4(左)和4(右)，5大于两个邻居，因此形成了另一个山丘[4,5,4\]。

因此，数组[4,5,6,5,4,5,4\]的输出应该是3。

### 2.2 图形表示

我们将数组[4,5,6,5,4,5,4\]绘制成一个山丘和山谷的图表，其中x轴表示索引，y轴表示每个索引处的值。我们用虚线连接这些点，以直观地表示山丘和山谷：

![](/assets/images/2025/algorithms/javaarraycounthillsvalleys01.png)

绿色标记表示“山丘”，其中一个值高于其两个相邻点，红色标记表示“山谷”，其中一个值低于其两个相邻点。

**图表清楚地显示我们有两座山丘和一座山谷**。

## 3. 算法

让我们定义一个算法来识别和计算给定数组中的所有山丘和山谷来解决这个问题：

```text
1. Accept an array of integers, let's call it numbers.
2. Loop through numbers from the second element to the second-to-last element.
3. For each element at index i:
       Let prev be numbers[i-1], current be numbers[i], and next be numbers[i+1].
4.     When consecutive equal elements are encountered, skip over them while maintaining the last seen value.
5.     Identify Hills:
           If current is greater than both prev and next, it's a hill.
           Increment hill count.
6.     Identify Valleys:
           If current is less than both prev and next, it's a valley.
           Increment valley count.
7. Repeat Steps 3–6 until the end of the array.
8. Return the sum of hills and valley.
```

因此，我们可以使用该算法有效地检测出所有山丘和山谷。**需要注意的是，我们只需要遍历数字数组一次**。

## 4. 实现

现在，让我们用Java实现这个算法：

```java
int countHillsAndValleys(int[] numbers) {
    int hills = 0;
    int valleys = 0;
    for (int i = 1; i < numbers.length - 1; i++) {
        int prev = numbers[i - 1];
        int current = numbers[i];
        int next = numbers[i + 1];

        while (i < numbers.length - 1 && numbers[i] == numbers[i + 1]) {
            //  skip consecutive duplicate elements
            i++;
        }

        if (i != numbers.length - 1) {
            // update the next value to the first distinct neighbor only if it exists
            next = numbers[i + 1];
        }

        if (current > prev && current > next) {
            hills++;
        } else if (current < prev && current < next) {
            valleys++;
        }
    }

    return hills + valleys;
}
```

这里，在countHillsAndValleys()方法中，我们有效地识别并计算了给定整数数组中的山丘和山谷的数量。首先，我们从第二个元素开始迭代数组索引，直到倒数第二个元素结束，这确保了我们检查的每个元素都有左右邻居。

接下来，对于每个索引，我们跳过连续的重复元素以避免冗余检查，并确保逻辑准确地应用于被邻居包围的分组元素。

对于每个剩余的索引，**我们将当前元素与其邻居进行比较-如果它大于两者，我们就将其识别为山丘；如果它小于两者，我们就将其识别为山谷**。

最后，我们返回山丘和山谷的总数。**总的来说，这种方法可以避免额外的列表和内存占用，从而优化性能，使我们的解决方案高效且直观**。

## 5. 测试

让我们通过将不同的数组传递给countHillsAndValleys()来测试我们的逻辑并处理各种测试用例：

```java
// Test case 1: Our example array
int[] array1 = { 4, 5, 6, 5, 4, 5, 4};
assertEquals(3, countHillsAndValleys(array1));

// Test case 2: Array with strictly increasing elements
int[] array2 = { 1, 2, 3, 4, 5, 6 };
assertEquals(0, countHillsAndValleys(array2));

// Test case 3: Constant array
int[] array3 = { 5, 5, 5, 5, 5 };
assertEquals(0, countHillsAndValleys(array3));

// Test case 4: Array with no hills or valleys
int[] array4 = { 6, 6, 5, 5, 4, 1 };
assertEquals(0, countHillsAndValleys(array4));

// Test case 5: Array with a flatten valley
int[] array5 = { 6, 5, 4, 4, 4, 5, 6 };
assertEquals(1, countHillsAndValleys(array5));

// Test case 6: Array with a flatten hill
int[] array6 = { 1, 2, 4, 4, 4, 2, 1 };
assertEquals(1, countHillsAndValleys(array6));
```

这里，**array2是一个严格递增的数组，因此它没有任何“山丘”或“山谷”。同样，array3是一个具有相同元素的常量数组，因此它没有任何“山丘”或“山谷”**。

## 6. 复杂度分析

让我们分析一下解决方案的时间和空间复杂度：

**时间复杂度**：O(n)，其中n是数组的长度，函数countHillsAndValleys()遍历数组一次，每个元素仅检查一次。
**空间复杂度**：O(1)，因为我们只为计数器使用固定大小的空间，并且不需要任何额外的数据结构。

## 7. 总结

在本教程中，我们探索了一种使用Java统计数组中山峰和谷值的结构化方法。此外，我们还处理了一些边缘情况，并通过各种测试用例验证了该解决方案。