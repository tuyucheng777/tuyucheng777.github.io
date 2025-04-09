---
layout: post
title:  Java中的最大子数组问题
category: algorithms
copyright: algorithms
excerpt: 最大子数组
---

## 1. 概述

最大子数组问题是在任何给定数组中找到具有最大和的一系列连续元素的任务。

例如，在下面的数组中，突出显示的子数组具有最大总和(6)：

![](/assets/images/2025/algorithms/javamaximumsubarray01.png)

在本教程中，我们将介绍两种在数组中查找最大子数组的解决方案。我们将设计其中一种，其[时间和空间复杂度](https://www.baeldung.com/java-algorithm-complexity)为O(n)。

## 2. 暴力算法

暴力算法是一种解决问题的迭代方法，在大多数情况下，解决方案需要对数据结构进行多次迭代。在接下来的几节中，我们将应用这种方法来解决最大子数组问题。

### 2.1 方法

一般来说，首先想到的解决方案是计算每个可能子数组的总和，并返回总和最大的子数组。

首先，我们计算从索引0开始的每个子数组的总和。类似地，我们将**找到从0到n-1的每个索引开始的所有子数组**，其中n是数组的长度：

![](/assets/images/2025/algorithms/javamaximumsubarray02.png)

因此，我们将从索引0开始，并将每个元素添加到迭代中的运行总和中。我们还将跟踪迄今为止看到的最大总和，此迭代显示在上图的左侧。

在图像的右侧，我们可以看到从索引3开始的迭代。在该图像的最后一部分，我们得到了索引3和6之间和值最大的子数组。

但是，**我们的算法将继续查找从0到n-1之间的索引开始的所有子数组**。

### 2.2 实现

现在让我们看看如何用Java实现这个解决方案：

```java
public int maxSubArray(int[] nums) {
 
    int n = nums.length;
    int maximumSubArraySum = Integer.MIN_VALUE;
    int start = 0;
    int end = 0;
 
    for (int left = 0; left < n; left++) {
 
        int runningWindowSum = 0;
 
        for (int right = left; right < n; right++) {
            runningWindowSum += nums[right];
 
            if (runningWindowSum > maximumSubArraySum) {
                maximumSubArraySum = runningWindowSum;
                start = left;
                end = right;
            }
        }
    }
    logger.info("Found Maximum Subarray between {} and {}", start, end);
    return maximumSubArraySum;
}
```

正如预期的那样，如果当前总和大于之前的最大总和，我们将更新maximumSubArraySum。值得注意的是，**我们随后还会更新start和end以找出此子数组的索引位置**。

### 2.3 复杂性

一般来说，暴力算法会多次迭代数组以获得所有可能的解决方案，这意味着此解决方案所花费的时间会随着数组中元素的数量二次方增长。对于较小的数组来说，这可能不是问题，**但随着数组大小的增加，此解决方案效率不高**。

通过检查代码，我们还可以看到有两个嵌套的for循环。因此，我们可以得出结论，**该算法的时间复杂度为O(n<sup>2<sup>**)。

在后面的章节中，我们将使用动态规划以O(n)复杂度解决这个问题。

## 3. 动态规划

动态规划通过将问题划分为较小的子问题来解决问题，这与分治算法求解技术非常相似。但是，主要区别在于动态规划只解决一次子问题。

然后，它会存储这个子问题的结果，并在稍后重新使用此结果来解决其他相关子问题，**这个过程称为记忆化**。

### 3.1 Kadane算法

Kadane算法是最大子数组问题的一种流行解决方案，该解决方案基于动态规划。

解决动态规划问题最重要的挑战是**找到最优子问题**。

### 3.2 方法

让我们换个方式来理解这个问题：

![](/assets/images/2025/algorithms/javamaximumsubarray03.png)

在上图中，我们假设最大子数组在最后一个索引位置结束。因此，子数组的最大和将是：

```java
maximumSubArraySum = max_so_far + arr[n-1]
```

**max_so_far是结束于索引n-2的子数组的最大和**，这也显示在上图中。

现在，我们可以将此假设应用于数组中的任何索引。例如，以n-2结尾的最大子数组和可以计算为：

```java
maximumSubArraySum[n-2] = max_so_far[n-3] + arr[n-2]
```

因此，我们可以得出结论：

```java
maximumSubArraySum[i] = maximumSubArraySum[i-1] + arr[i]
```

现在，由于数组中的每个元素都是一个大小为1的特殊子数组，我们还需要检查元素是否大于最大和本身：

```java
maximumSubArraySum[i] = Max(arr[i], maximumSubArraySum[i-1] + arr[i])
```

通过查看这些方程，我们可以看到我们需要在数组的每个索引处找到最大子数组和。因此，我们将问题分成n个子问题，只需迭代数组一次即可在每个索引处找到最大和：

![](/assets/images/2025/algorithms/javamaximumsubarray04.png)

突出显示的元素显示迭代中的当前元素，在每个索引处，我们将应用先前得出的公式来计算max_ending_here的值。这有助于我们**确定是否应将当前元素包含在子数组中，或者从此索引开始新的子数组**。

另一个变量max_so_far用于存储迭代过程中找到的最大子数组和，一旦我们迭代到最后一个索引，max_so_far将存储最大子数组的和。

### 3.3 实现

让我们看看如何按照上述方法在Java中实现Kadane算法：

```java
public int maxSubArraySum(int[] arr) {	
    int size = arr.length;	
    int start = 0;	
    int end = 0;	
    int maxSoFar = arr[0], maxEndingHere = arr[0];	
    for (int i = 1; i < size; i++) {	
        maxEndingHere = maxEndingHere + arr[i];	
        if (arr[i] > maxEndingHere) {	
            maxEndingHere = arr[i];	
            if (maxSoFar < maxEndingHere) {	
                start = i;	
            }	
        }	
        if (maxSoFar < maxEndingHere) {	
            maxSoFar = maxEndingHere;	
            end = i;	
        }	
    }	
    logger.info("Found Maximum Subarray between {} and {}", Math.min(start, end), end);	
    return maxSoFar;	
}
```

在这里，我们更新了start和end以找到最大子数组索引。

请注意，我们采用Math.min(start, end)而不是start作为最大子数组的起始索引。这是因为，如果数组仅包含负数，则最大子数组将是最大元素本身。在这种情况下，if(arr[i] > maxEndingHere)将始终为true；也就是说，start的值大于end的值。

### 3.4 复杂性

**由于我们只需要迭代数组一次，因此该算法的时间复杂度为O(n)**。

由此可见，该解决方案所花费的时间随数组中元素的数量线性增长。因此，它比我们在上一节讨论的暴力方法更有效。

## 4. 总结

在此快速教程中，我们描述了两种解决最大子阵问题的方法。

首先，我们探索了一种暴力方法，发现这种迭代解决方案需要二次方时间。然后，我们讨论了Kadane算法，并使用动态规划在线性时间内解决了这个问题。