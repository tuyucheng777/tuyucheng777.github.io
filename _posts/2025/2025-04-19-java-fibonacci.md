---
layout: post
title:  Java中的斐波那契数列
category: algorithms
copyright: algorithms
excerpt: 斐波那契
---

## 1. 概述

在本教程中，我们将研究斐波那契数列。

具体来说，我们将实现三种方法来计算斐波那契数列的第n项，最后一种方法是恒定时间解。

## 2. 斐波那契数列

**斐波那契数列是一系列数字，其中每一项都是前两项之和，它的前两项分别为0和1**。

例如，该级数的前11项是0、1、1、2、3、5、8、13、21、34和55。

用数学术语来说，斐波那契数列S<sub>n</sub>由递归关系定义：

```text
S(n) = S(n-1) + S(n-2), with S(0) = 0 and S(1) = 1
```

现在，我们来看看如何计算斐波那契数列的第n项。我们将重点介绍三种方法：递归、迭代和使用比奈公式。

### 2.1 递归方法

对于我们的第一个解决方案，让我们直接用Java表达递归关系：

```java
public static int nthFibonacciTerm(int n) {
    if (n == 1 || n == 0) {
        return n;
    }
    return nthFibonacciTerm(n-1) + nthFibonacciTerm(n-2);
}
```

如我们所见，我们检查n是否等于0或1，如果为true，则返回该值。在其他情况下，我们递归调用该函数计算第(n-1)项和第(n-2)项，并返回它们的和。

虽然递归方法实现起来很简单，但我们发现它进行了大量重复计算。例如，为了计算第6项，我们调用了计算第5项和第4项的函数。此外，计算第5项的函数调用又会调用计算第4项的函数。因此，**[递归方法](https://www.baeldung.com/java-recursion)进行了大量冗余工作**。

事实证明，这使得**它的[时间复杂度](https://www.baeldung.com/cs/fibonacci-computational-complexity)呈指数级增长；确切地说是O(Φ<sup>n</sup>)**。

### 2.2 迭代法

在迭代方法中，我们可以避免递归方法中的重复计算。相反，我们计算序列的项，并[存储前两项以计算下一项](https://www.baeldung.com/java-knapsack#dp)。

我们来看看它的实现：

```java
public static int nthFibonacciTerm(int n) {
    if(n == 0 || n == 1) {
        return n;
    }
    int n0 = 0, n1 = 1;
    int tempNthTerm;
    for (int i = 2; i <= n; i++) {
        tempNthTerm = n0 + n1;
        n0 = n1;
        n1 = tempNthTerm;
    }
    return n1;
}
```

首先，我们检查待计算的项是第0项还是第1项，如果是，则返回初始值。否则，我们使用n0和n1计算第2项。然后，我们修改n0和n1变量的值，分别用于存储第1项和第2项。我们不断迭代，直到计算出所需的项。

迭代方法通过将最后两个斐波那契数存储在变量中来避免重复工作，**迭代方法的时间复杂度和空间复杂度分别为O(n)和O(1)**。

### 2.3 比奈公式

我们只是根据前两个斐波那契数列定义了第n个斐波那契数列，现在，我们将看看比奈公式，如何在常数时间内计算第n个斐波那契数列。

**斐波那契数列保持着一种称为黄金分割率的比率，用Φ表示**，希腊字符发音为“phi”。

首先我们来看看黄金分割率是如何计算的：

```text
Φ = ( 1 + √5 ) / 2 = 1.6180339887...
```

现在，我们来看看比奈公式：

```text
Sn = Φⁿ–(– Φ⁻ⁿ)/√5
```

实际上，**这意味着我们只需一些算术运算就能得到第n个斐波那契数**。

让我们用Java来表达这一点：

```java
public static int nthFibonacciTerm(int n) {
    double squareRootOf5 = Math.sqrt(5);
    double phi = (1 + squareRootOf5)/2;
    int nthTerm = (int) ((Math.pow(phi, n) - Math.pow(-phi, -n))/squareRootOf5);
    return nthTerm;
}
```

我们首先计算5的平方根和φ，并将它们存储在变量中。然后，我们应用比奈公式得出所需的项。

由于我们这里处理的是无理数，所以我们只能得到一个近似值。因此，对于较大的斐波那契数，我们需要保留[更多的小数位](https://www.baeldung.com/java-bigdecimal-biginteger)，以弥补舍入误差。

**我们看到，上述方法在恒定时间内计算第n个斐波那契项，即O(1)**。

## 3. 总结

在这篇简短的文章中，我们研究了斐波那契数列，并研究了递归和迭代解法。然后，我们应用了比奈公式，创建了一个常数时间解法。