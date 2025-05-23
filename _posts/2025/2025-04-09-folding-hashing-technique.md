---
layout: post
title:  Java折叠技术指南
category: algorithms
copyright: algorithms
excerpt: 折叠技术
---

## 1. 简介

在本教程中，我们考虑在各种数据结构中使用的哈希技术，这些技术可提供对其元素的恒定时间访问。

我们更详细地讨论了所谓的折叠技术，并简要介绍了中间平方和分箱技术。

## 2. 概述

当我们选择用于存储对象的数据结构时，考虑因素之一是我们是否需要快速访问它们。

Java实用程序包提供了许多用于存储对象的数据结构，有关这些数据结构的更多信息，请参阅我们的[Java集合](https://www.baeldung.com/java-collections)系列，其中包含一些关于数据结构的指南。

我们知道，**其中一些数据结构允许我们在恒定时间内检索其元素**，而与它们包含的元素数量无关。

最简单的可能就是数组了；实际上，我们通过索引来访问数组中的元素，访问时间自然与数组的大小无关。事实上，在幕后，许多数据结构都大量使用数组。

问题在于数组索引必须是数字，而我们通常更喜欢使用对象来操作这些数据结构。

为了解决这个问题，许多数据结构尝试为对象分配一个可以作为数组索引的数值，**我们称这个值为哈希值或简称为哈希**。

## 3. 哈希

**[哈希](https://www.baeldung.com/cs/hashing)是将对象转换为数值的过程**，执行这些转换的函数称为哈希函数。

为了简单起见，我们考虑将字符串转换为数组索引的哈希函数，即将字符串转换为范围在[0,N\]之间的整数，其中N是有限的。

当然，**哈希函数适用于各种各样的字符串**，因此它的“全局”属性变得很重要。

![](/assets/images/2025/algorithms/foldinghashingtechnique01.png)

**不幸的是，哈希函数不可能总是能将不同的字符串转换成不同的数字**。

毕竟，字符串的数量比[0,N\]范围内的整数数量要大得多。因此，不可避免地会有一对不相等的字符串，而哈希函数会为其生成相等的值，**这种现象称为碰撞**。

我们不会深入研究哈希函数背后的工程细节，但很明显，一个好的哈希函数应该尝试将其定义的字符串统一地映射到数字。

另一个明显的要求是好的哈希函数必须足够快，如果计算哈希值的时间太长，那么就无法快速访问元素。

在本教程中，我们考虑一种尝试使映射统一同时保持快速的技术。

## 4. 折叠技术

我们的目标是找到一个将字符串转换为数组索引的函数，为了说明这个想法，假设我们希望这个数组具有10<sup>5</sup>个元素的容量，并以字符串“Java language”为例。

### 4.1 描述

我们先把字符串的字符转换成数字，ASCII是进行此操作的一个很好的选择：

![](/assets/images/2025/algorithms/foldinghashingtechnique02.png)

现在，我们将刚刚得到的数字按一定大小分组，通常，我们根据数组的大小(即10<sup>5</sup>)来选择组大小值。由于我们将字符转换为的数字包含2到3位数字，因此，在不失一般性的情况下，我们可以将组大小设置为2：

![](/assets/images/2025/algorithms/foldinghashingtechnique03.png)

下一步是将每组中的数字像字符串一样拼接起来，然后求出它们的总和：

![](/assets/images/2025/algorithms/foldinghashingtechnique04.png)

现在我们必须进行最后一步，检查数字348933是否可以作为大小为10<sup>5</sup>的数组的索引，当然，它超过了允许的最大值99999。我们可以通过应用模运算符来找到最终结果，从而轻松克服这个问题：
```text
348933 % 10000 = 48933
```

### 4.2 结语

我们看到该算法不包含任何耗时操作，因此速度非常快。**输入字符串的每个字符都会影响最终结果**，这确实有助于减少碰撞，但并不能完全避免碰撞。

例如，如果我们想跳过折叠并将模运算符直接应用于ASCII转换的输入字符串(忽略溢出问题)
```text
749711897321089711010311797103101 % 100000 = 3101
```

那么该哈希函数将为所有具有与输入字符串相同的最后两个字符的字符串产生相同的值：age、page、large，等等。

从算法描述中，我们很容易看出它并非没有碰撞。例如，该算法对“Java language”和“vaJa language”字符串产生相同的哈希值。

## 5. 其他技术

折叠技术很常见，但不是唯一的技术。有时，分箱或中间平方技术也可能有用。

我们用数字而不是字符串来解释他们的想法(假设我们已经以某种方式将字符串转换为数字)，我们不会讨论它们的优点和缺点，但你在了解算法之后应该可以形成自己的观点。

### 5.1 分箱技术

假设我们有100个整数，我们希望哈希函数将它们映射到一个包含10个元素的数组中，那么我们可以将这100个整数分成10组，前10个整数放在第一个容器中，后10个整数放在第二个容器中，依此类推：

![](/assets/images/2025/algorithms/foldinghashingtechnique05.png)

### 5.2 中间平方技术

该算法由约翰·冯·诺依曼提出，它允许我们从给定数字开始生成伪随机数。

![](/assets/images/2025/algorithms/foldinghashingtechnique06.png)

让我们用一个具体的例子来说明：假设我们有一个四位数1111，根据算法，我们对其求平方，得到1234321。现在，我们从中间取出4位数字，例如2343。算法允许我们重复这个过程，直到对结果满意为止。

## 6. 总结

在本教程中，我们探讨了几种哈希技术，并详细描述了折叠技术，还简要描述了如何实现分箱和中间平方。