---
layout: post
title:  在Java中查找数组中整数的众数
category: algorithms
copyright: algorithms
excerpt: 众数
---

## 1. 概述

在本文中，我们将探讨如何使用Java查找[数组](https://www.baeldung.com/java-arrays-guide)中整数的众数。

在Java中处理数据集时，我们经常需要查找统计指标，例如平均值、中位数和众数。**众数是数据集中出现频率最高的值，如果没有重复的数字，则数据集没有众数。如果多个数字具有相同的最高频率，则它们都被视为众数**。

## 2. 理解问题

该算法旨在找出数组中整数的众数，让我们考虑一些例子：

nums = {1,2,2,3,3,4,4,4,5}，此数组的众数为4。

nums = {1,2,2,1}，此数组的众数为{1,2}。

对于我们的代码，我们有一个整数数组示例：
```java
int[] nums = { 1, 2, 2, 3, 3, 4, 4, 4, 5 };
```

## 3. 使用排序

找到众数的一种方法是[排序](https://www.baeldung.com/java-sorting)数组并找出出现次数最多的元素，**这种方法利用了排序数组中重复元素相邻的特性**。我们来看代码：
```java
Arrays.sort(nums);
int maxCount = 1;
int currentCount = 1;
Set<Integer> modes = new HashSet<>();

for (int i = 1; i < nums.length; i++) {
    if (nums[i] == nums[i - 1]) {
        currentCount++;
    }
    else {
        currentCount = 1;
    }

    if (currentCount > maxCount) {
        maxCount = currentCount;
        modes.clear();
        modes.add(nums[i]);
    }
    else if (currentCount == maxCount) {
        modes.add(nums[i]);
    }
}

if (nums.length == 1) {
    modes.add(nums[0]);
}
```

此方法对输入数组进行排序，然后遍历数组以计算每个数字的频率。它会跟踪频率最高的数字并相应地更新众数列表，它还会处理数组仅包含一个元素的极端情况。

让我们看看时间和空间复杂度：

- 时间复杂度：由于排序步骤，因此为O(n log n)。
- 空间复杂度：如果使用的排序算法是归并排序，则最坏情况下为O(n)；如果我们只考虑用于存储众数的额外空间，则最坏情况下为O(k)。

这里，n是数组中元素的数量，k是众数的数量。

## 4. 使用频率矩阵

**如果数组中的整数范围已知且有限，则频率数组可能是一种非常有效的解决方案**。此方法使用数组索引来计数出现次数，让我们看看如何实现：
```java
Map<Integer, Integer> frequencyMap = new HashMap<>();

for (int num : nums) {
    frequencyMap.put(num, frequencyMap.getOrDefault(num, 0) + 1);
}

int maxFrequency = 0;
for (int frequency : frequencyMap.values()) {
    if (frequency > maxFrequency) {
        maxFrequency = frequency;
    }
}

Set<Integer> modes = new HashSet<>();
for (Map.Entry<Integer, Integer> entry : frequencyMap.entrySet()) {
    if (entry.getValue() == maxFrequency) {
        modes.add(entry.getKey());
    }
}
```

该方法将数组中每个整数的频率填充到一个Map中，然后确定Map中存在的最高频率。最后，它从Map中收集所有频率最高的整数。

让我们看看时间和空间复杂度：

- 时间复杂度：O(n + m)，在平均情况下简化为O(n)，因为m通常远小于n。
- 空间复杂度：O(m + k)，最坏情况下，如果所有元素都是唯一的，且每个元素都是一个众数，则复杂度可能是O(n)。

这里，n是数组中元素的数量，m是数组中唯一元素的数量，k是众数的数量。

## 5. 使用TreeMap

**[TreeMap](https://www.baeldung.com/java-treemap)可以提供排序的频率Map，这在某些情况下可能很有用**，其逻辑如下：
```java
Map<Integer, Integer> frequencyMap = new TreeMap<>();

for (int num : nums) {
    frequencyMap.put(num, frequencyMap.getOrDefault(num, 0) + 1);
}

int maxFrequency = 0;
for (int frequency : frequencyMap.values()) {
    if (frequency > maxFrequency) {
        maxFrequency = frequency;
    }
}

Set<Integer> modes = new HashSet<>();
for (Map.Entry<Integer, Integer> entry : frequencyMap.entrySet()) {
    if (entry.getValue() == maxFrequency) {
        modes.add(entry.getKey());
    }
}
```

使用的方法与上一节相同，唯一的区别是我们在这里使用了TreeMap。使用TreeMap可确保元素按排序顺序存储，这对于需要排序键的进一步操作非常有用。

让我们看看时间和空间复杂度：

- 时间复杂度：O(n log m + m)，在平均情况下简化为O(n log m)。
- 空间复杂度：O(m + k)，最坏情况下，如果所有元素都是唯一的，且每个元素都是众数，则复杂度可能为O(n)。

这里，n是数组中元素的数量，m是数组中唯一元素的数量，k是众数的数量。

## 6. 使用Stream

**当处理大型数据集时，我们可以利用Java的[并行流](https://www.baeldung.com/java-streams)来利用多核处理器**，具体逻辑如下：
```java
Map<Integer, Long> frequencyMap = Arrays.stream(nums)
    .boxed()
    .collect(Collectors.groupingBy(e -> e, Collectors.counting()));

long maxFrequency = Collections.max(frequencyMap.values());

Set<Integer> modes = frequencyMap.entrySet()
    .stream()
    .filter(entry -> entry.getValue() == maxFrequency)
    .map(Map.Entry::getKey)
    .collect(Collectors.toSet());
```

该代码使用Java Stream以函数式风格处理数组，这使得代码简洁且富有表现力。

首先，我们将[原始](https://www.baeldung.com/java-primitives#:~:text=2.-,PrimitiveDataTypes,objectsandrepresentrawvalues.)整数转换为Integer对象，以便它能够与通用流操作兼容，然后我们根据整数的值对其进行分组，并使用Collectors.groupingBy()和Collectors.counting()计算它们的出现次数，使用Collections.max()找到最大频率。最后，筛选出频率最大的条目，并将其键收集到列表中。

该方法非常高效，并且利用Java Stream API的强大功能以干净、易读的方式查找众数。

让我们看看时间和空间复杂度：

- 时间复杂度：O(n + m)，在平均情况下简化为O(n)，因为m通常远小于n。
- 空间复杂度：O(m + k)，最坏情况下，如果所有元素都是唯一的，且每个元素都是众数，则复杂度可能是O(n)。

这里，n是数组中元素的数量，m是数组中唯一元素的数量，k是众数的数量。

## 7. 总结

在本教程中，我们探索了查找数组中整数众数的各种方法，每种方法都有其优点，适用于不同的场景。以下是一些简要的总结，可以帮助我们选择正确的方法：

- 排序：对于中小型数组来说简单有效
- 频率阵列：如果数字范围较小，则效率很高
- TreeMap：如果我们需要排序频率Map，则很有用
- 并行流：非常适合利用多个核心的大型数据集

通过根据我们的具体要求选择适当的方法，我们可以优化在Java中查找数组中整数的众数的过程。