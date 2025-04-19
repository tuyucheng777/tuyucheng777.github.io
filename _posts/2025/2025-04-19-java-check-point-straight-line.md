---
layout: post
title:  在Java中检查一个点是否位于直线上的两点之间
category: algorithms
copyright: algorithms
excerpt: 直线
---

## 1. 概述

处理二维几何图形时，一个常见的问题是确定一个点是否位于直线上的另外两个点之间。

在本快速教程中，我们将探讨在Java中解决此问题的不同方法。

## 2. 理解问题陈述

假设平面上有两点：第一点A的坐标为(x<sub>1</sub>, y<sub>1</sub>)，第二点B的坐标为(x<sub>2</sub>, y<sub>2</sub>)，我们想检查给定点C的坐标为(x<sub>3</sub>, y<sub>3</sub>)是否位于A和B之间：

![](/assets/images/2025/algorithms/javacheckpointstraightline01.png)

在上图中，点C位于点A和点B之间，而点D不位于点A和点B之间。

## 3. 使用距离公式

**该方法需要计算点A到C、点C到B以及点A到B的距离：AC、CB和AB。如果C位于点A和B之间，则AC与CB之和等于AB**：

```text
distance(AC) + distance(CB) == distance(AB)
```

我们可以使用距离公式来[计算两个不同点之间的距离](https://www.baeldung.com/java-distance-between-two-points)，如果点A的坐标为(x<sub>1</sub>, y<sub>1</sub>)，点B的坐标为(x<sub>2</sub>, y<sub>2</sub>)，那么我们可以使用以下公式计算距离：

```text
distance = sqrt((y2 - y1) * (y2 - y1) + (x2 - x1) * (x2 - x1))
```

让我们使用上面的距离公式来验证该方法：

从点A(1,1)到点B(5,5)的距离 = 5.656

从点A(1,1)到点C(3,3)的距离 = 2.828

从点C(3,3)到点B(5,5)的距离 = 2.828

这里，距离(AC) + 距离(CB) = 5.656，等于距离(AB)，这表明点C位于点A和点B之间。

让我们使用距离公式来检查一个点是否位于两点之间：

```java
boolean isPointBetweenTwoPoints(double x1, double y1, double x2, double y2, double x, double y) {
    double distanceAC = Math.sqrt(Math.pow(x - x1, 2) + Math.pow(y - y1, 2));
    double distanceCB = Math.sqrt(Math.pow(x2 - x,2) + Math.pow(y2 - y, 2));
    double distanceAB = Math.sqrt(Math.pow(x2 - x1,2) + Math.pow(y2 - y1, 2));
    return Math.abs(distanceAC + distanceCB - distanceAB) < 1e-9;
}
```

这里，1e-9是一个较小的eplison值，用于计算浮点计算中可能出现的舍入误差和不精确性，如果绝对差非常小(小于1e-9)，我们就认为它是相等的。

让我们使用上述值来测试这种方法：

```java
void givenAPoint_whenUsingDistanceFormula_thenCheckItLiesBetweenTwoPoints() {
    double x1 = 1;    double y1 = 1;
    double x2 = 5;    double y2 = 5;
    double x = 3;    double y = 3;
    assertTrue(findUsingDistanceFormula(x1, y1, x2, y2, x, y));
}
```

## 4. 使用斜率公式

在这种方法中，我们将使用斜率公式计算直线AB和AC的斜率。**我们将比较这些斜率来检查共线性，即AB和AC的斜率是否相等，这将帮助我们确定点A、B和C是否对齐**。

如果点A的坐标为(x<sub>1</sub>, y<sub>1</sub>)，点B的坐标为(x<sub>2</sub>, y<sub>2</sub>)，则我们可以使用以下公式计算斜率：

```text
slope = (y2 - y1) / (x2 - x1)
```

如果AB和AC的斜率相等，并且点C位于A点和B点定义的x和y坐标范围内，则可以说点C位于点A点和点B点之间。

让我们计算以上AB和AC的斜率来验证该方法：

AB的斜率 = 1.0

AC的斜率 = 1.0

点C是(3,3)

这里，AB = AC，点C的x和y坐标位于A(1,1)和B(5,5)定义的范围之间，这表明点C位于点A和点B之间。 

让我们使用这种方法来检查一个点是否位于两点之间：

```java
boolean findUsingSlopeFormula(double x1, double y1, double x2, double y2, double x, double y) {
    double slopeAB = (y2 - y1) / (x2 - x1);
    double slopeAC = (y - y1) / (x - x1);
    return slopeAB == slopeAC && ((x1 <= x && x <= x2) || (x2 <= x && x <= x1)) && ((y1 <= y && y <= y2) || (y2 <= y && y <= y1));
}
```

让我们使用上述值来测试这种方法：

```java
void givenAPoint_whenUsingSlopeFormula_thenCheckItLiesBetweenTwoPoints() { 
    double x1 = 1;    double y1 = 1;
    double x2 = 5;    double y2 = 5;
    double x = 3;    double y = 3;
    assertTrue(findUsingSlopeFormula(x1, y1, x2, y2, x, y));
}
```

## 5. 总结

在本教程中，我们讨论了确定一个点是否位于直线上的另外两个点之间的方法。