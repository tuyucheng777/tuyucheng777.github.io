---
layout: post
title:  在Java中查找给定数字下的最大素数
category: algorithms
copyright: algorithms
excerpt: Log4j
---

## 1. 概述

寻找小于给定数字的最大[素数](https://en.wikipedia.org/wiki/Prime_number)是计算机科学和数学中的一个经典问题。

在这个简短的教程中，我们将探讨两种用Java解决此问题的方法。

## 2. 使用暴力破解

让我们从最直接的方式开始，我们可以通过从给定数开始向后迭代直到找到一个素数，来找到给定数下的最大素数。对于每个数，我们通过验证它不能被除1之外的任何小于自身的数字整除来检查它是否是素数：
```java
public static int findByBruteForce(int n) {
    for (int i = n - 1; i >= 2; i--) {
        if (isPrime(i)) {
            return i;
        }
    }
    return -1; // Return -1 if no prime number is found
}

public static boolean isPrime(int number) {
    for (int i = 2; i <= Math.sqrt(number); i++) {
        if (number % i == 0) {
            return false;
        }
    }
    return true;
}
```

isPrime()方法的时间复杂度为O(√N)，我们可能需要检查最多n个数字。因此，**该解决方案的时间复杂度为O(N√N)**。

## 3. 使用埃拉托斯特尼筛法

找到给定数字以下最大素数的更有效方法是使用[埃拉托斯特尼筛法](https://www.baeldung.com/cs/sieve-of-eratosthenes)算法，该算法可以高效地考虑给定范围内的所有素数。一旦我们得到了所有素数，就能轻松找到小于给定数的最大素数：
```java
public static int findBySieveOfEratosthenes(int n) {
    boolean[] isPrime = new boolean[n];
    Arrays.fill(isPrime, true);
    for (int p = 2; p*p < n; p++) {
        if (isPrime[p]) {
            for (int i = p * p; i < n; i += p) {
                isPrime[i] = false;
            }
        }
    }

    for (int i = n - 1; i >= 2; i--) {
        if (isPrime[i]) {
            return i;
        }
    }
    return -1;
}
```

这次，我们使用与第一个解决方案相同的isPrime()方法；我们的代码遵循3个基本步骤：

1. 初始化一个布尔数组isPrime[]来跟踪最多n个数字的素数状态，默认为true。
2. 对于每个素数p，从p * p到n将其倍数标记为非素数(false)，这可以有效地过滤掉非素数。
3. 从n-1向后迭代，找到标记为true的最高索引。

**埃拉托斯特尼筛选法的时间复杂度为O(N log(log(N)))**，对于较大的n，这比蛮力方法效率高得多。

## 4. 总结

在本教程中，我们探索了两种在Java中查找给定数字下最大素数的方法。暴力破解法更直接，但效率较低。埃拉托斯特尼筛法提供了一种更有效的解决方案，时间复杂度为O(n log log n)，因此对于较大的数字，它是首选。