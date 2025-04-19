---
layout: post
title:  在Java中查找数组的平衡索引
category: algorithms
copyright: algorithms
excerpt: 数组
---

## 1. 概述

在本教程中，我们首先学习[数组](https://www.baeldung.com/java-arrays-guide)平衡索引的定义，然后，我们将编写一个方法来识别和定位它们。

## 2. 问题的提出

**给定一个大小为N的0索引数组A，如果索引i较低元素的和等于索引i较高元素的和，则索引i为平衡索引**。也就是说：A[0\] + A[1\] + ... +A[i-1\] = A[i+1\] + A[i+2\] + ... +A[N-1\]。具体来说，对于数组的第一个和最后一个索引，其他元素的和应该为0。例如，考虑数组{1, -3, 0, 4, -5, 4, 0, 1, -2, -1}：

- 1是平衡索引，因为A[0\] = 1且A[2\] + A[3\] + A[4\] + A[5\] + A[6\] + A[7\] + A[8\] + A[9\] = 0 + 4 + (-5) + 4 + 0 + 1 + (-2) + (-1) = 1
- 4也是一个平衡索引，因为A[0\] + A[1\] + A[2\] + A[3\] = 1 + (-3) + 0 + 4 = 2且A[5\] + A[6\] + A[7\] + A[8\] + A[9\] = 4 + 0 + 1 + (-2) + (-1) = 2
- A[0\] + A[1\] + A[2\] + A[3\] + A[4\] + A[5\] + A[6\] + A[7\] + A[8\] = 1 + (-3) + 0 + 4 + (-5) + 4 + 0 + 1 + (-2) = 0并且没有索引大于9的元素，因此9也是该数组的平衡索引
- 另一方面，5不是平衡索引，因为A[0\] + A[1\] + A[2\] + A[3\] + A[4\] = 1 + (-3) + 0 + 4 + (-5) = -3，而A[6\] + A[7\] + A[8\] + A[9\] =0 + 1 + (-2) + (-1) = -2

## 3. 算法

让我们思考一下如何找到一个数组的平衡索引，首先想到的解决方案可能是遍历所有元素，然后计算两个[元素的和](https://www.baeldung.com/java-array-sum-average#find-sum)。然而，这将导致对数组元素进行内部迭代，从而影响算法的性能。

因此，我们最好从计算数组的所有部分和开始。**索引i处的部分和是A中所有索引小于或等于i的元素之和**，我们可以在初始数组上进行一次唯一的迭代来执行此操作。然后，我们会注意到，得益于部分和数组，我们可以获得所需的两个和：

- 求部分和数组中索引i-1处较低索引元素的和；否则，如果i = 0，则为0
- 较高索引元素的总和等于数组的总和减去直到索引i的所有数组元素的总和，或者用数学术语来说：A[i+1\] + A[i+2\] + ... +A[N-1\] = A[0\] + A[1\] + ... +A[i-1\] + A[i\] + A[i+1\] + ... + A[N-1\] – (A[0\] + A[1\] + ... + A[i\])，数组的总和是索引N-1处部分和数组的值，第二个和是索引i处部分和数组的值

之后，我们只需遍历该数组，如果两个表达式相等，则将元素添加到平衡索引列表中。因此，我们算法的复杂度为O(N)。

## 4. 计算部分和

除了部分和之外，0是A中索引0之前元素的和。此外，0是累加和的自然起点。因此，**在部分和数组的开头添加一个值为0的元素看起来很方便**：

```java
int[] partialSums = new int[array.length + 1];
partialSums[0] = 0;
for (int i=0; i<array.length; i++) {
    partialSums[i+1] = partialSums[i] + array[i]; 
}
```

简而言之，在我们的实现中，部分和数组在索引i+1处包含和A[0\] + A[1\] + ... + A[i\] 。换句话说，**部分和数组的第i个值等于A中所有索引低于i的元素之和**。

## 5. 列出所有平衡索引

**现在我们可以迭代我们的初始数组并确定给定的索引是否是平衡的**：

```java
List<Integer> equilibriumIndexes = new ArrayList<Integer>();
for (int i=0; i<array.length; i++) {
    if (partialSums[i] == (partialSums[array.length] - (partialSums[i+1]))) {
        equilibriumIndexes.add(i);
    }
}
```

可以看到，我们在结果[列表](https://www.baeldung.com/java-arraylist)中收集了所有符合条件的元素。

让我们从整体上看一下我们的方法：

```java
List<Integer> findEquilibriumIndexes(int[] array) {
    int[] partialSums = new int[array.length + 1];
    partialSums[0] = 0;
    for (int i=0; i<array.length; i++) {
        partialSums[i+1] = partialSums[i] + array[i]; 
    }
        
    List<Integer> equilibriumIndexes = new ArrayList<Integer>();
    for (int i=0; i<array.length; i++) {
        if (partialSums[i] == (partialSums[array.length] - (partialSums[i+1]))) {
            equilibriumIndexes.add(i);
        }
    }
    return equilibriumIndexes;
}
```

由于我们将类命名为EquilibriumIndexFinder，我们现在可以在示例数组上对我们的方法进行[单元测试](https://www.baeldung.com/java-unit-testing-best-practices)：

```java
@Test
void givenArrayHasEquilibriumIndexes_whenFindEquilibriumIndexes_thenListAllEquilibriumIndexes() {
    int[] array = {1, -3, 0, 4, -5, 4, 0, 1, -2, -1};
    assertThat(new EquilibriumIndexFinder().findEquilibriumIndexes(array)).containsExactly(1, 4, 9);
}
```

我们使用[AssertJ](https://www.baeldung.com/introduction-to-assertj)检查输出列表是否包含正确的索引。

## 6. 总结

在本文中，我们设计并实现了一种算法，用于查找Java数组的所有平衡索引。数据结构不一定是数组，也可以是List或任何有序的整数序列。