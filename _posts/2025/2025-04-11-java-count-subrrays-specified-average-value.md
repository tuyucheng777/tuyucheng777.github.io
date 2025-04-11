---
layout: post
title:  Java中计算具有特定算术平均值的子数组数量
category: algorithms
copyright: algorithms
excerpt: 算术平均值
---

## 1. 概述

在这个简短的教程中，我们将学习如何有效地找到具有给定算术平均值的数组的连续子数组的数量。

我们将从一个简单的方法开始，该方法以二次[时间复杂度](https://www.baeldung.com/cs/big-oh-asymptotic-complexity)O(n^2)计算任务。然后，我们将分析其效率低下的原因，并使用[前缀和](https://www.baeldung.com/cs/choosing-subarray-adds-up-to-number#prefixes-approach)与频率图将其优化为线性O(n)解决方案。

## 2. 问题陈述

让我们首先了解我们要解决的问题是什么。

假设我们有一个整数数组和一个目标平均值，该平均值只是一个整数。现在，**我们想要从输入数组开始计算具有特定算术平均值的连续子数组的数量**。例如，对于数组[5,3,6,2\]和目标平均值4，输出应该是3。这是因为子数组[5,3\]、[6,2\]和[5,3,6,2\]的平均值均为4。

此外，我们还想施加以下限制：

- 该数组最多可包含100000个元素
- 数组中每个数字的取值范围是[-1,000,000,000, +1,000,000,000\]
- 目标平均值在同一范围内

让我们首先用蛮力方法解决这个问题。

## 3. 暴力破解

解决这个问题最直接的方法是从两个嵌套循环开始，一个用于迭代子数组的所有可能索引，另一个用于计算子数组的总和并检查平均值是否等于目标平均值：
```java
static int countSubarraysWithMean(int[] inputArray, int mean) {
    int count = 0;
    for (int i = 0; i < inputArray.length; i++) {
        long sum = 0;
        for (int j = i; j < inputArray.length; j++) {
            sum += inputArray[j];
            if (sum * 1.0 / (j - i + 1) == mean) {
                count++;
            }
        }
    }
    return count;
}
```

在这里，我们计算每个子数组的总和和长度，以得出平均值。如果平均值等于目标平均值，则增加计数。此外，我们乘以一个浮点数，以便更精确地计算平均值。

无论这个解决方案看起来多么简单，它的性能都不佳。**当谈到算法复杂度时，我们通常关注的是时间复杂度**，这个暴力解法的时间复杂度为O(n^2)，效率很低。事实上，对于100000个元素的输入约束，我们需要执行100亿次操作，这太慢了。

让我们寻找一个替代、更有效的解决方案。

## 4. 线性解决方案

在继续之前，我们需要了解两个关键思想，以将解决方案优化为线性时间复杂度。

### 4.1 理解前缀和与频率图

首先，前缀和的概念使我们能够在O(1)时间内计算出任何子数组的和。其次，频率图的概念帮助我们通过跟踪某些值的频率来计数子数组。

现在，假设我们有一个输入数组X和一个目标均值S，让我们定义两个数组来表示输入数组的前缀和与调整后的前缀和：
```text
P[i] = X[0] + X[1] + ... + X[i] (prefix sum array)
ADJUSTED_PREFIX_SUM_ARRAY[i] = P[i] - S * i (adjusted prefix sum array)
```

然后，对于任何平均值S的子数组[i,j\]，我们定义Q：
```text
ADJUSTED_PREFIX_SUM_ARRAY[j] = P[j] - S * j
     = (P[j] - P[i-1]) - S * (j - (i-1))
     = (sum of subarray [i, j]) - (length of subarray [i, j]) * S
     = ADJUSTED_PREFIX_SUM_ARRAY[i-1]
```

**ADJUSTED_PREFIX_SUM_ARRAY计算子数组[i,j\]的总和，并减去平均值为S的子数组的预期和**。现在，有了这些值，我们可以通过计算索引(i-1,j)对的数量来计算平均值为S的子数组的数量，其中ADJUSTED_PREFIX_SUM_ARRAY[j\] = ADJUSTED_PREFIX_SUM_ARRAY[i-1\]。

这里的关键点是，如果我们在ADJUSTED_PREFIX_SUM_ARRAY数组中找到两个具有相同值的索引，则这两个索引之间的子数组(不包括较早的索引)的平均值就是S，这意味着从i到j的子数组的平均值恰好是S：
```text
(P[j] - P[i-1]) - S * (j - (i-1)) = 0
```

这个子数组等价于这个：
```text
P[j] - S * j = P[i-1] - S * (i-1)
```

左边是ADJUSTED_PREFIX_SUM_ARRAY[j\]，右边是ADJUSTED_PREFIX_SUM_ARRAY[i-1\]。

让我们来实现它。

### 4.2 Java实现

让我们首先创建两个数组：
```java
static int countSubarraysWithMean(int[] inputArray, int mean) {
    int n = inputArray.length;
    long[] prefixSums = new long[n+1];
    long[] adjustedPrefixes = new long[n+1];
    // More code
}
```

这里我们使用长整型而不是整数，以避免在计算包含较大值的子数组之和时出现溢出。然后，我们计算前缀和数组P以及调整后的前缀和数组Q：
```java
for (int i = 0; i < n; i++) {
    prefixSums[i+1] = prefixSums[i] + inputArray[i];
    adjustedPrefixes[i+1] = prefixSums[i+1] - (long) mean * (i+1);
}
```

接下来，我们创建一个频率图来计算子数组的数量并返回总数：
```java
Map<Long, Integer> count = new HashMap<>();
int total = 0;
for (long adjustedPrefix : adjustedPrefixes) {
    total += count.getOrDefault(adjustedPrefix, 0);
    count.put(adjustedPrefix, count.getOrDefault(adjustedPrefix, 0) + 1);
}

return total;
```

对于前缀数组中的每个已调整前缀，我们将总数增加迄今为止看到的adjustedPrefixes的频率。如果频率图不包含前缀，则返回0。

此解决方案的时间复杂度为O(n)，我们运行了两次for循环。首先，我们计算前缀和以及调整后的前缀和，第二次，我们计算子数组的数量。由于我们使用了两个大小为n的数组和一个可以存储n个条目的频率图，因此空间复杂度也为O(n)。

## 5. 总结

在本文中，我们解决了给定算术平均值的子数组计数问题。首先，我们实现了一个时间复杂度为O(n^2)的暴力解决方案。然后，我们使用前缀和以及频率图将其优化为线性时间复杂度O(n)。