---
layout: post
title:  Java中数组的排列
category: algorithms
copyright: algorithms
excerpt: 排列
---

## 1. 简介

在本文中，我们将研究如何[创建数组的排列](https://www.baeldung.com/cs/array-generate-all-permutations)。

首先，我们将定义什么是排列。其次，我们将讨论一些约束。最后，**我们将讨论三种计算排列的方法：递归、迭代和随机**。

我们将重点关注Java中的实现，因此不会涉及太多数学细节。

## 2. 什么是排列？

集合的排列是对其元素的重新排列，由n个元素组成的集合有n!个排列，其中n!是阶乘，是所有小于或等于n的正整数的乘积。

### 2.1 示例

整数数组[3,4,7\]有三个元素和六个排列：

n! =3! = 1 x 2 x 3 = 6

排列：[3,4,7\]；[3,7,4\]；[4,7,3\]；[4,3,7\]；[7,3,4\]；[7,4,3\]

### 2.2 约束

排列的数量随着n的增加而快速增加。虽然生成10个元素的所有排列仅需几秒钟，但生成15个元素的所有排列则需要两周时间：

![](/assets/images/2025/algorithms/javaarraypermutations01.png)

## 3. 算法

### 3.1 递归算法

我们看的第一个算法是[堆算法](https://en.wikipedia.org/wiki/Heap's_algorithm)，**它是一种递归算法，通过每次迭代交换一个元素来产生所有排列**。

输入数组将被修改，如果我们不想这样，我们需要在调用该方法之前创建数组的副本：
```java
public static <T> void printAllRecursive(int n, T[] elements, char delimiter) {

    if(n == 1) {
        printArray(elements, delimiter);
    } else {
        for(int i = 0; i < n-1; i++) {
            printAllRecursive(n - 1, elements, delimiter);
            if(n % 2 == 0) {
                swap(elements, i, n-1);
            } else {
                swap(elements, 0, n-1);
            }
        }
        printAllRecursive(n - 1, elements, delimiter);
    }
}
```

该方法使用两个辅助方法：
```java
private static void swap(T[] elements, int a, int b) {
    T tmp = elements[a];
    elements[a] = elements[b];
    elements[b] = tmp;
}
```

```java
private static void printArray(T[] elements, char delimiter) {
    String delimiterSpace = delimiter + " ";
    for(int i = 0; i < elements.length; i++) {
        System.out.print(elements[i] + delimiterSpace);
    }
    System.out.print('\n');
}
```

在这里，我们将结果写入System.out，但是，我们可以轻松地将结果存储在数组或列表中。

### 3.2 迭代算法

**堆算法也可以用迭代来实现**：
```java
int[] indexes = new int[n];
int[] indexes = new int[n];
for (int i = 0; i < n; i++) {
    indexes[i] = 0;
}

printArray(elements, delimiter);

int i = 0;
while (i < n) {
    if (indexes[i] < i) {
        swap(elements, i % 2 == 0 ?  0: indexes[i], i);
        printArray(elements, delimiter);
        indexes[i]++;
        i = 0;
    }
    else {
        indexes[i] = 0;
        i++;
    }
}
```

### 3.3 按字典顺序排列

如果元素是可比较的，我们可以生成按元素的自然顺序排序的排列：
```java
public static <T extends Comparable<T>> void printAllOrdered(T[] elements, char delimiter) {

    Arrays.sort(elements);
    boolean hasNext = true;

    while(hasNext) {
        printArray(elements, delimiter);
        int k = 0, l = 0;
        hasNext = false;
        for (int i = elements.length - 1; i > 0; i--) {
            if (elements[i].compareTo(elements[i - 1]) > 0) {
                k = i - 1;
                hasNext = true;
                break;
            }
        }

        for (int i = elements.length - 1; i > k; i--) {
            if (elements[i].compareTo(elements[k]) > 0) {
                l = i;
                break;
            }
        }

        swap(elements, k, l);
        Collections.reverse(Arrays.asList(elements).subList(k + 1, elements.length));
    }
}
```

该算法在每次迭代中都有一个逆操作；因此，它在数组上的效率低于堆的算法。

### 3.4 随机化算法

如果n很大，我们可以通过打乱数组来生成随机排列：
```java
Collections.shuffle(Arrays.asList(elements));
```

我们可以重复此操作几次来生成排列样本。

我们可能会多次创建相同的排列，但是，对于较大的n值，生成两次相同排列的机会很低。

## 4. 总结

有很多方法可以生成数组的所有排列。在本文中，我们看到了递归和迭代堆算法以及如何生成有序的排列列表。

对于大型数组，生成所有排列是不可行的，因此，我们可以生成随机排列。