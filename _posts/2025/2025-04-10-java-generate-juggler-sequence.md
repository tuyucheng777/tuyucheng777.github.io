---
layout: post
title:  在Java中生成Juggler序列
category: algorithms
copyright: algorithms
excerpt: Juggler
---

## 1. 概述

[杂耍者序列](https://en.wikipedia.org/wiki/Juggler_sequence)以其有趣的行为和优雅的简洁性而引人注目。

在本教程中，我们将了解杂耍序列并探索如何使用Java中给定的初始数字生成该序列。

## 2. 理解杂耍者序列

在我们深入研究生成杂耍序列的代码之前，让我们快速了解一下杂耍序列是什么。

在数论中，杂耍序列是一个整数序列，其递归定义如下：

- 以正整数n作为序列的第一项
- 如果n为偶数，则下一个项为n<sup>1/2</sup>，向下舍入到最接近的整数
- 如果n为奇数，则下一个项为n<sup>3/2</sup>，向下舍入到最接近的整数

**该过程持续直至达到1，序列终止**。 

值得一提的是，**n<sup>1/2</sup>和n<sup>3/2</sup>都可以转化为平方根计算**：

- n<sup>1/2</sup>是n的平方根，因此n<sup>1/2</sup> = sqrt(n)
- n<sup>3/2</sup>= n<sup>1</sup> * n<sup>1/2</sup> = n * sqrt(n)

一个例子可以帮助我们快速理解这个序列：
```text
Given number: 3
-----------------
 3 -> odd  ->  3 * sqrt(3) -> (int)5.19.. -> 5
 5 -> odd  ->  5 * sqrt(5) -> (int)11.18.. -> 11
11 -> odd  -> 11 * sqrt(11)-> (int)36.48.. -> 36
36 -> even -> sqrt(36) -> (int)6 -> 6
 6 -> even -> sqrt(6) -> (int)2.45.. -> 2
 2 -> even -> sqrt(2) -> (int)1.41.. -> 1
 1

sequence: 3, 5, 11, 36, 6, 2, 1
```

值得注意的是，据推测所有杂耍者序列最终都会达到1，但这一猜想尚未得到证实。因此，我们实际上无法完成[大O](https://www.baeldung.com/cs/big-oh-asymptotic-complexity)时间复杂度分析。

现在我们知道了杂耍者序列是如何生成的，让我们用Java实现一些序列生成方法。

## 3. 基于循环的解决方案

我们先实现一个基于循环的生成方法：
```java
class JugglerSequenceGenerator {

    public static List<Integer> byLoop(int n) {
        if (n <= 0) {
            throw new IllegalArgumentException("The initial integer must be greater than zero.");
        }
        List<Integer> seq = new ArrayList<>();
        int current = n;
        seq.add(current);
        while (current != 1) {
            int next = (int) (Math.sqrt(current) * (current % 2 == 0 ? 1 : current));
            seq.add(next);
            current = next;
        }
        return seq;
    }
}
```

代码看起来非常简单，让我们快速浏览一下代码并了解它的工作原理：

- 首先，验证输入n，因为初始数字必须是正整数
- 然后，创建seq列表来存储结果序列，将初始整数分配给current，并将其添加到seq
- while循环负责根据我们之前讨论的计算生成每个术语并将其附加到序列中
- 一旦循环终止(当current变为1时)，就会返回存储在seq列表中的生成序列

接下来，让我们创建一个测试方法来验证我们的基于循环的方法是否可以产生预期的结果：
```java
assertThrows(IllegalArgumentException.class, () -> JugglerSequenceGenerator.byLoop(0));
assertEquals(List.of(3, 5, 11, 36, 6, 2, 1), JugglerSequenceGenerator.byLoop(3));
assertEquals(List.of(4, 2, 1), JugglerSequenceGenerator.byLoop(4));
assertEquals(List.of(9, 27, 140, 11, 36, 6, 2, 1), JugglerSequenceGenerator.byLoop(9));
assertEquals(List.of(21, 96, 9, 27, 140, 11, 36, 6, 2, 1), JugglerSequenceGenerator.byLoop(21));
assertEquals(List.of(42, 6, 2, 1), JugglerSequenceGenerator.byLoop(42));
```

## 4. 基于递归的解决方案

或者，我们可以[递归](https://www.baeldung.com/java-recursion)地从给定的数字生成一个杂耍序列，首先，让我们将byRecursion()方法添加到JugglerSequenceGenerator类：
```java
public static List<Integer> byRecursion(int n) {
    if (n <= 0) {
        throw new IllegalArgumentException("The initial integer must be greater than zero.");
    }
    List<Integer> seq = new ArrayList<>();
    fillSeqRecursively(n, seq);
    return seq;
}
```

我们可以看到，byRecursion()方法是另一个杂耍者序列生成器的入口点，它验证输入的数字并准备结果序列列表。但是，主要的序列生成逻辑是在fillSeqRecursively()方法中实现的：
```java
private static void fillSeqRecursively(int current, List<Integer> result) {
    result.add(current);
    if (current == 1) {
        return;
    }
    int next = (int) (Math.sqrt(current) * (current % 2 == 0 ? 1 : current));
    fillSeqRecursively(next, result);
}
```

如代码所示，**该方法使用next和result列表递归调用自身**，这意味着该方法将重复将current数字添加到序列、检查终止条件和计算下一个项的过程，直到满足终止条件(current == 1)。

递归方法通过了相同的测试：
```java
assertThrows(IllegalArgumentException.class, () -> JugglerSequenceGenerator.byRecursion(0));
assertEquals(List.of(3, 5, 11, 36, 6, 2, 1), JugglerSequenceGenerator.byRecursion(3));
assertEquals(List.of(4, 2, 1), JugglerSequenceGenerator.byRecursion(4));
assertEquals(List.of(9, 27, 140, 11, 36, 6, 2, 1), JugglerSequenceGenerator.byRecursion(9));
assertEquals(List.of(21, 96, 9, 27, 140, 11, 36, 6, 2, 1), JugglerSequenceGenerator.byRecursion(21));
assertEquals(List.of(42, 6, 2, 1), JugglerSequenceGenerator.byRecursion(42));
```

## 5. 总结

在本文中，我们首先讨论了什么是杂耍序列，需要注意的是，尚未证明所有杂耍序列最终都会达到1。

此外，我们探索了两种从给定整数开始生成杂耍序列的方法。