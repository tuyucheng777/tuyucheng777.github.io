---
layout: post
title:  Java程序估算Pi
category: algorithms
copyright: algorithms
excerpt: Pi
---

## 1. 概述

圆周率(π)是圆的周长与直径的比值，无论圆的大小，π的值都约等于3.14159，它适用于与圆相关的计算。

在本教程中，我们将学习如何使用Java计算圆周率(π)，我们将采用蒙特卡洛算法来解决此任务。

## 2. 蒙特卡洛算法

圆周率(π)是一个无理数-它不能用简单的分数或确定的小数来表示，我们可以使用各种数学方法将圆周率计算到任何所需的精度。

首先，蒙特卡洛方法是估计圆周率(π)值的方法之一，该方法利用随机抽样来获得数学问题的数值解。

例如，假设我们有一块空画布。接下来，我们在画布上画一个正方形，并在正方形内画一个大圆。然后，我们在正方形内生成随机点，有些点会落在圆内，有些点会落在圆外。为了估算圆周率π，我们需要计算点的总数以及圆内点的总数。[下图](https://www.101computing.net/wp/wp-content/uploads/estimating-pi-monte-carlo-method.png)描述了正方形、随机生成的点和内切圆：

![](/assets/images/2025/algorithms/javamontecarlocomputepi01.png)

我们知道，圆的面积是圆周率乘以半径平方，而圆内切正方形的面积是圆周率乘以半径平方。如果我们用圆的面积除以圆内切正方形的面积，就等于圆周率除以4，这个比率也适用于正方形内点的数量和圆内点的数量。

因此，让我们看看使用蒙特卡洛方法估计pi的公式：

![](/assets/images/2025/algorithms/javamontecarlocomputepi02.png)

接下来，我们将用Java实现该[公式](https://www.baeldung.com/java-lang-math)。

## 3. Pi Java程序

让我们看一个使用蒙特卡洛算法计算圆周率的简单Java程序：

```java
@Test
void givenPiCalculator_whenCalculatePiWithTenThousandPoints_thenEstimatedPiIsWithinTolerance() {
    int totalPoints = 10000;
    int insideCircle = 0;

    Random random = new Random();
    for (long i = 0; i < totalPoints; i++) {
        double x = random.nextDouble() * 2 - 1;
        double y = random.nextDouble() * 2 - 1;
        if (x * x + y * y <= 1) {
            insideCircle++;
        }
    }
    double pi = 4.0 * insideCircle / totalPoints;
    assertEquals(Math.PI, pi, 0.01);
}
```

在上面的例子中，我们在以原点为中心、边长为2的正方形内生成10000个随机点。

接下来，我们检查每个点是否在圆内，任何距离原点一以内的点都算作在圆内。

最后，我们通过找到圆内的点与总点数的比例并将结果乘以4来估算圆周率的值。

## 4. 总结

在本文中，我们学习了如何使用蒙特卡洛算法估算圆周率(π)的值。虽然还有其他一些数学方法可以估算圆周率，但蒙特卡洛方法简单易用，易于在Java中实现。