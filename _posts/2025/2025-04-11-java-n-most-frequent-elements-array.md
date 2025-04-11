---
layout: post
title:  查找Java数组中出现频率最高的N个元素
category: algorithms
copyright: algorithms
excerpt: 数组
---

## 1. 概述

在这个简短的教程中，我们将了解如何在Java数组中找到第n个最常见的元素。

## 2. 使用HashMap和PriorityQueue

我们可以使用HashMap来计算每个元素的出现次数，并使用[PriorityQueue](https://www.baeldung.com/java-priorityqueue)根据元素的出现次数对元素进行优先排序。

这使我们能够有效地找到数组中出现频率最高的n个元素：
```java
public static List<Integer> findByHashMapAndPriorityQueue(Integer[] array, int n) {
    Map<Integer, Integer> countMap = new HashMap<>();

    // For each element i in the array, add it to the countMap and increment its count
    for (Integer i : array) {
        countMap.put(i, countMap.getOrDefault(i, 0) + 1);
    }

    // Create a max heap (priority queue) that will prioritize elements with higher counts.
    PriorityQueue<Integer> heap = new PriorityQueue<>((a, b) -> countMap.get(b) - countMap.get(a));

    // Add all the unique elements in the array to the heap.
    heap.addAll(countMap.keySet());

    List<Integer> result = new ArrayList<>();
    for (int i = 0; i < n && !heap.isEmpty(); i++) {
        // Poll the highest-count element from the heap and add it to the result list.
        result.add(heap.poll());
    }

    return result;
}
```

让我们测试一下我们的方法：
```java
Integer[] inputArray = {1, 2, 3, 2, 2, 1, 4, 5, 6, 1, 2, 3};
Integer[] outputArray = {2, 1, 3};
assertThat(findByHashMapAndPriorityQueue(inputArray, 3)).containsExactly(outputArray);
```

## 3. 使用Stream API

我们还可以使用[Stream API](https://www.baeldung.com/java-streams)创建一个Map来计算每个元素的出现次数，并按频率降序对Map中的条目进行排序，然后提取前n个元素：
```java
public static List<Integer> findByStream(Integer[] arr, int n) {
    return Arrays.stream(arr).collect(Collectors.groupingBy(i -> i, Collectors.counting()))
        .entrySet().stream()
        .sorted(Collections.reverseOrder(Map.Entry.comparingByValue()))
        .map(Map.Entry::getKey)
        .limit(n)
        .collect(Collectors.toList());
}
```

## 4. 使用TreeMap

或者，我们可以使用TreeMap并创建一个自定义[Comparator](https://www.baeldung.com/java-comparator-comparable)，按频率降序比较Map中Map.Entry对象中的值：
```java
public static List<Integer> findByTreeMap(Integer[] arr, int n) {
    // Create a TreeMap and use a reverse order comparator to sort the entries by frequency in descending order
    Map<Integer, Integer> countMap = new TreeMap<>(Collections.reverseOrder());

    for (int i : arr) {
        countMap.put(i, countMap.getOrDefault(i, 0) + 1);
    }

    // Create a list of the map entries and sort them by value (i.e. by frequency) in descending order
    List<Map.Entry<Integer, Integer>> sortedEntries = new ArrayList<>(countMap.entrySet());
    sortedEntries.sort((e1, e2) -> e2.getValue().compareTo(e1.getValue()));

    // Extract the n most frequent elements from the sorted list of entries
    List<Integer> result = new ArrayList<>();
    for (int i = 0; i < n && i < sortedEntries.size(); i++) {
        result.add(sortedEntries.get(i).getKey());
    }

    return result;
}
```

## 5. 总结

总而言之，我们学习了在Java数组中查找第n个最常见元素的不同方法。