---
layout: post
title:  Java中的插值搜索
category: algorithms
copyright: algorithms
excerpt: 搜索
---

## 1. 简介

在本教程中，我们将介绍[插值搜索](https://www.baeldung.com/cs/interpolation-search)算法并讨论其优缺点。此外，我们将用Java实现它并讨论该算法的时间复杂度。

## 2. 动机

**插值搜索是对[二分搜索](https://www.baeldung.com/java-binary-search)的改进，专门针对均匀分布的数据**。

二分查找无论数据分布如何，都会在每一步中将搜索空间减半，因此其时间复杂度始终为O(log(n))。

另一方面，**插值搜索的时间复杂度取决于数据分布**。对于均匀分布的数据，它比二分搜索更快，时间复杂度为O(log(log(n)))。然而，在最坏的情况下，它的性能可能低至O(n)。

## 3. 插值搜索

与二分搜索类似，插值搜索只能在已排序的数组上进行，它在每次迭代中将探针放置在计算出的位置，如果探针正好位于我们要查找的元素上，则返回该位置；否则，搜索空间将限制在探针的右侧或左侧。

**探针位置计算是二分搜索和插值搜索之间的唯一区别**。

在二分查找中，探测位置始终是剩余搜索空间的最中间项。

相反，插值搜索根据以下公式计算探针位置：

![](/assets/images/2025/algorithms/javainterpolationsearch01.png)

让我们看一下每个术语：

- probe：新的探针位置将被分配给此参数
- lowEnd：当前搜索空间中最左边项的索引
- highEnd：当前搜索空间中最右边项的索引
- data[]：包含原始搜索空间的数组
- item：我们正在寻找的元素

为了更好地理解插值搜索的工作原理，让我们通过一个例子来演示它。

假设我们想在下面的数组中找到84的位置：

![](/assets/images/2025/algorithms/javainterpolationsearch02.png)

数组的长度为8，因此最初highEnd = 7且lowEnd = 0(因为数组的索引从0开始，而不是1)。

在第一步中，探针位置公式计算出probe = 5：

![](/assets/images/2025/algorithms/javainterpolationsearch03.png)

因为84(我们要查找的元素)大于73(当前探测位置元素)，所以下一步将通过分配lowEnd = probe + 1放弃数组的左侧。现在搜索空间仅包含84和101，探针位置公式将设置probe = 6，这正是84的索引：

![](/assets/images/2025/algorithms/javainterpolationsearch04.png)

由于找到了我们正在寻找的元素，因此将返回位置6。

## 4. Java实现

现在我们了解了该算法的工作原理，让我们用Java来实现它。

首先，我们初始化lowEnd和highEnd：
```java
int highEnd = (data.length - 1);
int lowEnd = 0;
```

接下来，我们设置一个循环，在每次迭代中，我们根据上述公式计算新的probe，循环条件通过将item与data[lowEnd\]和data[highEnd\]进行比较来确保我们不会超出搜索空间：
```java
while (item >= data[lowEnd] && item <= data[highEnd] && lowEnd <= highEnd) {
    int probe = lowEnd + (highEnd - lowEnd) * (item - data[lowEnd]) / (data[highEnd] - data[lowEnd]);
}
```

每次新的probe分配后，我们还会检查是否找到了该元素。

最后，我们调整lowEnd或highEnd以减少每次迭代的搜索空间：
```java
public int interpolationSearch(int[] data, int item) {

    int highEnd = (data.length - 1);
    int lowEnd = 0;

    while (item >= data[lowEnd] && item <= data[highEnd] && lowEnd <= highEnd) {
        int probe = lowEnd + (highEnd - lowEnd) * (item - data[lowEnd]) / (data[highEnd] - data[lowEnd]);

        if (highEnd == lowEnd) {
            if (data[lowEnd] == item) {
                return lowEnd;
            } else {
                return -1;
            }
        }

        if (data[probe] == item) {
            return probe;
        }

        if (data[probe] < item) {
            lowEnd = probe + 1;
        } else {
            highEnd = probe - 1;
        }
    }
    return -1;
}
```

## 5. 总结

在本文中，我们通过一个示例探讨了插值搜索，并用Java实现了它。