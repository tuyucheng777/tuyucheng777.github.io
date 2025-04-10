---
layout: post
title:  Java实现的就地排序算法指南
category: algorithms
copyright: algorithms
excerpt: 排序
---

## 1. 简介

在本教程中，我们将解释就地排序算法的工作原理。

## 2. 就地算法

**就地算法是指那些不需要任何辅助数据结构来转换输入数据的算法**，简单来说，这意味着该算法不会占用额外的空间来处理输入，它实际上是用输出覆盖输入。

然而，实际上，算法实际上可能需要一个很小且非常量的额外空间来存放辅助变量。**在大多数情况下，这个空间的复杂度为O(log n)，尽管有时允许任何小于线性的复杂度**。

## 3. 伪代码

现在让我们看一些伪代码，并将就地算法与非就地算法进行比较。

假设我们想要反转一个包含n个数字的数组。

### 3.1 就地算法

如果我们仔细思考这个问题，就会发现我们有一个输入数组和一个反转后的数组作为输出。最后，我们实际上不需要原始数组，只需要反转后的数组。

那么，为什么不覆盖输入，而是将其值移动到全新的数组中呢？这看起来似乎是最显而易见的方法。为此，我们只需要一个额外的变量来临时存储我们当前正在使用的值：
```text
reversInPlace(array A[n])
    for i from 0 to n/2
    temp = A[i]
    A[i] = A[n - 1 - i]
    A[n - 1 - i] = temp
```

值得一提的是，无论数组有多大，在这种情况下我们需要的额外空间始终是O(1)。

该图显示我们需要的步骤比以前的情况更少：

![](/assets/images/2025/algorithms/javainplacesorting01.png)

### 3.2 非就地算法

另一方面，我们也可以用一种非常简单、更明显的方式来做到这一点。我们可以创建一个相同大小的新数组，按相应的顺序从原始数组中复制值，然后删除原始数组：
```text
reverseOutOfPlace(array A[n])
    create new array B[n]
    for i from 0 to n - 1
        B[i] = A[i]
    delete A
    return B
```

虽然这可以达到我们的预期，但效率不够高。由于**我们有两个数组需要操作**，因此需要O(n)的额外空间。除此之外，创建和删除新数组通常也很慢。

让我们看一下该过程的图示：

![](/assets/images/2025/algorithms/javainplacesorting02.png)

## 4. Java实现

现在让我们看看如何用Java实现上一节所学的内容。

首先，我们将实现一个就地算法：
```java
public static int[] reverseInPlace(int A[]) {
    int n = A.length;
    for (int i = 0; i < n / 2; i++) {
        int temp = A[i];
        A[i] = A[n - 1 - i];
        A[n - 1 - i] = temp;
    }
    return A;
}
```

我们可以轻松测试它是否按预期工作：
```java
@Test
void givenArray_whenInPlaceSort_thenReversed() {
    int[] input = {1, 2, 3, 4, 5, 6, 7};
    int[] expected = {7, 6, 5, 4, 3, 2, 1};
    assertArrayEquals("the two arrays are not equal", expected, InOutSort.reverseInPlace(input));
}
```

其次，我们来检查一下不合适的算法实现：
```java
public static int[] reverseOutOfPlace(int A[]) {
    int n = A.length;
    int[] B = new int[n];
    for (int i = 0; i < n; i++) {
        B[n - i - 1] = A[i];
    }
    return B;
}
```

测试非常简单：
```java
@Test
void givenArray_whenOutOfPlaceSort_thenReversed() {
    int[] input = {1, 2, 3, 4, 5, 6, 7};
    int[] expected = {7, 6, 5, 4, 3, 2, 1};
    assertArrayEquals("the two arrays are not equal", expected, InOutSort.reverseOutOfPlace(input));
}
```

## 5. 示例

有许多排序算法使用就地方法，其中包括[插入排序](https://www.baeldung.com/java-insertion-sort)、[冒泡排序](https://www.baeldung.com/java-bubble-sort)、[堆排序](https://www.baeldung.com/java-heap-sort)、[快速排序](https://www.baeldung.com/java-quicksort)和[希尔排序](https://www.baeldung.com/java-shell-sort)，你可以了解有关它们的更多信息并查看它们的Java实现。

另外，我们需要提到梳排序和堆排序，它们的空间复杂度均为O(log n)。

学习有关[大O表示法](https://www.baeldung.com/big-o-notation)理论的更多信息以及查看[有关算法复杂度的一些实际Java示例](https://www.baeldung.com/java-algorithm-complexity)也很有用。

## 6. 总结

在本文中，我们描述了所谓的就地算法，使用伪代码和一些示例说明了它们的工作原理，列出了几种基于此原理的算法，最后用Java实现了基本示例。