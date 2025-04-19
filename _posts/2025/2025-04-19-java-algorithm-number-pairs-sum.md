---
layout: post
title:  在Java中查找数组中所有相加等于给定和的数字对
category: algorithms
copyright: algorithms
excerpt: 数组
---

## 1. 概述

在本快速教程中，我们将展示如何实现一个算法，用于查找数组中所有数字对，其和等于给定数字；我们将重点介绍两种解决这个问题的方法。

在第一种方法中，我们会找到所有这样的对，无论它们是否唯一。在第二种方法中，我们只找到唯一的数字组合，并删除冗余的对。

对于每种方法，我们将提供两种实现- 一种是使用for循环的传统实现，另一种是使用Java 8 Stream API的实现。

## 2. 返回所有匹配的对

我们将遍历一个整数数组，使用嵌套循环的暴力方法，找出所有和等于给定数字(sum)的数对(i和j)，该算法的运行时复杂度为O(n²)。

为了演示，我们将使用以下输入数组查找所有总和等于6的数字对：

```java
int[] input = { 2, 4, 3, 3 };
```

在这种方法中，我们的算法应该返回：

```text
{2,4}, {4,2}, {3,3}, {3,3}
```

在每种算法中，当我们找到一对总和等于目标数字的目标数字时，我们将使用实用方法addPairs(i,j)来收集该对。

我们可能想到的实现解决方案的第一种方法是使用传统的for循环：

```java
for (int i = 0; i < input.length; i++) {
    for (int j = 0; j < input.length; j++) {
        if (j != i && (input[i] + input[j]) == sum) {
            addPairs(input[i], sum-input[i]));
        }
    }
}
```

这可能有点简陋，所以我们也使用Java 8 Stream API编写一个实现。

这里，我们使用IntStream.range方法生成一个连续的数字流。然后，我们根据条件对它们进行过滤：数字1 + 数字2 = 总和：

```java
IntStream.range(0,  input.length)
    .forEach(i -> IntStream.range(0,  input.length)
        .filter(j -> i != j && input[i] + input[j] == sum)
        .forEach(j -> addPairs(input[i], input[j]))
);
```

## 3. 返回所有唯一匹配对

对于这个例子，我们必须开发一个更智能的算法，只返回唯一的数字组合，省略冗余对。

为了实现这一点，我们将每个元素添加到一个HashMap中(不进行排序)，首先检查该对是否已显示。如果没有，我们将检索它并将其标记为已显示(将value字段设置为null)。

因此，使用与之前相同的输入数组，并且目标总和为6，我们的算法应该只返回不同的数字组合：

```text
{2,4}, {3,3}
```

如果我们使用传统的for循环，我们将得到：

```java
Map<Integer, Integer> pairs = new HashMap();
for (int i : input) {
    if (pairs.containsKey(i)) {
        if (pairs.get(i) != null) {            
            addPairs(i, sum-i);
        }                
        pairs.put(sum - i, null);
    } else if (!pairs.containsValue(i)) {        
        pairs.put(sum-i, i);
    }
}
```

请注意，此实现改进了以前的复杂性，因为我们只使用一个for循环，所以时间复杂度为O(n)。

现在让我们使用Java 8和Stream API来解决这个问题：

```java
Map<Integer, Integer> pairs = new HashMap();
IntStream.range(0, input.length).forEach(i -> {
    if (pairs.containsKey(input[i])) {
        if (pairs.get(input[i]) != null) {
            addPairs(input[i], sum - input[i]);
        }
        pairs.put(sum - input[i], null);
    } else if (!pairs.containsValue(input[i])) {
        pairs.put(sum - input[i], input[i]);
    }
});
```

## 4. 总结

在本文中，我们解释了几种在Java中查找所有和等于给定数字的对的不同方法。我们介绍了两种不同的解决方案，每种方案都使用了两种Java核心方法。