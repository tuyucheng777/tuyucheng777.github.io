---
layout: post
title:  将零移动到Java数组的末尾
category: algorithms
copyright: algorithms
excerpt: 数组
---

## 1. 概述

在Java中使用数组时，一个常见的任务是重新排列数组以优化其结构，其中一个场景是将零移动到数组的末尾。

在本教程中，我们将探索使用Java实现此任务的不同方法。

## 2. 问题介绍

在深入实现之前，让我们首先了解这个问题的要求。

我们的输入是一个整数数组，目标是重新排列这些整数，**使所有0元素都移到数组末尾**。此外，**非0元素的顺序必须保留**。

举个例子可以帮助我们快速理解这个问题，假设我们有一个整数数组：

```text
[ 42, 2, 0, 3, 4, 0 ]
```

重新排列其元素后，我们期望获得与以下结果等同的数组：

```java
static final int[] EXPECTED = new int[] { 42, 2, 3, 4, 0, 0 };
```

接下来，我们将介绍两种解决这个问题的方法，我们还将简要讨论它们的性能特点。

## 3. 使用额外数组

为了解决这个问题，首先想到的可能是使用额外的数组。

假设我们创建了一个新的数组并将其命名为result，**该数组初始化为与输入数组相同的长度，并且其所有元素均设置为0**。

接下来，**我们遍历输入数组，每当遇到非0数字时，我们就相应地更新结果数组中的相应元素**。

让我们来实现这个想法：

```java
int[] array = new int[] { 42, 2, 0, 3, 4, 0 };
int[] result = new int[array.length];
int idx = 0;
for (int n : array) {
    if (n != 0) {
        result[idx++] = n;
    }
}
assertArrayEquals(EXPECTED, result);
```

可以看到，代码非常简单。有两点值得一提：

- 在Java中，[int[]数组使用0作为元素的默认值](https://www.baeldung.com/java-arrays-guide#2-initialization)。因此，初始化结果数组时无需显式地用0填充。
- 当我们在测试中断言两个数组相等时，**我们应该使用[assertArrayEquals()](https://www.baeldung.com/junit-assertions#junit4-assertArrayEquals)而不是assertEquals()**。

在这个解决方案中，我们只遍历输入数组一次，因此，**这种方法具有线性[时间复杂度](https://www.baeldung.com/cs/time-vs-space-complexity)：[O(n)](https://www.baeldung.com/cs/big-oh-asymptotic-complexity)**。但是，由于它复制了输入数组，因此**其空间复杂度为O(n)**。

接下来我们来探讨如何改进该方案，实现就地排列，保持O(1)的常数空间复杂度。

## 4. 线性时间复杂度的就地排列

让我们首先回顾一下“初始化新数组”的方法，我们在新数组上维护了一个非0指针(idx)，这样一旦在原始数组中检测到非0值，我们就知道结果数组中的哪个元素需要更新。

实际上，**我们可以在输入数组上设置非0指针**。这样，当我们迭代输入数组时，就可以将非0元素移到前面，保持它们的相对顺序。迭代完成后，我们将用0填充剩余的位置。

让我们以输入数组为例来了解该算法的工作原理：

```text
Iteration pointer: v
Non-zero-pointer:  ^

v
42, 2, 0, 3, 4, 0
^ (replace 42 with 42)
 
    v
42, 2, 0, 3, 4, 0
    ^ (replace 2 with 2)
 
       v 
42, 2, 0, 3, 4, 0
    ^
 
          v 
42, 2, 3, 3, 4, 0
       ^ (replace 0 with 3)
 
             v
42, 2, 3, 4, 4, 0
          ^ (replace 3 with 4)
 
                v
42, 2, 3, 4, 4, 0
          ^
 
The final step: Fill 0s to the remaining positions:
                v
42, 2, 3, 4, 0, 0
                ^
```

接下来我们来实现这个逻辑：

```java
int[] array = new int[] { 42, 2, 0, 3, 4, 0 };
int idx = 0;
for (int n : array) {
    if (n != 0) {
        array[idx++] = n;
    }
}
while (idx < array.length) {
    array[idx++] = 0;
}
assertArrayEquals(EXPECTED, array);
```

可以看出，上述代码中没有引入任何额外的数组。非0指针idx跟踪非0元素的放置位置，在迭代过程中，如果当前元素非0，则将其移至最前面并增加指针值。迭代完成后，我们使用while循环将剩余位置填充为0。

这种方法执行的是就地重排，也就是说，不需要额外的空间；因此，它的空间复杂度为O(1)。

在最坏的情况下，输入数组中的所有元素都为0，缺点是idx指针在迭代后保持不变。因此，后续的while循环将再次遍历整个数组。尽管如此，**由于迭代执行的次数是常数，因此整体时间复杂度仍然不受影响，为O(n)**。

## 5. 总结

在本文中，我们探讨了两种将0重定位到整数数组末尾的方法。此外，我们还讨论了它们在时间和空间复杂度方面的性能。