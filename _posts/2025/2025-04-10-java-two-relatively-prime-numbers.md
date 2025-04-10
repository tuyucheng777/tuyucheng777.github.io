---
layout: post
title:  在Java中判断两个数是否互质
category: algorithms
copyright: algorithms
excerpt: 质数
---

## 1. 概述

**给定两个整数a和b，如果能整除它们的唯一因数是1，我们说它们是互质的，互质是互质数的同义词**。

在本快速教程中，我们将介绍使用Java解决此问题的方法。

## 2. 最大公因数算法

事实证明，如果两个数a和b的最大公约数(gcd)为1(即gcd(a,b) = 1)，则a和b互质。因此，判断两个数是否互质只需判断gcd是否为1。

## 3. 欧几里得算法实现

在本节中，我们将使用[欧几里得算法](https://en.wikipedia.org/wiki/Euclidean_algorithm)来计算两个数字的最大公约数。

在展示我们的实现之前，让我们总结一下算法，并看一个如何应用它的简单例子以便于理解。

假设我们有两个整数a和b，在迭代方法中，我们首先将a除以b并得出余数。接下来，我们将b的值赋给a，将余数值赋给b。重复此过程，直到b = 0。最后，当到达这一点时，我们返回a的值作为gcd结果，如果a = 1，我们可以说a和b互质。

让我们对两个整数a = 81和b = 35尝试一下。

在这种情况下，81和35的余数(81 % 35)是11。因此，在第一个迭代步骤中，我们以a = 35和b = 11结束。因此，我们将进行另一次迭代。

35除以11的余数是2，因此，我们现在得到a = 11(我们交换了值)和b = 2。

再进一步，结果将为a = 2和b = 1。

最后，再经过一次迭代，我们将得到a = 1和b = 0。算法返回1，我们可以得出结论，81和35确实是互质的。

### 3.1 命令式实现

首先，让我们实现上面描述的欧几里得算法的命令式Java版本：
```java
int iterativeGCD(int a, int b) {
    int tmp;
    while (b != 0) {
        if (a < b) {
            tmp = a;
            a = b;
            b = tmp;
        }
        tmp = b;
        b = a % b;
        a = tmp;
    }
    return a;
}
```

我们可以注意到，当a小于b时，我们先交换值再继续。当b为0时，算法停止。

### 3.2 递归实现

接下来，我们来看一个递归实现，这可能更简洁，因为它避免了显式的变量值交换：
```java
int recursiveGCD(int a, int b) {
    if (b == 0) {
        return a;
    }
    if (a < b) {
        return recursiveGCD(b, a);
    }
    return recursiveGCD(b, a % b);
}
```

## 4. 使用BigInteger的实现

BigInteger类提供了一个gcd方法，该方法实现了用于查找最大公约数的欧几里得算法。

利用这种方法，我们可以更容易地编写互质算法如下：
```java
boolean bigIntegerRelativelyPrime(int a, int b) {
    return BigInteger.valueOf(a).gcd(BigInteger.valueOf(b)).equals(BigInteger.ONE);
}
```

## 5. 总结

在此快速教程中，我们提出了使用gcd算法的三种实现来解决两个数字是否互质的问题。