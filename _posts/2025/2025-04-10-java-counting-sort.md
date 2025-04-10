---
layout: post
title:  Java中的计数排序
category: algorithms
copyright: algorithms
excerpt: 排序
---

## 1. 概述

通用排序算法(例如[归并排序](https://www.baeldung.com/java-merge-sort))不对输入做出任何假设，因此在最坏情况下无法超越O(n log n)。相反，计数排序对输入有一个假设，这使得它成为一个线性时间的排序算法。

在本教程中，我们将熟悉计数排序的机制，然后用Java实现它。

## 2. 计数排序

与大多数经典排序算法不同，计数排序不会通过比较元素对给定的输入进行排序。相反，**它假设输入元素是[0, k\]范围内的n个整数**，当k = O(n)时，计数排序将在O(n)时间内运行。

请注意，我们不能将计数排序用作通用排序算法。但是，当输入符合此假设时，它的速度相当快。

### 2.1 频率矩阵

假设我们要对[0,5\]范围内的值进行输入数组排序：[
](https://www.baeldung.com/wp-content/uploads/2019/09/counts.png)

![](/assets/images/2025/algorithms/javacountingsort01.png)

首先，**我们应该计算输入数组中每个数字的出现次数，如果我们用数组C表示计数，则C[i\]表示输入数组中数字i的频率**：

![](/assets/images/2025/algorithms/javacountingsort02.png)

例如，由于5在输入数组中出现了3次，因此索引5的值等于3。

**现在给定数组C，我们需要确定有多少个元素小于或等于每个输入元素**，例如：

- 一个元素小于或等于0，或者换句话说，只有一个0值，等于C[0\]
- 两个元素小于或等于1，等于C[0\] + C[1\]
- 四个值小于或等于2，等于C[0\] + C[1\] +C[2\]

**因此，如果我们继续计算C中n个连续元素的总和，我们就可以知道输入数组中有多少个元素小于或等于数字n - 1**。总之，通过应用这个简单的公式，我们可以将C更新如下：

![](/assets/images/2025/algorithms/javacountingsort03.png)

### 2.2 算法

现在我们可以使用辅助数组C对输入数组进行排序，计数排序的工作原理如下：

- 它反向迭代输入数组
- 对于每个元素i，C[i\] – 1表示排序数组中数字i的位置，这是因为有C[i\]个元素小于或等于i
- 然后，在每一轮结束时减小C[i\] 

为了对示例输入数组进行排序，我们首先应该从数字5开始，因为它是最后一个元素。根据C[5\]，有11个元素小于或等于数字5。

所以，5应该是排序数组中的第11个元素，因此索引为10：

![](/assets/images/2025/algorithms/javacountingsort04.png)

由于我们将5移到了排序数组中，因此我们应该减小C[5\]的值。逆序排列的下一个元素是2，由于有4个元素小于或等于2，因此该数字应该是排序数组中的第4个元素：

![](/assets/images/2025/algorithms/javacountingsort05.png)

类似地，我们可以找到下一个元素的正确位置，即0：

![](/assets/images/2025/algorithms/javacountingsort06.png)

如果我们继续反向迭代并适当地移动每个元素，我们最终会得到如下结果：

![](/assets/images/2025/algorithms/javacountingsort07.png)

## 3. 计数排序–Java实现

### 3.1 计算频率矩阵

首先，给定一个输入元素数组和k，我们应该计算数组C：
```java
int[] countElements(int[] input, int k) {
    int[] c = new int[k + 1];
    Arrays.fill(c, 0);

    for (int i : input) {
        c[i] += 1;
    }

    for (int i = 1; i < c.length; i++) {
        c[i] += c[i - 1];
    }

    return c;
}
```

让我们分解一下方法签名：

- input表示我们要排序的数字数组
- input数组是[0,k\]范围内的整数数组，因此k表示input中的最大数字 
- 返回类型是一个表示C数组的整数数组

countElements方法的工作原理如下：

- 首先，我们初始化C数组，由于[0,k\]范围包含k + 1个数字，因此我们创建一个能够包含k + 1个数字的数组
- 然后对于 对于输入中的每个数字，我们计算该数字的频率
- 最后，我们将连续的元素相加，以了解有多少个元素小于或等于特定数字

另外，我们可以验证countElements方法是否按预期工作：
```java
@Test
void countElements_GivenAnArray_ShouldCalculateTheFrequencyArrayAsExpected() {
    int k = 5;
    int[] input = { 4, 3, 2, 5, 4, 3, 5, 1, 0, 2, 5 };

    int[] c = CountingSort.countElements(input, k);
    int[] expected = { 1, 2, 4, 6, 8, 11 };
    assertArrayEquals(expected, c);
}
```

### 3.2 对输入数组进行排序

现在我们可以计算频率数组，我们应该能够对任何给定的数字集进行排序：
```java
int[] sort(int[] input, int k) {
    int[] c = countElements(input, k);

    int[] sorted = new int[input.length];
    for (int i = input.length - 1; i >= 0; i--) {
        int current = input[i];
        sorted[c[current] - 1] = current;
        c[current] -= 1;
    }

    return sorted;
}
```

sort方法的工作原理如下：

- 首先，计算C数组
- 然后，它反向迭代input数组，并针对input中的每个元素，找到其在排序后数组中的正确位置。input中的第i个元素应为排序数组中的第C[i\]个元素。由于Java数组是从0开始索引的，因此C[i\] - 1项是第C[i\]个元素-例如，sorted[5\]是排序数组中的第6个元素 
- 每当我们找到一个匹配项，它就会减小相应的C[i\]值

类似地，我们可以验证sort方法是否按预期工作：
```java
@Test
void sort_GivenAnArray_ShouldSortTheInputAsExpected() {
    int k = 5;
    int[] input = { 4, 3, 2, 5, 4, 3, 5, 1, 0, 2, 5 };

    int[] sorted = CountingSort.sort(input, k);

    // Our sorting algorithm and Java's should return the same result
    Arrays.sort(input);
    assertArrayEquals(input, sorted);
}
```

## 4. 重新审视计数排序算法

### 4.1 复杂度分析

**大多数经典排序算法(如[归并排序](https://www.baeldung.com/java-merge-sort))都是通过比较输入元素对任何给定输入进行排序**，这类排序算法称为比较排序。在最坏的情况下，比较排序对n个元素进行排序的时间复杂度至少为O(n log n)。

另一方面，计数排序不通过比较输入元素对输入进行排序，因此它显然不是比较排序算法。

让我们看看对输入进行排序需要花费多少时间：

- 它在O(n + k)时间内计算C数组：它在O(n)中迭代一次大小为n的输入数组，然后在O(k)中迭代C-因此总共是O(n + k) 
- 计算出C后，它会通过迭代输入数组并在每次迭代中执行一些基本操作来对输入进行排序。因此，实际的排序操作需要O(n)

总的来说，计数排序需要O(n + k)的时间来运行：
```text
O(n + k) + O(n) = O(2n + k) = O(n + k)
```

**如果我们假设k = O(n)，则计数排序算法会以线性时间对输入进行排序**。与通用排序算法相反，计数排序对输入做出假设，并且执行时间小于O(n log n)下限。

###  4.2 稳定性

刚才，我们制定了一些关于计数排序机制的特殊规则，但始终未能阐明其背后的原因，更具体地说：

- **为什么我们应该反向迭代输入数组**？
- **为什么我们每次使用C[i\]时都会减小它**？

为了更好地理解第一条规则，让我们从头开始。假设我们要对一个简单的整数数组进行排序，如下所示：

![](/assets/images/2025/algorithms/javacountingsort08.png)

在第一次迭代中，我们应该找到第一个1的排序位置：

![](/assets/images/2025/algorithms/javacountingsort09.png)

因此，数字1的第一次出现将获得排序数组中的最后一个索引。跳过数字0，让我们看看数字1第二次出现时会发生什么：

![](/assets/images/2025/algorithms/javacountingsort10.png)

**输入和排序数组中具有相同值元素的出现顺序不同，因此当我们从头开始迭代时，算法并不[稳定](https://www.baeldung.com/stable-sorting-algorithms)**。

如果我们每次使用后不减小C[i\]的值，会发生什么？让我们看看：

![](/assets/images/2025/algorithms/javacountingsort11.png)

数字1的两次出现都位于排序数组的最后位置，因此，如果我们在每次使用后不减小C[i\]的值，我们可能会在排序时丢失一些数字。

## 5. 总结

在本教程中，我们首先学习了计数排序的内部工作原理。然后，我们用Java实现了该排序算法，并编写了一些测试来验证其行为。最后，我们证明了该算法是一种具有线性时间复杂度的稳定排序算法。