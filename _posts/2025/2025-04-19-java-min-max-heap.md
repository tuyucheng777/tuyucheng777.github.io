---
layout: post
title:  如何在Java中实现最小最大堆
category: algorithms
copyright: algorithms
excerpt: 堆
---

## 1. 概述

在本教程中，我们将研究如何在Java中实现最小最大[堆](https://www.baeldung.com/cs/heap-vs-binary-search-tree)。

## 2. 最小最大堆

首先我们来了解一下堆的定义和特点，最小最大堆是一个完全[二叉树](https://www.baeldung.com/cs/binary-tree-intro)，同时具备最小堆和最大堆的特征：

![](/assets/images/2025/algorithms/javaminmaxheap01.png)

正如我们上面所看到的，**树中偶数级别的每个节点都小于其所有后代，而树中奇数级别的每个节点都大于其所有后代，其中根位于0级别**。

最小最大堆中的每个节点都有一个数据成员，通常称为键。根节点拥有最小最大堆中最小的键，而第二层的两个节点中有一个节点拥有最大的键。对于最小最大堆中像X这样的每个节点：

- 如果X位于最小(或偶数)级别，则X.key是根X的子树中所有键中的最小键
- 如果X位于最大(或奇数)级别，则X.key是根X的子树中所有键中的最大键

与最小堆或最大堆类似，插入和删除可以在O(logN)的[时间复杂度](https://www.baeldung.com/cs/time-vs-space-complexity)内进行。

## 3. Java实现

让我们从一个代表最小最大堆的简单类开始：

```java
public class MinMaxHeap<T extends Comparable<T>> {
    private List<T> array;
    private int capacity;
    private int indicator;
}
```

如上所示，我们使用一个indicator来确定添加到数组的最后一个元素的索引。但在继续之前，**我们需要记住数组索引从0开始，但我们假设堆中的索引从1开始**。

我们可以使用以下方法找到左孩子和右孩子的索引：

```java
private int getLeftChildIndex(int i) {
   return 2 * i;
}

private int getRightChildIndex(int i) {
    return ((2 * i) + 1);
}
```

同样，我们可以通过以下代码找到数组中元素的父级和祖父级的索引：

```java
private int getParentIndex(int i) {
   return i / 2;
}

private int getGrandparentIndex(int i) {
   return i / 4;
}
```

现在，让我们继续完成简单的最小最大堆类：

```java
public class MinMaxHeap<T extends Comparable<T>> {
    private List<T> array;
    private int capacity;
    private int indicator;

    MinMaxHeap(int capacity) {
        array = new ArrayList<>();
        this.capacity = capacity;
        indicator = 1;
    }

    MinMaxHeap(List<T> array) {
        this.array = array;
        this.capacity = array.size();
        this.indicator = array.size() + 1;
    }
}
```

我们可以通过两种方式创建最小最大堆的实例，首先，我们用[ArrayList](https://www.baeldung.com/java-arraylist)和特定容量初始化一个数组；其次，我们根据现有数组创建一个最小最大堆。

现在，让我们讨论一下堆上的操作。

### 3.1 创建

首先我们来看一下如何从一个现有的数组构建一个最小最大堆，这里我们使用了Floyd算法，并对其进行了一些改进，例如[Heapify算法](https://www.baeldung.com/cs/binary-tree-max-heapify)：

```java
public List<T> create() {
    for (int i = Math.floorDiv(array.size(), 2); i >= 1; i--) {
        pushDown(array, i);
    }
    return array;
}
```

让我们通过仔细观察以下代码中的pushDown来看看上面的代码中到底发生了什么：

```java
private void pushDown(List<T> array, int i) {
    if (isEvenLevel(i)) {
        pushDownMin(array, i);
    } else {
        pushDownMax(array, i);
    }
}
```

可以看到，对于所有偶数层，我们用pushDownMin检查数组项。这个算法类似于堆化，我们将用它来进行removeMin和removeMax的操作：

```java
private void pushDownMin(List<T> h, int i) {
    while (getLeftChildIndex(i) < indicator) {
        int indexOfSmallest = getIndexOfSmallestChildOrGrandChild(h, i);
        //...
        i = indexOfSmallest;
    }
}
```

首先，我们找到第'i'个元素的最小子元素或孙子元素的索引。然后，我们根据以下条件进行处理。

**如果最小的子元素或孙子元素不小于当前元素，则中断。换句话说，当前元素的排列类似于最小堆**：

```java
if (h.get(indexOfSmallest - 1).compareTo(h.get(i - 1)) < 0) {
    // ...
} else {
    break;
}
```

**如果最小的子元素或孙子元素小于当前元素，则将其与其父元素或祖父元素交换**：

```java
if (getParentIndex(getParentIndex(indexOfSmallest)) == i) {
    if (h.get(indexOfSmallest - 1).compareTo(h.get(i - 1)) < 0) {
        swap(indexOfSmallest - 1, i - 1, h);
        if (h.get(indexOfSmallest - 1).compareTo(h.get(getParentIndex(indexOfSmallest) - 1)) > 0) {
            swap(indexOfSmallest - 1, getParentIndex(indexOfSmallest) - 1, h);
        }
    }
    else if (h.get(indexOfSmallest - 1).compareTo(h.get(i - 1)) < 0) {
        swap(indexOfSmallest - 1, i - 1, h);
    }  
}
```

**我们将继续上述操作，直到找到元素“i”的子元素**。

现在，让我们看看getIndexOfSmallestChildOrGrandChild是如何工作的。其实很简单，首先，我们假设左孩子具有最小值，然后将它与其他孩子进行比较：

```java
private int getIndexOfSmallestChildOrGrandChild(List<T> h, int i) {
    int minIndex = getLeftChildIndex(i);
    T minValue = h.get(minIndex - 1);
    // rest of the implementation
}
```

**在每一步中，如果索引大于indicator，则最后找到的最小值就是答案**。

例如，让我们将最小值与右孩子进行比较：

```java
if (getRightChildIndex(i) < indicator) {
    if (h.get(getRightChildIndex(i) - 1).compareTo(minValue) < 0) {
        minValue = h.get(getRightChildIndex(i));
        minIndex = getRightChildIndex(i);
    }
} else {
     return minIndex;
}
```

现在，让我们创建一个测试来验证从无序数组创建最小最大堆是否可以正常工作：

```java
@Test
public void givenUnOrderedArray_WhenCreateMinMaxHeap_ThenIsEqualWithMinMaxHeapOrdered() {
    List<Integer> list = Arrays.asList(34, 12, 28, 9, 30, 19, 1, 40);
    MinMaxHeap<Integer> minMaxHeap = new MinMaxHeap<>(list);
    minMaxHeap.create();
    Assert.assertEquals(List.of(1, 40, 34, 9, 30, 19, 28, 12), list);
}
```

**pushDownMax的算法与pushDownMin的算法相同，但是所有的比较、运算符都是相反的**。

### 3.2 插入

让我们看看如何向最小最大堆添加元素：

```java
public void insert(T item) {
    if (isEmpty()) {
        array.add(item);
        indicator++;
    } else if (!isFull()) {
        array.add(item);
        pushUp(array, indicator);
        indicator++;
    } else {
        throw new RuntimeException("invalid operation !!!");
    }
}
```

首先，我们检查堆是否为空。如果堆为空，则添加新元素并自增indicator值。否则，添加的新元素可能会改变最小最大堆的顺序，因此我们需要使用pushUp来调整堆：

```java
private void pushUp(List<T>h,int i) {
    if (i != 1) {
        if (isEvenLevel(i)) {
            if (h.get(i - 1).compareTo(h.get(getParentIndex(i) - 1)) < 0) {
                pushUpMin(h, i);
            } else {
                swap(i - 1, getParentIndex(i) - 1, h);
                i = getParentIndex(i);
                pushUpMax(h, i);
            }
        } else if (h.get(i - 1).compareTo(h.get(getParentIndex(i) - 1)) > 0) {
            pushUpMax(h, i);
        } else {
            swap(i - 1, getParentIndex(i) - 1, h);
            i = getParentIndex(i);
            pushUpMin(h, i);
        }
    }
}
```

正如我们上面看到的，新元素与其父元素进行比较，然后：

- 如果发现它小于(大于)父元素，那么它肯定小于(大于)堆根路径上最大(最小)级别的所有其他元素。
- 从新元素到根的路径(仅考虑最小/最大层级)应该像插入之前一样按降序(升序)排列。因此，我们需要将新元素二叉插入到这个序列中。

现在，让我们看一下如下的pushUpMin：

```java
private void pushUpMin(List<T> h , int i) {
    while(hasGrandparent(i) && h.get(i - 1)
            .compareTo(h.get(getGrandparentIndex(i) - 1)) < 0) {
        swap(i - 1, getGrandparentIndex(i) - 1, h);
        i = getGrandparentIndex(i);
    }
}
```

**从技术上讲，当父元素较大时，将新元素与其父元素交换会更简单。此外，pushUpMax与pushUpMin相同，只是所有比较操作符都相反**。

现在，让我们创建一个测试来验证将新元素插入最小最大堆是否可以正常工作：

```java
@Test
public void givenNewElement_WhenInserted_ThenIsEqualWithMinMaxHeapOrdered() {
    MinMaxHeap<Integer> minMaxHeap = new MinMaxHeap(8);
    minMaxHeap.insert(34);
    minMaxHeap.insert(12);
    minMaxHeap.insert(28);
    minMaxHeap.insert(9);
    minMaxHeap.insert(30);
    minMaxHeap.insert(19);
    minMaxHeap.insert(1);
    minMaxHeap.insert(40);
    Assert.assertEquals(List.of(1, 40, 28, 12, 30, 19, 9, 34), minMaxHeap.getMinMaxHeap());
}
```

### 3.3 查找最小值

最小最大堆中的主要元素始终位于根部，因此我们可以在时间复杂度O(1)内找到它：

```java
public T min() {
    if (!isEmpty()) {
        return array.get(0);
    }
    return null;
}
```

### 3.4 查找最大值

最小最大堆中的最大元素总是位于第一个奇数层，因此我们可以通过简单的比较以时间复杂度O(1)找到它：

```java
public T max() {
    if (!isEmpty()) {
        if (indicator == 2) {
            return array.get(0);
        }
        if (indicator == 3) {
            return array.get(1);
        }
        return array.get(1).compareTo(array.get(2)) < 0 ? array.get(2) : array.get(1);
    }
    return null;
}
```

### 3.5 删除最小值

在这种情况下，我们将找到最小元素，然后将其替换为数组的最后一个元素：

```java
public T removeMin() {
    T min = min();
    if (min != null) {
        if (indicator == 2) {
            array.remove(indicator--);
            return min;
        }
        array.set(0, array.get(--indicator - 1));
        array.remove(indicator - 1);
        pushDown(array, 1);
    }
    return min;
}
```

### 3.6 删除最大值

删除最大元素与删除最小元素相同，唯一的变化是我们找到最大元素的索引，然后调用pushDown：

```java
public T removeMax() {
    T max = max();
    if (max != null) {
        int maxIndex;
        if (indicator == 2) {
            maxIndex = 0;
            array.remove(--indicator - 1);
            return max;
        } else if (indicator == 3) {
            maxIndex = 1;
            array.remove(--indicator - 1);
            return max;
        } else {
            maxIndex = array.get(1).compareTo(array.get(2)) < 0 ? 2 : 1;
        }
        array.set(maxIndex, array.get(--indicator - 1));
        array.remove(indicator - 1);
        pushDown(array, maxIndex + 1);
    }
    return max;
}
```

## 4. 总结

在本教程中，我们看到了如何在Java中实现最小最大堆，并探索了一些最常见的操作。

首先，我们了解了最小-最大堆的本质，包括一些最常见的特性。然后，我们学习了如何在最小-最大堆的实现中创建、插入、查找最小值、查找最大值、移除最小值和移除最大值元素。