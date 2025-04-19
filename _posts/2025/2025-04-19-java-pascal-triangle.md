---
layout: post
title:  Java程序打印帕斯卡三角形
category: algorithms
copyright: algorithms
excerpt: 帕斯卡三角形
---

## 1. 概述

[帕斯卡三角形](https://en.wikipedia.org/wiki/Pascal's_triangle)是二项式系数的三角形排列，帕斯卡三角形中的数字排列方式是，每个数字都是其上方两个数字之和。

在本教程中，我们将了解如何**在Java中打印帕斯卡三角形**。

## 2. 使用递归

我们可以使用递归公式nCr: n! / ((n – r) !r !)来打印帕斯卡三角形。

首先，让我们创建一个递归函数：

```java
public int factorial(int i) {
    if (i == 0) {
        return 1;
    }
    return i * factorial(i - 1);
}
```

然后我们可以使用该函数打印三角形：

```java
private void printUseRecursion(int n) {
    for (int i = 0; i <= n; i++) {
        for (int j = 0; j <= n - i; j++) {
            System.out.print(" ");
        }

        for (int k = 0; k <= i; k++) {
            System.out.print(" " + factorial(i) / (factorial(i - k) * factorial(k)));
        }

        System.out.println();
    }
}
```

n = 5的结果将如下所示：

```text
       1
      1 1
     1 2 1
    1 3 3 1
   1 4 6 4 1
  1 5 10 10 5 1
```

## 3. 避免使用递归

另一种不使用递归来打印帕斯卡三角形的方法是使用二项式展开。

我们总是在每一行的开头都有值1，那么第(n)行和第(i)位置的k值将按如下方式计算：

```text
k = (k * (n - i) / i)
```

让我们使用这个公式创建我们的函数：

```java
public void printUseBinomialExpansion(int n) {
    for (int line = 1; line <= n; line++) {
        for (int j = 0; j <= n - line; j++) {
            System.out.print(" ");
        }

        int k = 1;
        for (int i = 1; i <= line; i++) {
            System.out.print(k + " ");
            k = k * (line - i) / i;
        }

        System.out.println();
    }
}
```

## 4. 总结

在本快速教程中，我们学习了两种在Java中打印帕斯卡三角形的方法。