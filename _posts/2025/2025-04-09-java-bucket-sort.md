---
layout: post
title:  Java中的桶排序
category: algorithms
copyright: algorithms
excerpt: 排序
---

## 1. 简介

在本文中，我们将深入研究**[桶排序](https://en.wikipedia.org/wiki/Bucket_sort)算法**。我们将从一些简单的理论开始，然后研究Java实现并对我们的解决方案进行单元测试。最后，我们将研究桶排序的时间复杂度。

## 2. 桶排序理论

桶排序(有时也称为箱排序)是一种特定的排序算法，其工作原理是将我们要排序的元素分配到几个单独排序的桶中，通过这样做，我们可以减少元素之间的比较次数，并有助于缩短排序时间。

让我们快速看一下执行桶排序所需的步骤：

1. 设置最初为空的桶数组
2. 将元素分配到适当的桶中
3. 对每个桶进行排序
4. 将排序好的桶拼接在一起，重新创建完整列表

## 3. Java实现

虽然此算法并非特定于语言，但我们将用Java实现排序，让我们逐步浏览上述步骤并编写代码来对整数列表进行排序。

### 3.1 桶设置

首先，**我们需要确定一个哈希算法来决定将哪个元素放入哪个存储桶中**：
```java
private int hash(int i, int max, int numberOfBuckets) {
    return (int) ((double) i / max * (numberOfBuckets - 1));
}
```

定义了哈希方法后，我们现在可以**将桶的数量指定为输入列表大小的平方根**：
```java
final int numberOfBuckets = (int) Math.sqrt(initialList.size());
List<List<Integer>> buckets = new ArrayList<>(numberOfBuckets);
for(int i = 0; i < numberOfBuckets; i++) {
    buckets.add(new ArrayList<>());
}
```

最后，我们需要一个简短的方法来确定输入列表中的最大整数：
```java
private int findMax(List<Integer> input) {
    int m = Integer.MIN_VALUE;
    for (int i : input) {
        m = Math.max(i, m);
    }
    return m;
}
```

### 3.2 分布元素

现在我们已经定义了存储桶，我们可以**使用hash方法将输入列表的每个元素分配到其相关的存储桶中**：
```java
int max = findMax(initialList);

for (int i : initialList) {
    buckets.get(hash(i, max, numberOfBuckets)).add(i);
}
```

### 3.3 对单个桶进行排序

定义好存储桶并填充整数后，**让我们使用Comparator对它们进行排序**：
```java
Comparator<Integer> comparator = Comparator.naturalOrder();

for(List<Integer> bucket  : buckets){
    bucket.sort(comparator);
}
```

### 3.4 拼接桶

最后，我们需要将存储桶合并在一起以重新创建单个列表。由于存储桶已排序，因此我们只需循环遍历每个存储桶一次并将元素追加到主列表中：
```java
List<Integer> sortedArray = new LinkedList<>();

for(List<Integer> bucket : buckets) {
    sortedArray.addAll(bucket);
} 

return sortedArray;
```

## 4. 测试

完成实现后，让我们编写一个快速单元测试以确保它按预期工作：
```java
BucketSorter sorter = new IntegerBucketSorter();

List<Integer> unsorted = Arrays.asList(80,50,60,30,20,10,70,0,40,500,600,602,200,15);
List<Integer> expected = Arrays.asList(0,10,15,20,30,40,50,60,70,80,200,500,600,602);

List<Integer> sorted = sorter.sort(unsorted);

assertEquals(expected, sorted);
```

## 5. 时间复杂度

接下来，我们来快速看一下执行桶排序的时间复杂度。

### 5.1 最坏情况

**在最坏的情况下，我们会发现所有元素都位于同一个桶中，并且顺序相反**。当这种情况发生时，我们将桶排序简化为一种简单排序，其中每个元素都与其他每个元素进行比较，**时间复杂度为O(n²)**。

### 5.2 平均情况

**在平均情况下，我们发现元素在输入桶中分布相对均匀**，由于每个步骤只需要对输入桶进行一次迭代，因此**桶排序的复杂度为O(n)**。

## 6. 总结

在本文中，我们了解了如何在Java中实现桶排序，我们还研究了桶排序算法的时间复杂度。
