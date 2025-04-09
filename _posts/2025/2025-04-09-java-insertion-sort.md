---
layout: post
title:  Java中的插入排序
category: algorithms
copyright: algorithms
excerpt: 排序
---

## 1. 概述

在本教程中，我们将讨论插入排序算法并研究其Java实现。

插入排序是一种高效的排序算法，适用于对少量元素进行排序，该方法基于纸牌玩家对一手扑克牌进行排序的方式。

我们先从空着的左手开始，将牌放在桌子上，然后我们一次从桌子上取出一张牌，并将其插入左手的正确位置。为了找到新牌的正确位置，我们会将其与手中已经排序好的牌组从右到左进行比较。

让我们首先了解伪代码形式的算法步骤。

## 2. 伪代码

我们将用一个名为INSERTION-SORT的过程来呈现插入排序的伪代码，其参数是一个包含n个待排序元素的数组A[1..n\]，**该算法对输入数组进行就地排序**(通过重新排列数组A中的元素)。

该过程完成后，输入数组A包含输入序列的排列，但按排序顺序排列：
```text
INSERTION-SORT(A)

for i=2 to A.length
    key = A[i]
    j = i - 1 
    while j > 0 and A[j] > key
        A[j+1] = A[j]
        j = j - 1
    A[j + 1] = key
```

让我们简单回顾一下上面的算法。

索引i表示要处理的数组中当前项的位置。

我们从第二项开始，因为根据定义，只有一个元素的数组被认为是已排序的。索引i处的元素称为键，一旦有了键，算法的第二部分就是找到它的正确索引。**如果键小于索引j处元素的值，则键向左移动一个位置**，该过程持续进行，直到找到一个小于键的元素。

值得注意的是，在开始迭代查找索引i处键的正确位置之前，数组A[1..j –1\]已经排序。

## 3. 命令式执行

对于命令式的情况，我们将编写一个名为replacementSortImperative的函数，以整数数组作为参数，该函数从第二项开始迭代数组。

在迭代期间的任何给定时间，**我们可以将该数组视为在逻辑上被分为两部分**；左侧是已排序的部分，右侧包含尚未排序的元素。

这里需要注意的是，找到插入新元素的正确位置后，我们要将元素向右移动(而不是交换)以释放空间。
```java
public static void insertionSortImperative(int[] input) {
    for (int i = 1; i < input.length; i++) {
        int key = input[i];
        int j = i - 1;
        while (j >= 0 && input[j] > key) {
            input[j + 1] = input[j];
            j = j - 1;
        }
        input[j + 1] = key;
    }
}
```

接下来，让我们对上述方法创建一个测试：
```java
@Test
void givenUnsortedArray_whenInsertionSortImperative_thenSortedAsc() {
    int[] input = {6, 2, 3, 4, 5, 1};
    InsertionSort.insertionSortImperative(input);
    int[] expected = {1, 2, 3, 4, 5, 6};
    assertArrayEquals("the two arrays are not equal", expected, input);
}
```

上述测试证明算法对输入数组<6,2,3,4,5,1\>按升序正确排序。

## 4. 递归实现

递归情况的函数称为insertSortRecursive，并接收整数数组作为输入(与命令式情况相同)。

这里与命令式情况的区别(尽管它是递归的)在于，**它调用一个重载函数，其第2个参数等于要排序的元素数**。

由于我们要对整个数组进行排序，因此我们将传递与其长度相等的元素数：
```java
public static void insertionSortRecursive(int[] input) {
    insertionSortRecursive(input, input.length);
}
```

递归情况稍微有点挑战性，**基本情况发生在我们尝试对一个只有一个元素的数组进行排序时**，在这种情况下，我们什么也不做。

所有后续递归调用对输入数组的预定义部分进行排序-从第二项开始直到到达数组末尾：
```java
private static void insertionSortRecursive(int[] input, int i) {
    if (i <= 1) {
        return;
    }
    insertionSortRecursive(input, i - 1);
    int key = input[i - 1];
    int j = i - 2;
    while (j >= 0 && input[j] > key) {
        input[j + 1] = input[j];
        j = j - 1;
    }
    input[j + 1] = key;
}
```

对于包含6个元素的输入数组，调用栈如下所示：
```text
insertionSortRecursive(input, 6)
insertionSortRecursive(input, 5) and insert the 6th item into the sorted array
insertionSortRecursive(input, 4) and insert the 5th item into the sorted array
insertionSortRecursive(input, 3) and insert the 4th item into the sorted array
insertionSortRecursive(input, 2) and insert the 3rd item into the sorted array
insertionSortRecursive(input, 1) and insert the 2nd item into the sorted array
```

我们来看看它的测试：
```java
@Test
void givenUnsortedArray_whenInsertionSortRecursively_thenSortedAsc() {
    int[] input = {6, 4, 5, 2, 3, 1};
    InsertionSort.insertionSortRecursive(input);
    int[] expected = {1, 2, 3, 4, 5, 6};
    assertArrayEquals("the two arrays are not equal", expected, input);
}
```

上述测试证明算法对输入数组<6,2,3,4,5,1\>按升序正确排序。

## 5. 时间和空间复杂度

**插入排序运行所需的时间为O(n^2)**，对于每个新元素，我们从右到左遍历数组中已排序的部分以找到其正确位置，然后我们通过将元素向右移动一个位置来插入它。

该算法进行就地排序，因此**其空间复杂度对于命令式实现为O(1)，对于递归实现为O(n)**。

## 6. 总结

在本教程中，我们了解了如何实现插入排序。

此算法适用于对少量元素进行排序，**当对包含超过100个元素的输入序列进行排序时，该算法效率会变得很低**。 

请记住，尽管它的复杂度为二次方，但它可以在不需要辅助空间的情况下进行就地排序，就像归并排序一样。