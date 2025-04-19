---
layout: post
title:  在Java中寻找最大公约数
category: algorithms
copyright: algorithms
excerpt: 最大公约数
---

## 1. 概述

在数学中，**两个非零整数的[最大公约数(GCD)](https://en.wikipedia.org/wiki/Greatest_common_divisor)是能够整除这两个整数的最大正整数**。

在本教程中，我们将介绍三种求两个整数最大公约数(GCD)的方法。此外，我们还将介绍它们在Java中的实现。

## 2. 暴力破解

对于我们的第一种方法，我们从1迭代到给定的最小数，并检查给定的整数是否可以被索引整除，能整除给定数的最大索引就是给定数的最大公约数(GCD)。

```java
int gcdByBruteForce(int n1, int n2) {
    int gcd = 1;
    for (int i = 1; i <= n1 && i <= n2; i++) {
        if (n1 % i == 0 && n2 % i == 0) {
            gcd = i;
        }
    }
    return gcd;
}
```

我们可以看出，上述实现的复杂度是O(min(n1,n2))，因为我们需要循环迭代n次(相当于较小的数字)来找到GCD。

## 3. 欧几里得算法

其次，我们可以使用欧几里得算法来求最大公约数(GCD)。欧几里得算法不仅高效，而且易于理解，并且易于在Java中使用递归实现。

欧几里得方法依赖于两个重要定理：

- 首先，如果我们从较大的数字中减去较小的数字，GCD不会改变-**因此，如果我们继续减去这个数字，我们最终会得到它们的GCD**
- 其次，当较小的数字恰好能整除较大的数字时，较小的数字就是两个给定数字的最大公约数

请注意，在我们的实现中，我们将使用模数而不是减法，因为它基本上一次进行多次减法：

```java
int gcdByEuclidsAlgorithm(int n1, int n2) {
    if (n2 == 0) {
        return n1;
    }
    return gcdByEuclidsAlgorithm(n2, n1 % n2);
}
```

另外，请注意我们如何在算法的递归步骤中使用n2在n1的位置，并使用余数在n2的位置。

此外，**[欧几里得算法的复杂度](https://www.baeldung.com/cs/euclid-time-complexity)为O(Log min(n1,n2))，与我们之前看到的暴力方法相比要好一些**。

## 4. Stein算法或二进制GCD算法

最后，我们可以使用Stein算法(也称为二进制GCD算法)来查找两个非负整数的GCD。该算法使用简单的算术运算，例如算术移位、比较和减法。

Stein算法反复应用与GCD相关的以下基本恒等式来寻找两个非负整数的GCD：

1. gcd(0,0) = 0,gcd(n1,0) = n1,gcd(0,n2) = n2
2. 当n1和n2均为偶数时，则gcd(n1,n2) = 2 * gcd(n1/2,n2/2)，因为2是公约数
3. 如果n1是偶数，n2是奇数，则gcd(n1,n2) = gcd(n1/2,n2)，因为2不是公约数，反之亦然
4. 如果n1和n2都是奇数，且n1 >= n2，则gcd(n1,n2) = gcd((n1-n2)/2,n2)，反之亦然

我们重复步骤2-4，直到n1等于n2，或n1 = 0。最大公约数(GCD)为(2<sup>n</sup>) * n2。其中，n表示在执行步骤2时，2在n1和n2中出现次数：

```java
int gcdBySteinsAlgorithm(int n1, int n2) {
    if (n1 == 0) {
        return n2;
    }

    if (n2 == 0) {
        return n1;
    }

    int n;
    for (n = 0; ((n1 | n2) & 1) == 0; n++) {
        n1 >>= 1;
        n2 >>= 1;
    }

    while ((n1 & 1) == 0) {
        n1 >>= 1;
    }

    do {
        while ((n2 & 1) == 0) {
            n2 >>= 1;
        }

        if (n1 > n2) {
            int temp = n1;
            n1 = n2;
            n2 = temp;
        }
        n2 = (n2 - n1);
    } while (n2 != 0);
    return n1 << n;
}
```

我们可以看到，我们使用算术移位运算来除以或乘以2。此外，我们使用减法来减少给定的数字。

**当n1 > n2时，Stein算法的复杂度为O((log<sub>2</sub>n1)<sup>2</sup>)，而当n1 < n2时，其复杂度为O((log<sub>2</sub>n2)<sup>2</sup>)**。

## 5. 总结

在本教程中，我们研究了计算两个数的最大公约数(GCD)的各种方法，我们还用Java实现了这些方法，并快速了解了它们的复杂度。