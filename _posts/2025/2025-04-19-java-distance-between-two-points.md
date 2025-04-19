---
layout: post
title:  用Java计算两点之间的距离
category: algorithms
copyright: algorithms
excerpt: 距离
---

## 1. 概述

在本快速教程中，我们将展示如何在Java中计算两点之间的距离。

## 2. 距离的数学公式

假设平面上有两点：第一点A的坐标为(x1,y1)，第二点B的坐标为(x2,y2)，我们想计算两点之间的距离AB。

首先，我们来画一个直角三角形，斜边为AB：

![](/assets/images/2025/algorithms/javadistancebetweentwopoints01.png)

根据勾股定理，三角形两条直角边的平方和等于三角形斜边的平方：AB<sup>2</sup> = AC<sup>2</sup> + CB<sup>2</sup>。

其次，我们来计算AC和CB。

明显地：

```text
AC = y2 - y1
```

相似地：

```text
BC = x2 - x1
```

让我们代入等式的各部分：

```text
distance * distance = (y2 - y1) * (y2 - y1) + (x2 - x1) * (x2 - x1)
```

最后，通过上面的等式我们可以计算出点之间的距离：

```text
distance = sqrt((y2 - y1) * (y2 - y1) + (x2 - x1) * (x2 - x1))
```

现在让我们进入实现部分。

## 3. Java实现

### 3.1 使用简单公式

虽然java.lang.Math和java.awt.geom.Point2D包提供了现成的解决方案，但我们首先按原样实现上述公式：

```java
public double calculateDistanceBetweenPoints(double x1, double y1, double x2, double y2) {       
    return Math.sqrt((y2 - y1) * (y2 - y1) + (x2 - x1) * (x2 - x1));
}
```

为了测试这个答案，我们以直角边为3和4的三角形为例(如上图所示)。很明显，斜边的值5是合适的：

```text
3 * 3 + 4 * 4 = 5 * 5
```

让我们检查一下解决方案：

```java
@Test
public void givenTwoPoints_whenCalculateDistanceByFormula_thenCorrect() {
    double x1 = 3;
    double y1 = 4;
    double x2 = 7;
    double y2 = 1;

    double distance = service.calculateDistanceBetweenPoints(x1, y1, x2, y2);

    assertEquals(distance, 5, 0.001);
}
```

### 3.2 使用java.lang.Math包

如果calculateDistanceBetweenPoints()方法中的乘法结果过大，则可能会发生溢出。与此不同，Math.hypot()方法可以防止中间溢出或下溢：

```java
public double calculateDistanceBetweenPointsWithHypot(
    double x1, 
    double y1, 
    double x2, 
    double y2) {
        
    double ac = Math.abs(y2 - y1);
    double cb = Math.abs(x2 - x1);
        
    return Math.hypot(ac, cb);
}
```

让我们取和以前相同的点并检查距离是否相同：

```java
@Test
public void givenTwoPoints_whenCalculateDistanceWithHypot_thenCorrect() {
    double x1 = 3;
    double y1 = 4;
    double x2 = 7;
    double y2 = 1;

    double distance = service.calculateDistanceBetweenPointsWithHypot(x1, y1, x2, y2);

    assertEquals(distance, 5, 0.001);
}
```

### 3.3 使用java.awt.geom.Point2D包

最后，让我们用Point2D.distance()方法计算距离：

```java
public double calculateDistanceBetweenPointsWithPoint2D( 
    double x1, 
    double y1, 
    double x2, 
    double y2) {
        
    return Point2D.distance(x1, y1, x2, y2);
}
```

现在让我们以同样的方式测试该方法：

```java
@Test
public void givenTwoPoints_whenCalculateDistanceWithPoint2D_thenCorrect() {

    double x1 = 3;
    double y1 = 4;
    double x2 = 7;
    double y2 = 1;

    double distance = service.calculateDistanceBetweenPointsWithPoint2D(x1, y1, x2, y2);

    assertEquals(distance, 5, 0.001);
}
```

## 4. 总结

在本教程中，我们展示了几种在Java中计算两点之间距离的方法。