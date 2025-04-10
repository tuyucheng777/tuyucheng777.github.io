---
layout: post
title:  查找Java数组中的前K个元素
category: algorithms
copyright: algorithms
excerpt: 数组
---

## 1. 概述

在本教程中，我们将使用Java实现查找数组中最大k个元素的问题的不同解决方案。为了描述时间复杂度，我们将使用[大O表示法](https://www.baeldung.com/cs/big-o-notation)。

## 2. 暴力破解

这个问题的暴力解决方案是迭代给定的数组k次，**在每次迭代中，我们将找到最大值**，然后我们将从数组中删除此值并将其放入输出列表中：
```java
public List findTopK(List input, int k) {
    List array = new ArrayList<>(input);
    List topKList = new ArrayList<>();

    for (int i = 0; i < k; i++) {
        int maxIndex = 0;

        for (int j = 1; j < array.size(); j++) {
            if (array.get(j) > array.get(maxIndex)) {
                maxIndex = j;
            }
        }

        topKList.add(array.remove(maxIndex));
    }

    return topKList;
}
```

如果假设n是给定数组的大小，则**该解决方案的时间复杂度为O(n * k)**。此外，这是效率最低的解决方案。

## 3. Java集合方法

但是，这个问题存在更有效的解决方案。在本节中，我们将使用Java集合来解释其中两种解决方案。

### 3.1 TreeSet

TreeSet以[红黑树](https://www.baeldung.com/cs/red-black-trees)数据结构为核心，因此，将值放入此集合的复杂度为O(log n)。TreeSet是一个有序集合，因此，我们可以**将所有值放入TreeSet中并提取其中的前k个**：
```java
public List<Integer> findTopK(List<Integer> input, int k) {
    Set<Integer> sortedSet = new TreeSet<>(Comparator.reverseOrder());
    sortedSet.addAll(input);

    return sortedSet.stream().limit(k).collect(Collectors.toList());
}
```

**该解决方案的时间复杂度为O(n * log n)**，最重要的是，如果k ≥ log n，这应该比蛮力方法更有效。

需要注意的是，TreeSet不包含重复项。因此，该解决方案仅适用于具有不同值的输入数组。

### 3.2 PriorityQueue

PriorityQueue是Java中的堆数据结构，借助它，**我们可以实现O(n * log k)解决方案**。而且，这将比前一个解决方案更快。由于所述问题，k始终小于数组的大小。因此，这意味着O(n * log k) ≤ O(n * log n)。

该算法对给定的数组进行一次迭代，每次迭代时，我们都会向堆中添加一个新元素。同时，我们会保持堆的大小小于或等于k。因此，我们必须从堆中删除多余的元素并添加新元素。因此，在迭代数组后，堆将包含k个最大值：
```java
public List<Integer> findTopK(List<Integer> input, int k) {
    PriorityQueue<Integer> maxHeap = new PriorityQueue<>();

    input.forEach(number -> {
        maxHeap.add(number);

        if (maxHeap.size() > k) {
            maxHeap.poll();
        }
    });

    List<Integer> topKList = new ArrayList<>(maxHeap);
    Collections.reverse(topKList);

    return topKList;
}
```

## 4. 选择算法

有很多方法可以解决给定的问题，尽管这超出了本教程的范围，但使用[选择算法](https://en.wikipedia.org/wiki/Selection_algorithm)方法将是最佳选择，因为它具有线性时间复杂度。

## 5. 总结

在本教程中，我们描述了几种查找数组中最大k个元素的解决方案。