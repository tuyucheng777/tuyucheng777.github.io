---
layout: post
title:  在Java中计算阶乘
category: algorithms
copyright: algorithms
excerpt: 阶乘
---

## 1. 概述

给定一个非负整数n，阶乘是所有小于或等于n的正整数的乘积。

在本快速教程中，我们将探索**在Java中计算给定数字阶乘的不同方法**。

## 2. 20以内的阶乘

### 2.1 使用for循环计算阶乘

让我们看一下使用for循环的基本阶乘算法：

```java
public long factorialUsingForLoop(int n) {
    long fact = 1;
    for (int i = 2; i <= n; i++) {
        fact = fact * i;
    }
    return fact;
}
```

上述解决方案对于不超过20的数字有效，但是，如果我们尝试大于20的数字，则将失败，因为结果太大，无法放入long中，从而导致溢出。

让我们再看几个，**注意每个都只适用于较小的数字**。

### 2.2 使用Java 8 Streams计算阶乘

我们还可以使用[Java 8 Stream API](https://www.baeldung.com/java-8-streams-introduction)轻松计算阶乘：

```java
public long factorialUsingStreams(int n) {
    return LongStream.rangeClosed(1, n)
        .reduce(1, (long x, long y) -> x * y);
}
```

在这个程序中，我们首先使用LongStream来遍历1到n之间的数字。然后我们使用reduce()函数，它使用一个标识值和一个累加器函数来完成归约步骤。

### 2.3 使用递归计算阶乘

让我们看另一个阶乘程序的例子，这次使用递归：

```java
public long factorialUsingRecursion(int n) {
    if (n <= 2) {
        return n;
    }
    return n * factorialUsingRecursion(n - 1);
}
```

### 2.4 使用Apache Commons Math计算阶乘

[Apache Commons Math](https://www.baeldung.com/apache-commons-math)有一个CombinatoricsUtils类，它有一个静态factorial方法，我们可以使用它来计算阶乘。

为了包含Apache Commons Math，我们将[commons-math3依赖](https://mvnrepository.com/artifact/org.apache.commons/commons-math3)添加到pom中：

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-math3</artifactId>
    <version>3.6.1</version>
</dependency>
```

让我们看一个使用CombinatoricsUtils类的示例：

```java
public long factorialUsingApacheCommons(int n) {
    return CombinatoricsUtils.factorial(n);
}
```

请注意，它的返回类型是long，就像我们自己开发的解决方案一样。

这意味着如果计算值超过Long.MAX_VALUE，则会抛出MathArithmeticException。

为了变得更大，我们将需要不同的返回类型。

## 3. 大于20的阶乘

### 3.1 使用BigInteger计算阶乘

如前所述，long数据类型仅适用于n <= 20的阶乘。

对于较大的n值，我们可以使用java.math包中的BigInteger类，该类可以保存最大为2^Integer.MAX_VALUE的值：

```java
public BigInteger factorialHavingLargeResult(int n) {
    BigInteger result = BigInteger.ONE;
    for (int i = 2; i <= n; i++)
        result = result.multiply(BigInteger.valueOf(i));
    return result;
}
```

### 3.2 使用Guava进行阶乘

Google的[Guava](https://www.baeldung.com/tag/guava)库还提供了一种用于计算较大数字的阶乘的实用方法。

为了包含该库，我们可以将其[guava依赖](https://mvnrepository.com/search?q=guava)添加到pom中：

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

现在，我们可以使用BigIntegerMath类中的静态阶乘方法来计算给定数字的阶乘：

```java
public BigInteger factorialUsingGuava(int n) {
    return BigIntegerMath.factorial(n);
}
```

## 4. 总结

在本文中，我们看到了使用核心Java以及一些外部库来计算阶乘的几种方法。

我们首先了解了使用long数据类型计算20以下数字阶乘的解决方案；然后，我们了解了几种使用BigInteger处理大于20的数字的方法。