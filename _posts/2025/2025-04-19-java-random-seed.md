---
layout: post
title:  Java中随机种子的工作原理
category: algorithms
copyright: algorithms
excerpt: 随机种子
---

## 1. 概述

随机性是一个引人入胜的概念，在密码学、游戏、模拟和机器学习等各个领域都有应用。在计算机系统中，真正的随机性难以捉摸。

在Java中，随机性通常使用伪随机数生成器(PRNG)来生成，这些生成器并非真正随机，而是依赖于一些算法，这些算法会生成看似随机的数字序列，但这些数字序列由一个起点(称为种子)决定。

在本教程中，我们将探索这些随机种子在Java中的工作原理，并揭示它们在随机数生成中的作用。我们还将讨论不同的Java类如何利用这些种子来生成可预测的随机值序列，以及这些机制对各种应用的影响。

## 2. Random类原理

要在Java中[生成随机数](https://www.baeldung.com/java-generating-random-numbers)，我们使用[Random](https://www.baeldung.com/java-17-random-number-generators#2-random)类，该类生成的数字看似随机。然而，我们得到的是一个伪随机数，这意味着虽然序列看似随机，但确定性算法会根据初始输入(种子)生成它。

**包括Java在内的许多编程语言中Random的实现都使用线性同余生成器(LCG)算法，该算法基于一个简单的数学公式生成一个数字序列**：

```text
Xn+1 = (aXn + C) % m
```

其中X<sub>n</sub>为当前值，X<sub>n+1</sub>为下一个值，a为乘数，c为增量，m为模数，初始值X<sub>0</sub>为种子。

a、c和m的选择会显著影响生成的随机数的质量，对m取余数，类似于计算一个球最终会落在一个有编号区域的旋转轮盘上的哪个位置。

例如，当m=10且X<sub>0</sub> = a = c = 7时获得的序列为：

```text
7,6,9,0,7,6,9,0,...
```

如上例所示，对于a、m、c和X<sub>0</sub>的所有值，该序列并不总是随机的。

## 3. 种子的作用

种子是启动[PRNG](https://en.wikipedia.org/wiki/Pseudorandom_number_generator)进程的初始输入，种子就像一把钥匙，可以从一个庞大的预定集合中解锁特定的数字序列。**使用相同的种子总是会产生相同的数字序列，例如，用35的种子初始化一个Random对象，并要求它生成12个随机数，每次运行代码都会得到相同的序列**：

```java
public void givenNumber_whenUsingSameSeeds_thenGenerateNumbers() {
    Random random1 = new Random(35);
    Random random2 = new Random(35);

    int[] numbersFromRandom1 = new int[12];
    int[] numbersFromRandom2 = new int[12];

    for(int i = 0 ; i < 12; i++) {
        numbersFromRandom1[i] = random1.nextInt();
        numbersFromRandom2[i] = random2.nextInt();
    }
    assertArrayEquals(numbersFromRandom1, numbersFromRandom2);
}
```

当我们需要可预测的测试或调试、模拟和加密结果时，此属性至关重要，但它也允许在需要时实现随机性。

## 4. Java中的默认种子

**我们可以创建一个Random类对象而不指定种子，Java将使用当前系统时间作为种子。在内部，Random类会调用其构造函数，该构造函数接收一个long类型的种子参数，但它会根据系统时间计算该种子**。

这种方法提供了一定程度的随机性，但并不完美。系统时间相对可预测，并且两个Random对象可能几乎同时创建，并且具有相似的种子，从而导致相关的随机序列。

我们可以使用[System.nanoTime()](https://www.baeldung.com/java-system-currenttimemillis-vs-system-nanotime#the-systmnanotim-method)来获取更精确、更难以预测的种子。然而，即使是这种方法也有局限性。对于真正无法预测的数字，我们需要使用加密随机数生成器(CSPRNG)或基于硬件的随机数生成器(HRNG)。

让我们看看如何使用System.nanoTime()作为种子：

```java
public void whenUsingSystemTimeAsSeed_thenGenerateNumbers() {
    long seed = System.nanoTime();
    Random random = new Random(seed);
 
    for(int i = 0; i < 10; i++) {
        int randomNumber = random.nextInt(100);
        assertTrue(randomNumber >= 0 && randomNumber < 100);
    }
}
```

## 5. 超越Random类

我们可以使用Random类轻松地在Java中[生成随机数](https://www.baeldung.com/java-generating-random-numbers)，但是，还有其他可用的选项，有些选项更适合需要高质量或加密安全随机数的应用程序。

### 5.1 SecureRandom

java.util.Random的标准JDK实现使用[线性同余生成器](https://en.wikipedia.org/wiki/Linear_congruential_generator)(LCG)算法来生成随机数，该算法的问题在于其加密强度不够。**换句话说，生成的值更容易预测，因此攻击者可能会利用它来入侵我们的系统**。

为了解决这个问题，我们应该在任何安全决策中使用[java.security.SecureRandom](https://www.baeldung.com/java-secure-random)。

### 5.2 ThreadLocalRandom

Random类在多线程环境中性能不佳，原因是争用-假设多个线程共享同一个Random实例。

为了解决这个限制，**Java在JDK 7中引入了[java.util.concurrent.ThreadLocalRandom](https://www.baeldung.com/java-thread-local-random)类-用于在多线程环境中生成随机数**。

## 6. 总结

在本文中，我们看到种子在控制Random类的行为中起着关键作用。我们还观察到使用相同的种子如何始终如一地产生相同的随机数序列，从而得到相同的输出。

当我们理解随机种子的作用及其背后的算法时，我们就能在Java应用程序中做出明智的随机数生成选择，这有助于我们确保它们满足我们对质量、可重复性和安全性的特定需求。