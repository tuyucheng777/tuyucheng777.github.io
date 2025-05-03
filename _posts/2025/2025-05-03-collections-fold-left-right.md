---
layout: post
title:  何时使用左折叠和右折叠
category: programming
copyright: programming
excerpt: 编程
---

## 1. 简介

在本教程中，我们将研究集合的“左折叠”和“右折叠”操作，我们将探讨它们之间的区别以及分别应该使用的情况。

## 2. 什么是折叠？

**在函数式编程中，折叠是一种标准操作，可用于将集合折叠为单个结果**。它的工作原理是遍历集合并应用相同的累加器函数，将当前累积的结果与集合中的下一个结果组合起来。

一个常见的例子是对一组数字求和，在这种情况下，我们要对集合中的数字应用的累加函数是plus，期望的结果是：

```text
result <- 1 + 2 + 3;
```

我们可以将其描述为使用plus函数折叠集合：

```text
plus <- (acc, next) => acc + next;
result <- [1, 2, 3].fold(plus);
```

这相当于：

```text
result <- plus(plus(1, 2), 3);
```

我们可以清楚地看到结果与我们期望的结果相同。

## 3. 折叠 vs 归约

值得注意的是，**我们有时会看到用归约来代替折叠**。根据上下文，这可能只是同一操作的另一种说法。但是，在其他情况下，它可能表示具有给定初始值的折叠操作。

让我们看一个例子：

```text
result <- [1, 2, 3].reduce(plus, 4);
```

这与以下内容相同：

```text
result <- plus(plus(plus(4, 1), 2), 3);
```

**这里最大的优势在于，我们不再需要让累加器函数的输出与集合条目的类型相同**。在我们之前的折叠示例中，累加器函数需要有两个参数和一个返回值，它们都是同一类型，这些参数和返回值也需要与我们正在处理的集合中的条目类型相同。

通过归约操作，累加器函数的输出现在可以与集合条目具有不同的类型，只要它与提供的初始值和累加器函数的第一个参数相同即可：

```text
concat <- (acc, next) => acc + next.toString();
result <- [1, 2, 3].reduce(concat, "");
```

这相当于：

```java
result <- concat(concat(concat("", 1.toString()), 2.toString()), 3.toString());
```

这里，结果将是一个字符串，而集合条目是整数。

## 4. 左折叠 vs 右折叠

在讨论折叠操作时，我们经常会看到它们被称为“左折叠”或“右折叠”，**这决定了我们应该从集合的左端还是右端开始应用累加器函数**。

如果没有指定方向，通常表示向左折叠。在本例中，我们希望从集合左侧开始合并值。

这正是我们已经看到的：

```text
result <- [1, 2, 3, 4].foldLeft(plus);
result <- plus(plus(plus(1, 2), 3), 4);
```

在这里，我们将集合中的前两个元素与累加器函数合并。接下来，我们将这个结果与集合中的下一个元素合并。如此循环，直到到达集合末尾。

**右折叠意味着我们开始组合集合右侧的值**：

```text
result <- [1, 2, 3, 4].foldRight(plus);
result <- plus(1, plus(2, plus(3, 4)));
```

在这种情况下，我们将集合中的第一个条目与折叠集合其余部分的结果合并。然后重复此步骤，直到到达集合的末尾。

## 5. 运算顺序

左折叠和右折叠看起来非常相似，那么它们之间有什么区别呢？**最显著的区别在于操作的执行顺序**，在某些情况下，这完全没有区别。然而，在其他情况下，这却可能至关重要。

如果我们的累加函数具有[结合律](https://en.wikipedia.org/wiki/Associative_property)，那么左折叠和右折叠将产生完全相同的结果，我们可以直接从结合律的定义中看到这一点：

```text
(x + y) + z = x + (y + z) for all x, y, z in S
```

等式左边对应于“左折叠”的操作方式，右边对应于“右折叠”的操作方式。因此，如果运算符合结合律，则“左折叠”和“右折叠”的结果相同。

但是，**如果运算不满足结合律，我们就会得到不同的结果**。例如，我们先不看加法，而是看subtract函数：

```text
leftResult <- [1, 2, 3].foldLeft(subtract); // (1 - 2) - 3 = -4
rightResult <- [1, 2, 3].foldRight(subtract); // 1 - (2 - 3) = 2
```

因此，如果我们知道我们的累加器函数不具有结合性，那么选择正确的折叠方向就变得很重要。但是，如果我们的累加器函数具有结合性，那么我们需要探索选择向左折叠还是向右折叠的其他原因。

## 6. 实现效率

**另一个需要考虑的因素是实现的效率**，传统的左折叠和右折叠定义是递归的：

```text
algorithm FoldLeft(collection, initial, fn):
    if collection.empty:
        return initial;

    head <- collection[0]
    tail <- collection.slice(1)

    return FoldLeft(tail, fn(initial, head), fn)

algorithm FoldRight(collection, initial, fn):
    if collection.empty:
        return initial;

    head <- collection[0]
    tail <- collection.slice(1)

    return fn(head, FoldRight(tail, initial, fn))
```

值得注意的是，这些示例都提供了初始值，但这只是为了方便理解。同样，不使用初始值也可以实现这两个示例。

这两个算法非常相似，唯一的区别在于递归调用的方式。然而，从效率的角度来看，这种差异非常重要。具体来说，**左折叠是[尾递归](https://www.baeldung.com/cs/tail-vs-non-tail-recursion)的，而右折叠不是**。因此，**我们可以轻松地将左折叠重写为迭代的。这样一来，时间和内存使用效率都会更高**：

```text FoldLeft(collection, initial, fn):
    result <- initial

    for next in collection:
        result <- fn(result, next)

    return result
```

将右折叠重写为可迭代的要困难得多，在某些情况下，这可能是不可能的——它要求集合是可逆迭代的，但并非所有集合都是如此。

这意味着，对于同一个集合，左折叠可能比右折叠更高效。然而，正如我们之前所见，这假设该操作是结合的。此外，效率提升可能只在非常大的集合中才会显著。

## 7. 总结

在本文中，我们了解了函数式编程中折叠的含义。我们探讨了左折叠和右折叠之间的一些区别，以及如何在它们之间进行选择。