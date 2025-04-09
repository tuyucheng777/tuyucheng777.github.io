---
layout: post
title:  Java中使用堆计算整数流的中位数
category: algorithms
copyright: algorithms
excerpt: 中位数
---

## 1. 概述

在本教程中，**我们将学习如何计算整数流的中位数**。

我们将首先通过示例陈述问题，然后分析问题，最后用Java实现几种解决方案。

## 2. 问题陈述

**[中位数](http://www.icoachmath.com/math_dictionary/median.html)是有序数据集的中间值，对于一组整数，小于中位数的元素与大于中位数的元素一样多**。

在有序集合中：

- 奇数个整数，中间元素为中位数-在有序集合{5,7,10}中，中位数为7
- 偶数个整数，没有中间元素；中位数计算为两个中间元素的平均值-在有序集合{5,7,8,10}中，中位数为(7 + 8) / 2 = 7.5

现在，假设我们读取的不是有限集，而是数据流中的整数，我们可以**将整数流的中位数定义为迄今为止读取的整数集的中位数**。

让我们将问题陈述形式化，给定一个整数流的输入，我们必须设计一个类，针对我们读取的每个整数执行以下两个任务：

1. 将整数添加到整数集合中
2. 找到迄今为止读取的整数的中位数

例如：
```text
add 5         // sorted-set = { 5 }, size = 1
get median -> 5

add 7         // sorted-set = { 5, 7 }, size = 2 
get median -> (5 + 7) / 2 = 6

add 10        // sorted-set = { 5, 7, 10 }, size = 3 
get median -> 7

add 8         // sorted-set = { 5, 7, 8, 10 }, size = 4 
get median -> (7 + 8) / 2 = 7.5
..
```

尽管流是非有限的，但我们可以假设我们可以一次将流的所有元素保存在内存中。

我们可以用代码将我们的任务表示为以下操作：
```java
void add(int num);

double getMedian();
```

## 3. 简单方法

### 3.1. 排序列表

让我们从一个简单的想法开始-我们可以通过索引访问列表的中间元素或中间两个元素来计算有序整数列表的中位数，getMedian操作的时间复杂度为O(1)。

在添加新整数时，我们必须确定其在列表中的正确位置，以使列表保持排序。此操作可以在O(n)时间内完成，其中n是列表的大小。因此，向列表添加新元素并计算新中位数的总成本为O(n)。

### 3.2 改进简单方法

添加操作的运行时间是线性的，这并非最优方案，本节我们将尝试解决这个问题。

我们可以将列表拆分为两个已排序的列表：**较小的一半整数按降序排列，较大的一半整数按升序排列**；我们可以将一个新的整数添加到适当的一半，使两个列表的大小最多相差1：
```text
if element is smaller than min. element of larger half:
    insert into smaller half at appropriate index
    if smaller half is much bigger than larger half:
        remove max. element of smaller half and insert at the beginning of larger half (rebalance)
else
    insert into larger half at appropriate index:
    if larger half is much bigger than smaller half:
        remove min. element of larger half and insert at the beginning of smaller half (rebalance)

```

![](/assets/images/2025/algorithms/javastreamintegersmedianusingheap01.png)

现在，我们可以计算中位数：
```text
if lists contain equal number of elements:
    median = (max. element of smaller half + min. element of larger half) / 2
else if smaller half contains more elements:
    median = max. element of smaller half
else if larger half contains more elements:
    median = min. element of larger half
```

尽管我们只是将add运算的时间复杂度提高了一些常数倍，但我们已经取得了进展。

让我们分析一下在两个排序列表中访问的元素，在(排序的)add操作期间，我们可能会在移动每个元素时访问它们。更重要的是，在重新平衡的add操作和getMedian操作期间，我们分别访问较大和较小一半的最小值和最大值(极值)。

我们可以看到，**极值是其各自列表的第一个元素**。因此，我们必须优化访问每一半索引0处的元素，以改善add操作的总体运行时间。

## 4. 基于堆的方法

让我们运用从简单方法中学到的知识来完善对问题的理解：

1. 我们必须在O(1)时间内获取数据集的最小/最大元素
2. 只要我们能有效地获取最小/最大元素，元素就不必保持排序顺序
3. 我们需要找到一种向数据集添加元素的方法，并且其时间成本小于O(n)

接下来，我们来了解一下能够帮助我们高效实现目标的堆数据结构。

### 4.1 堆数据结构

**[堆](https://www.baeldung.com/java-heap-sort#heap-data-structure)是一种通常用数组实现但可以被认为是二叉树的数据结构**。

![](/assets/images/2025/algorithms/javastreamintegersmedianusingheap02.png)

堆受到堆属性的约束：

#### 4.1.1 最大堆

子节点的值不能大于其父节点的值，因此，在最大堆中，根节点的值始终是最大的。

#### 4.1.2 最小堆

父节点的值不能大于其子节点的值，因此，在最小堆中，根节点的值始终是最小的。

在Java中，[PriorityQueue](https://algs4.cs.princeton.edu/24pq/)类表示一个堆，让我们继续讨论第一个使用堆的解决方案。

### 4.2 第一个解决方案

让我们用两个堆替换我们简单方法中的列表：

- 包含较大一半元素的最小堆，最小元素位于根
- 包含较小一半元素的最大堆，最大元素位于根

现在，我们可以通过比较传入的整数与最小堆的根，将其添加到相应的一半。接下来，如果插入后，一个堆的大小与另一个堆的大小相差超过1，我们可以重新平衡这两个堆，从而保持大小差最多为1：
```text
if size(minHeap) > size(maxHeap) + 1:
    remove root element of minHeap, insert into maxHeap
if size(maxHeap) > size(minHeap) + 1:
    remove root element of maxHeap, insert into minHeap
```

通过这种方法，如果两个堆的大小相等，我们可以将中位数计算为两个堆根元素的平均值。否则，**元素较多的堆的根元素就是中位数**。

我们将使用PriorityQueue类来表示堆，PriorityQueue的默认堆属性是最小堆，也可以使用Comparator.reverserOrder(使用自然顺序的逆序)来创建最大堆：
```java
class MedianOfIntegerStream {

    private Queue<Integer> minHeap, maxHeap;

    MedianOfIntegerStream() {
        minHeap = new PriorityQueue<>();
        maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
    }

    void add(int num) {
        if (!minHeap.isEmpty() && num < minHeap.peek()) {
            maxHeap.offer(num);
            if (maxHeap.size() > minHeap.size() + 1) {
                minHeap.offer(maxHeap.poll());
            }
        } else {
            minHeap.offer(num);
            if (minHeap.size() > maxHeap.size() + 1) {
                maxHeap.offer(minHeap.poll());
            }
        }
    }

    double getMedian() {
        double median;
        if (minHeap.size() < maxHeap.size()) {
            median = maxHeap.peek();
        } else if (minHeap.size() > maxHeap.size()) {
            median = minHeap.peek();
        } else {
            median = (minHeap.peek() + maxHeap.peek()) / 2.0;
        }
        return median;
    }
}
```

在分析代码的运行时间之前，我们先来看看我们使用的堆操作的时间复杂度：
```text
find-min/find-max        O(1)    

delete-min/delete-max    O(log n)

insert                   O(log n)
```

因此，getMedian操作可以在O(1)时间内完成，因为它只需要find-min和find-max函数。add操作的时间复杂度为O(log n)-3次insert/delete调用，每次都需要O(log n)时间。

### 4.3 堆大小不变解决方案

在之前的方法中，我们将每个新元素与堆的根元素进行比较。让我们探索另一种使用堆的方法，利用堆的性质在适当的一半位置添加新元素。

正如我们之前的解决方案一样，我们从两个堆开始-一个最小堆和一个最大堆。接下来，引入一个条件：**最大堆的大小必须始终为(n / 2)，而最小堆的大小可以是(n / 2)或(n / 2) + 1，具体取决于两个堆中的元素总数**。换句话说，当元素总数为奇数时，我们只允许那个额外的元素在最小堆中。

在堆大小不变的情况下，如果两个堆的大小都是(n / 2)，我们可以将中位数计算为两个堆根元素的平均值。否则，**最小堆的根元素就是中位数**。

当我们添加一个新的整数时，我们有两种情况：
```text
1. Total no. of existing elements is even
   size(min-heap) == size(max-heap) == (n / 2)

2. Total no. of existing elements is odd
   size(max-heap) == (n / 2)
   size(min-heap) == (n / 2) + 1
```

我们可以通过向其中一个堆中添加新元素并每次重新平衡来保持不变性：

![](/assets/images/2025/algorithms/javastreamintegersmedianusingheap03.png)

重新平衡的工作原理是将最大元素从最大堆移到最小堆，或将最小元素从最小堆移到最大堆。这样，虽然我们在将新整数添加到堆之前没有对其进行比较，但随后的重新平衡可确保我们遵守较小和较大两半的底层不变量。

让我们使用PriorityQueues在Java中实现我们的解决方案：
```java
class MedianOfIntegerStream {

    private Queue<Integer> minHeap, maxHeap;

    MedianOfIntegerStream() {
        minHeap = new PriorityQueue<>();
        maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
    }

    void add(int num) {
        if (minHeap.size() == maxHeap.size()) {
            maxHeap.offer(num);
            minHeap.offer(maxHeap.poll());
        } else {
            minHeap.offer(num);
            maxHeap.offer(minHeap.poll());
        }
    }

    double getMedian() {
        double median;
        if (minHeap.size() > maxHeap.size()) {
            median = minHeap.peek();
        } else {
            median = (minHeap.peek() + maxHeap.peek()) / 2.0;
        }
        return median;
    }
}
```

**我们的操作的时间复杂度保持不变**：getMedian花费时间为O(1)，而add的运行时间为O(log n)，且操作次数完全相同。

这两种基于堆的解决方案都提供了类似的空间和时间复杂度；虽然第二种解决方案很巧妙，并且实现更简洁，但这种方法并不直观。另一方面，第一种解决方案自然而然地遵循了我们的直觉，并且更容易推断出其add操作的正确性。

## 5. 总结

在本教程中，我们学习了如何计算整数流的中位数；我们评估了几种方法，并使用PriorityQueue在Java中实现了几种不同的解决方案。