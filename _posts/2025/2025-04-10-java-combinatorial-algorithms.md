---
layout: post
title:  Java组合问题概述
category: algorithms
copyright: algorithms
excerpt: 组合
---

## 1. 概述

在本教程中，我们将学习如何解决一些常见的组合问题。这些问题在日常工作中可能用处不大，但从算法的角度来看，它们很有意思，**我们可能会发现它们在测试中很有用**。

请记住，解决这些问题的方法有很多种，我们尽力让所介绍的解决方案易于理解。

## 2. 生成排列

首先，我们先从排列说起。排列是指重新排列一个序列，使其具有不同的顺序。

我们从数学中知道，**对于n个元素的序列，有n!种不同的排列**，n!被称为[阶乘](https://www.baeldung.com/java-calculate-factorial)运算：

>n！= 1 * 2 * ... * n

例如，对于序列[1,2,3\]有6种排列：
```text
[1, 2, 3]

[1, 3, 2]

[2, 1, 3]

[2, 3, 1]

[3, 1, 2]

[3, 2, 1]
```

阶乘增长非常快-对于10个元素的序列，我们有3628800种不同的排列，在这种情况下，**我们讨论的是排列序列，其中每个元素都是不同的**。

### 2.1 算法

考虑以递归方式生成排列是一个好主意，**让我们介绍一下状态的概念，它将由两部分组成：当前排列和当前处理元素的索引**。

在这种状态下唯一要做的工作就是将元素与每个剩余元素交换，并执行到修改序列且索引增加1的状态的转换。

我们举一个例子来说明一下。

我们希望生成一个由4个元素组成的序列[1,2,3,4\]的所有排列，因此，将有24种排列。下图展示了该算法的部分步骤：

![](/assets/images/2025/algorithms/javacombinatorialalgorithms01.png)

**树的每个节点都可以理解为一种状态**，顶部的红色数字表示当前处理的元素的索引，节点中的绿色数字表示交换。

因此，我们从[1,2,3,4\]状态开始，索引等于0。我们将第一个元素与每个元素交换(包括第一个元素，它不交换任何内容)，然后进入下一个状态。

现在，我们想要的排列位于右边的最后一列。

### 2.2 Java实现

用Java编写的算法很简短：
```java
private static void permutationsInternal(List<Integer> sequence, List<List<Integer>> results, int index) {
    if (index == sequence.size() - 1) {
        permutations.add(new ArrayList<>(sequence));
    }

    for (int i = index; i < sequence.size(); i++) {
        swap(sequence, i, index);
        permutationsInternal(sequence, permutations, index + 1);
        swap(sequence, i, index);
    }
}
```

**我们的函数接收3个参数：当前处理的序列、结果(排列)和当前正在处理的元素的索引**。

首先要检查是否已到达最后一个元素，如果是，就将序列添加到结果列表中。

然后，在for循环中，我们执行交换，对方法进行递归调用，然后将元素交换回来。

最后一部分是一个小小的性能技巧-我们可以一直对同一个序列对象进行操作，而不必为每次递归调用创建一个新的序列。

将第一个递归调用隐藏在门面方法下也可能是一个好主意：
```java
public static List<List<Integer>> generatePermutations(List<Integer> sequence) {
    List<List<Integer>> permutations = new ArrayList<>();
    permutationsInternal(sequence, permutations, 0);
    return permutations;
}
```

**请记住，所示算法仅适用于唯一元素序列**，对具有重复元素的序列应用相同算法将产生重复。

## 3. 生成集合的幂集

另一个常见的问题是生成集合的幂集，我们先来看一下定义：

> 集合S的幂集是S的所有子集的集合，包括空集和S本身

例如，给定一个集合[a,b,c\]，其幂集包含8个子集：
```text
[]

[a]

[b]

[c]

[a, b]

[a, c]

[b, c]

[a, b, c]
```

我们从数学中知道，一个包含n个元素的集合，**其幂集应该包含2^n个子集**。这个数字增长很快，但是没有阶乘那么快。

### 3.1 算法

这次，我们也将采用递归的方式思考。现在，我们的状态将由两部分组成：集合中当前正在处理的元素的索引和累加器。

**我们需要在每个状态下做出两个选择的决定：是否将当前元素放入累加器**。当索引到达集合末尾时，我们就得到了一个可能的子集。这样，我们就可以生成所有可能的子集。

### 3.2 Java实现

用Java编写的算法非常易读：
```java
private static void powersetInternal(List<Character> set, List<List<Character>> powerset, List<Character> accumulator, int index) {
    if (index == set.size()) {
        results.add(new ArrayList<>(accumulator));
    } else {
        accumulator.add(set.get(index));
        powerSetInternal(set, powerset, accumulator, index + 1);
        accumulator.remove(accumulator.size() - 1);
        powerSetInternal(set, powerset, accumulator, index + 1);
    }
}
```

我们的函数有4个参数：我们想要生成子集的集合、得到的幂集、累加器和当前处理元素的索引。

为简单起见，**我们将集合保存在列表中；我们希望能够快速访问索引指定的元素**，这可以通过List实现，但不能通过Set实现。

此外，单个元素由单个字母(Java中的Character类)表示。

**首先，我们检查索引是否超出了集合大小**，如果是，则将累加器放入结果集；否则我们：

- 将当前考虑的元素放入累加器
- 使用递增索引和扩展累加器进行递归调用
- 从累加器中删除我们之前添加的最后一个元素
- 使用不变的累加器和增加的索引再次进行调用

再次，我们用门面方法隐藏实现：
```java
public static List<List<Character>> generatePowerset(List<Character> sequence) {
    List<List<Character>> powerset = new ArrayList<>();
    powerSetInternal(sequence, powerset, new ArrayList<>(), 0);
    return powerset;
}
```

## 4. 生成组合

现在，是时候解决组合问题了，我们定义如下：

> 集合S的k个组合是S中k个不同元素的子集，其中元素的顺序无关紧要

[k种组合](https://www.baeldung.com/cs/generate-k-combinations)的数量由二项式系数描述：

![](/assets/images/2025/algorithms/javacombinatorialalgorithms02.png)

例如，对于集合[a,b,c\]，我们有三个2组合：
```text
[a, b]

[a, c]

[b, c]
```

组合有很多组合用法和解释，例如，假设我们有一个由16支球队组成的足球联赛，我们可以看到多少场不同的比赛？

答案是![](/assets/images/2025/algorithms/javacombinatorialalgorithms03.png)，计算结果为120。

### 4.1 算法

从概念上讲，我们将做一些类似于前面幂集算法的事情。我们将有一个递归函数，其状态由当前处理元素的索引和累加器组成。

**同样，我们对每个状态都做出了相同的决定：是否将元素添加到累加器？不过，这一次我们有一个额外的限制-我们的累加器不能超过k个元素**。

值得注意的是，二项式系数不一定非要很大，例如：

![](/assets/images/2025/algorithms/javacombinatorialalgorithms04.png)等于4950，而

![](/assets/images/2025/algorithms/javacombinatorialalgorithms05.png)有30位数字。

### 4.2 Java实现

为了简单起见，我们假设集合中的元素是整数。

我们来看看该算法的Java实现：
```java
private static void combinationsInternal(List<Integer> inputSet, int k, List<List<Integer>> results, ArrayList<Integer> accumulator, int index) {
    int needToAccumulate = k - accumulator.size();
    int canAccumulate = inputSet.size() - index;

    if (accumulator.size() == k) {
        results.add(new ArrayList<>(accumulator));
    } else if (needToAccumulate <= canAccumulate) {
        combinationsInternal(inputSet, k, results, accumulator, index + 1);
        accumulator.add(inputSet.get(index));
        combinationsInternal(inputSet, k, results, accumulator, index + 1);
        accumulator.remove(accumulator.size() - 1);
    }
}
```

这次，我们的函数有5个参数：一个输入集、k参数、一个结果列表、一个累加器和当前处理元素的索引。

我们首先定义辅助变量：

- needToAccumulate：表示我们需要向累加器添加多少元素才能获得正确的组合
- canAccumulate：表示我们可以向累加器添加多少个元素

现在，我们检查累加器大小是否等于k；如果是，那么我们可以将复制的数组放入结果列表中。

在另一种情况下，**如果集合的剩余部分仍有足够的元素，我们会进行两次单独的递归调用：将当前处理的元素放入累加器，以及不将当前处理的元素放入累加器**，这部分类似于我们之前生成幂集的方式。

当然，这个方法可以写得更快一些。例如，我们可以稍后声明needToAccumulate和canAccumulate变量。但是，我们更注重可读性。

再次，门面方法可以隐藏实现：
```java
public static List<List<Integer>> combinations(List<Integer> inputSet, int k) {
    List<List<Integer>> results = new ArrayList<>();
    combinationsInternal(inputSet, k, results, new ArrayList<>(), 0);
    return results;
}
```

## 5. 总结

在本文中，我们讨论了不同的组合问题。此外，我们还展示了使用Java实现来解决这些问题的简单算法，在某些情况下，这些算法可以帮助满足一些特殊的测试需求。