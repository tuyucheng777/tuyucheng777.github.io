---
layout: post
title:  Java中的二分查找算法
category: algorithms
copyright: algorithms
excerpt: 二分查找
---

## 1. 概述

在本文中，我们将介绍[二分搜索相对于简单线性搜索](https://www.baeldung.com/cs/linear-search-vs-binary-search)的优势，并介绍其在Java中的实现。

## 2. 高效搜索的需求

假设我们从事葡萄酒销售业务，每天有数百万买家访问我们的应用程序。

通过我们的应用程序，客户可以筛选出价格低于n美元的商品，从搜索结果中选择一瓶，然后将其添加到购物车中。我们每秒都有数百万用户在寻找限价葡萄酒，结果必须快速呈现。

在后端，我们的算法对整个葡萄酒列表进行线性搜索，将客户输入的价格限制与列表中每瓶葡萄酒的价格进行比较。

然后，返回价格小于或等于价格限制的商品，此线性搜索的时间复杂度为O(n)。

这意味着我们系统中的葡萄酒瓶数量越多，搜索所需的时间就越长，**时间会随着新引入的葡萄酒数量而增加**。

如果我们开始按排序顺序保存元素并使用二分搜索来搜索元素，我们可以实现O(log n)的复杂度。

**使用二分搜索，搜索结果所花费的时间自然会随着数据集的大小而增加，但并不是成比例的**。

## 3. 二分查找

简单来说，该算法将键值与数组中间元素进行比较，如果不相等，则排除键不能属于的那一半，继续在剩下的一半中搜索，直到成功。

请记住-这里的关键是数组已经排序。

如果搜索结束时剩余一半为空，则表示该键不在数组中。

### 3.1 迭代实现
```java
public int runBinarySearchIteratively(int[] sortedArray, int key, int low, int high) {
    int index = Integer.MAX_VALUE;

    while (low <= high) {
        int mid = low  + ((high - low) / 2);
        if (sortedArray[mid] < key) {
            low = mid + 1;
        } else if (sortedArray[mid] > key) {
            high = mid - 1;
        } else if (sortedArray[mid] == key) {
            index = mid;
            break;
        }
    }
    return index;
}
```

runBinarySearchIteratively方法接收sortedArray、key以及sortedArray的low和high索引作为参数，该方法首次运行时，sortedArray的第一个索引low为0，而sortedArray的最后一个索引high等于其长度-1。

middle是sortedArray的中间索引，现在，算法运行一个while循环，将键与sortedArray中间索引的数组值进行比较。

**注意中间索引(int mid = low + ((high – low) / 2)是如何生成的，这是为了适应极大的数组**。如果只是通过获取中间索引(int mid = (low + high) / 2)来生成中间索引，则对于包含2<sup>30</sup>或更多元素的数组可能会发生溢出，因为low + high的总和会容易超过最大正int值。

### 3.2 递归实现

现在，让我们看一个简单的递归实现：

```java
public int runBinarySearchRecursively(int[] sortedArray, int key, int low, int high) {
    int middle = low  + ((high - low) / 2);

    if (high < low) {
        return -1;
    }

    if (key == sortedArray[middle]) {
        return middle;
    } else if (key < sortedArray[middle]) {
        return runBinarySearchRecursively(sortedArray, key, low, middle - 1);
    } else {
        return runBinarySearchRecursively(sortedArray, key, middle + 1, high);
    }
}
```

runBinarySearchRecursively方法接收sortedArray、key以及sortedArray的low索引和high索引。

### 3.3 使用Arrays.binarySearch()
```java
int index = Arrays.binarySearch(sortedArray, key);
```

sortedArray和要在整数数组中搜索的int键作为参数传递给Java Arrays类的binarySearch方法。

### 3.4 使用Collections.binarySearch()
```java
int index = Collections.binarySearch(sortedList, key);
```

要在Integer对象列表中搜索的sortedList和Integer键作为参数传递给Java Collections类的binarySearch方法。

### 3.5  排序数组和重复项

如果数组是一个包含重复项的排序数组，并且我们使用前面讨论过的二分查找方法，它将返回key的任何出现位置。但是，如果我们需要找到key的第一次和最后一次出现位置，则需要修改传统的二分查找逻辑。找到key元素的第一次出现和最后一次出现位置，我们就可以得到key元素出现的整个窗口。

为了找到第一次出现的情况，我们首先对key元素进行二分搜索，然后继续在左半部分搜索：

```java
int startIndexSearch(int[] sortedArray, int target) {
    int left = 0;
    int right = sortedArray.length - 1;
    int result = -1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (sortedArray[mid] == target) {
            result = mid;
            right = mid - 1; // continue search on left half
        } else if (sortedArray[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }

    return result;
}
```

类似地，为了找到key元素的最后一次出现，我们可以修改传统的二分搜索算法，即使找到key元素后也继续在右半部分搜索：
```java
int endIndexSearch(int[] sortedArray, int target) {
    int left = 0;
    int right = sortedArray.length - 1;
    int result = -1;

    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (sortedArray[mid] == target) {
            result = mid;
            left = mid + 1; // continue search in the right half
        } else if (sortedArray[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }

    return result;
}
```

通过第一次和最后一次出现，我们可以获取sortedArray中所有key元素的索引：
```java
List<Integer> runBinarySearchOnSortedArraysWithDuplicates(int[] sortedArray, Integer key) {
    int startIndex = startIndexSearch(sortedArray, key);
    int endIndex = endIndexSearch(sortedArray, key);
    return IntStream.rangeClosed(startIndex, endIndex)
        .boxed()
        .collect(Collectors.toList());
}
```

在上面的例子中，我们从key元素的startIndex(第一次出现)到endIndex(最后一次出现)进行迭代，然后将结果收集到一个列表中。

### 3.6 性能

**使用递归或迭代方法编写算法主要取决于个人喜好**，但我们应注意以下几点：

1. 递归可能会比较慢，因为维护堆栈的开销较大，并且通常占用更多内存
2. 递归不是堆栈友好的，处理大数据集时可能会导致StackOverflowException
3. 递归使代码更清晰，因为与迭代方法相比，它代码更短

理想情况下，对于较大的n值，二分查找的比较次数会比线性查找少。对于较小的n值，线性查找的效果可能优于二分查找。

应该知道，这种分析是理论上的，并且可能根据具体情况而变化。

此外，**二分搜索算法需要一个已排序的数据集，而这也有成本**。如果我们使用合并排序算法对数据进行排序，则会给我们的代码增加n(log n)的额外复杂度。

因此，我们首先需要充分分析我们的需求，然后决定哪种搜索算法最适合我们的要求。

## 4. 总结

本教程演示了二分搜索算法的实现以及使用它代替线性搜索的场景。