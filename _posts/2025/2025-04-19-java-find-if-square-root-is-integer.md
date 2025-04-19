---
layout: post
title:  在Java中确定整数的平方根是否为整数
category: algorithms
copyright: algorithms
excerpt: 平方根
---

## 1. 概述

**完全平方数是可以表示为两个相等整数的乘积的数**。

在本文中，我们将探索Java中多种判断整数是否为完全平方的方法。此外，我们还将讨论每种技术的优缺点，以确定其效率以及哪种技术最快。

## 2. 检查整数是否为完全平方数

众所周知，Java提供了两种定义整数的数据类型。第一种是int，它表示32位数字；另一种是long，它表示64位数字。在本文中，我们将使用long数据类型来处理最坏情况(最大可能的整数)。

由于Java用64位表示长整数，因此长整数的范围是从-9,223,372,036,854,775,808到9,223,372,036,854,775,807。而且，由于我们要处理的是完全平方数，所以我们只关心处理正整数集，因为任何整数乘以自身都会得出一个正数。

另外，由于最大的数字约为2<sup>63</sup>，这意味着大约有2<<sup>31.5</sup>个整数的平方小于2<sup>63</sup>。同样，我们可以假设，查找这些数字的表是低效的。

### 2.1 在Java中使用sqrt方法

**检查一个整数是否为完全平方最简单、最直接的方法是使用sqrt函数**。众所周知，sqrt函数返回一个double值。因此，我们需要将结果转换为int类型，并将其乘以自身。然后，检查结果是否等于我们一开始的整数：

```java
public static boolean isPerfectSquareByUsingSqrt(long n) {
    if (n <= 0) {
        return false;
    }
    double squareRoot = Math.sqrt(n);
    long tst = (long)(squareRoot + 0.5);
    return tst*tst == n;
}
```

请注意，由于处理double值时可能会遇到精度误差，我们可能需要在结果上加0.5。有时，将整数赋给double变量时，可以用小数点表示。

例如，如果我们将数字3赋给一个double变量，那么它的值可能是3.00000001或2.99999999。因此，为了避免这种情况，我们在将其转换为long之前加0.5，以确保我们得到的是实际值。

此外，如果我们用一个数字测试sqrt函数，我们会注意到执行时间很快。另一方面，如果我们需要多次调用sqrt函数，并且我们尝试减少sqrt函数执行的运算次数，这种微优化实际上可能会产生影响。

### 2.2 使用二分查找

**我们可以使用二分查找来找到一个数字的平方根，而无需使用sqrt函数**。

由于数字的范围是1到2<sup>63</sup>，根在1到2<sup>31.5</sup>之间。因此，二分搜索算法需要大约16次迭代才能得到平方根：

```java
public boolean isPerfectSquareByUsingBinarySearch(long low, long high, long number) {
    long check = (low + high) / 2L;
    if (high < low) {
        return false;
    }
    if (number == check * check) {
        return true;
    }
    else if (number < check * check) {
        high = check - 1L;
        return isPerfectSquareByUsingBinarySearch(low, high, number);
    }
    else {
        low = check + 1L;
        return isPerfectSquareByUsingBinarySearch(low, high, number);
    }
}
```

### 2.3 二分查找的增强

为了增强二分查找，我们可以注意到，如果我们确定了基数的位数，那么我们就得到了根的范围。

例如，如果数字仅由一位数字组成，则平方根的范围在1到4之间。原因是一位数字的最大整数是9，而它的根是3。此外，如果数字由两位数字组成，则范围在4到10之间，依此类推。

因此，**我们可以构建一个查找表，根据起始数字的位数来指定平方根的范围，这将缩小二分查找的范围**。因此，得到平方根所需的迭代次数会更少：

```java
public class BinarySearchRange {
    private long low;
    private long high;

    // standard constructor and getters
}
```

```java
private void initiateOptimizedBinarySearchLookupTable() {
    lookupTable.add(new BinarySearchRange());
    lookupTable.add(new BinarySearchRange(1L, 4L));
    lookupTable.add(new BinarySearchRange(3L, 10L));
    for (int i = 3; i < 20; i++) {
        lookupTable.add(
                new BinarySearchRange(
                        lookupTable.get(i - 2).low * 10,
                        lookupTable.get(i - 2).high * 10));
    }
}
```

```java
public boolean isPerfectSquareByUsingOptimizedBinarySearch(long number) {
    int numberOfDigits = Long.toString(number).length();
    return isPerfectSquareByUsingBinarySearch(
            lookupTable.get(numberOfDigits).low,
            lookupTable.get(numberOfDigits).high,
            number);
}
```

### 2.4 整数运算的牛顿法

**一般来说，我们可以用牛顿法求任意数的平方根，即使是非整数**。牛顿法的基本思想是假设一个数X是另一个数N的平方根，然后，我们可以开始一个循环，不断计算这个根，最终一定会得到N的正确平方根。

**然而，通过对牛顿法进行一些修改，我们可以用它来检查一个整数是否是完全平方数**：

```java
public static boolean isPerfectSquareByUsingNewtonMethod(long n) {
    long x1 = n;
    long x2 = 1L;
    while (x1 > x2) {
        x1 = (x1 + x2) / 2L;
        x2 = n / x1;
    }
    return x1 == x2 && n % x1 == 0L;
}
```

## 3. 优化整数平方根算法

正如我们所讨论的，有多种算法可以检查整数的平方根。然而，**我们总是可以通过一些技巧来优化任何算法**。

**技巧应该考虑避免执行确定平方根的主要运算**，例如，我们可以直接排除负数。

我们可以利用的一个事实是“**在16进制中，完全平方数的尾数只能是0、1、4或9**”。因此，我们可以在开始计算之前将整数转换为16进制。之后，我们排除将该数视为非完全平方根的情况：

```java
public static boolean isPerfectSquareWithOptimization(long n) {
    if (n < 0) {
        return false;
    }
    switch((int)(n & 0xF)) {
        case 0: case 1: case 4: case 9:
            long tst = (long)Math.sqrt(n);
            return tst*tst == n;
        default:
            return false;
    }
}
```

## 4. 总结

在本文中，我们讨论了多种判断整数是否为完全平方的方法。正如我们所见，我们总是可以通过一些技巧来增强算法。

这些技巧会在算法开始主要操作之前排除大量情况，因为很多整数很容易被判定为非完全平方数。