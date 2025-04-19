---
layout: post
title:  用Java解决磁带平衡问题
category: algorithms
copyright: algorithms
excerpt: 磁带平衡问题
---

## 1. 概述

在本教程中，我们将介绍磁带平衡问题并找到一个有效的解决方案，这个问题是一个常见的面试题。我们将看到，解决这个问题的一个好方法是[确定数组的平衡索引](https://www.baeldung.com/java-equilibrium-index-array)。

“磁带”一词是对数字序列的物理隐喻，将其切成两半会得到两个部分。找到平衡点，重点在于平衡这两个部分，以最小化它们之间的差异。

## 2. 问题的提出

**给定一个大小为N的整数[数组](https://www.baeldung.com/java-arrays-guide)A，其中N >= 2，目标是找到通过在任何位置P(其中0 < P < N)拆分数组而创建的两个非空部分之和之间的最小[绝对差](https://www.baeldung.com/java-absolute-difference-of-two-integers)**。

例如，让我们考虑数组{-1,6,8,-4,7}：

- |A[0\] – (A[1\] + A[2\] + A[3\] + A[4\])| = |(-1) – (6 + 8 + (-4) + 7)| = 18
- |(A[0\] + A[1\]) - (A[2\] +A[3\] + A[4\])| = |((-1) +6) - (8 + (-4) + 7)| = 6
- |(A[0\] + A[1\] + A[2\]) – (A[3\] + A[4\])| = |((-1) + 6 + 8) – ((-4) + 7)| = 10
- |(A[0\] + A[1\] + A[2\] + A[3\]) - A[4\]| = |((-1) + 6 + 8 + (-4)) - 7| = 2

由于2低于6、10和18，在这种情况下，A的磁带均衡为2。

由于我们的书写方式，较低索引元素的和通常称为左和，而较高索引元素的和称为右和。

## 3. 算法

计算数组磁带平衡的简单解决方案是计算每个索引处左右和的差值，然而，这需要对数组元素进行内部迭代，这会损害算法的性能。

因此，我们更倾向于从计算数组的部分和开始，**索引i处的部分和是A中所有索引小于或等于i的元素之和**。

换句话说，部分和数组包含每个索引处的左侧和。然后，我们可以注意到，索引i处的右侧和是总和与索引i处的部分和之间的差，因为A[i+1\] + A[i+2\] + ... + A[N-1\] = (A[0\] + A[1\] + ... + A[i\] + A[i+1\] + ... + A[N-1\]) – (A[0\] + A[1\] + ... + A[i\])。

简而言之，我们可以从部分和数组中检索两个和。此解决方案只需要对数组进行两次连续迭代：算法复杂度为O(N)。

## 4. 计算部分和

**让我们计算部分和数组：索引i处的值是原始数组中索引低于或等于i的所有元素的总和**：

```java
int[] partialSums = new int[array.length];
partialSums[0] = array[0];
for (int i = 1; i < array.length; i++) {
    partialSums[i] = partialSums[i - 1] + array[i];
}
```

## 5. 给定索引处左右和的绝对差

让我们编写一个方法来计算给定索引处的绝对差，**正如我们所说，索引i处的右和等于数组总和减去索引i处的部分和**。此外，我们可以注意到，数组总和也是索引N-1处部分和的值：

```java
int absoluteDifferenceAtIndex(int[] partialSums, int index) {
    return Math.abs((partialSums[partialSums.length - 1] - partialSums[index]) - partialSums[index]);
}
```

我们已经以一种可以清楚地表明哪个和是右边的、哪个是左边的方式编写了该方法。

## 6. 恢复磁带平衡

现在我们可以回顾一下所有内容：

```java
int calculateTapeEquilibrium(int[] array) {
    int[] partialSums = new int[array.length];
    partialSums[0] = array[0];
    for (int i = 1; i < array.length; i++) {
        partialSums[i] = partialSums[i - 1] + array[i];
    }

    int minimalDifference = absoluteDifferenceAtIndex(partialSums, 0);
    for (int i = 1; i < array.length - 1; i++) {
        minimalDifference = Math.min(minimalDifference, absoluteDifferenceAtIndex(partialSums, i));
    }
    return minimalDifference;
}
```

**首先，我们计算部分和，然后遍历所有数组索引以寻找左右和之间的最小差异**。

由于我们将类命名为TapeEquilibriumSolver，现在可以检查我们得到的示例数组的正确结果：

```java
@Test
void whenCalculateTapeEquilibrium_thenReturnMinimalDifference() {
    int[] array = {-1, 6, 8, -4, 7};
    assertEquals(2, new TapeEquilibriumSolver().calculateTapeEquilibrium(array));
}
```

## 7. 总结

在本文中，我们用Java解决了著名的磁带平衡问题。在实现之前，我们确保选择了一种高效的算法。