---
layout: post
title:  在Java中查找两条线的交点
category: algorithms
copyright: algorithms
excerpt: 交点
---

## 1. 概述

在本快速教程中，我们将展示**如何找到斜率截距形式中线性函数定义的两条线的交点**。

## 2. 交集的数学公式

平面上的任何直线(垂直线除外)都可以用线性函数定义：

```text
y = mx + b
```

其中m是斜率，b是y截距。

对于垂直线，m等于无穷大，这就是我们将其排除的原因。如果两条线平行，则它们的斜率相同，即m的值相同。

假设我们有两条线，第一个函数定义第一条：

```text
y = m1x + b1
```

第二个函数定义第二条：

```text
y = m2x + b2
```

![](/assets/images/2025/algorithms/javaintersectionoftwolines01.png)

我们想找到这两条线的交点，显然，对于交点，该方程成立：

```text
y1 = y2
```

让我们代入y变量：

```text
m1x + b1 = m2x + b2
```

**从上面的等式我们可以找到x坐标**：

```text
x(m1 - m2) = b2 - b1
x = (b2 - b1) / (m1 - m2)
```

**最后，我们可以找到交点的y坐标**：

```text
y = m1x + b1
```

现在让我们进入实现部分。

## 3. Java实现

首先，我们有4个输入变量：第一条线的m1、b1，以及第二条线的m2、b2。

其次，我们将计算出的交点转换为java.awt.Point类型的对象。

最后，线可能是平行的，因此我们将返回值设为Optional<Point\>：

```java
public Optional<Point> calculateIntersectionPoint(
        double m1,
        double b1,
        double m2,
        double b2) {

    if (m1 == m2) {
        return Optional.empty();
    }

    double x = (b2 - b1) / (m1 - m2);
    double y = m1 * x + b1;

    Point point = new Point();
    point.setLocation(x, y);
    return Optional.of(point);
}
```

现在让我们选择一些值并测试平行线和非平行线的方法。

例如，让我们以x轴(y = 0)作为第一条线，以y = x - 1定义的线作为第二条线。

对于第二条线，斜率m等于1，即45度，y截距等于-1，即该线在点(0,-1)处截取y轴。

直观上很明显，第二条线与x轴的交点一定是(1,0)：

![](/assets/images/2025/algorithms/javaintersectionoftwolines02.png)

我们来检查一下。

首先，让我们确保存在一个点，因为线不是平行的，然后检查x和y的值：

```java
@Test
public void givenNotParallelLines_whenCalculatePoint_thenPresent() {
    double m1 = 0;
    double b1 = 0;
    double m2 = 1;
    double b2 = -1;

    Optional<Point> point = service.calculateIntersectionPoint(m1, b1, m2, b2);

    assertTrue(point.isPresent());
    assertEquals(point.get().getX(), 1, 0.001);
    assertEquals(point.get().getY(), 0, 0.001);
}
```

最后，让我们取两条平行线并确保返回值为空：

![](/assets/images/2025/algorithms/javaintersectionoftwolines03.png)

```java
@Test
public void givenParallelLines_whenCalculatePoint_thenEmpty() {
    double m1 = 1;
    double b1 = 0;
    double m2 = 1;
    double b2 = -1;

    Optional<Point> point = service.calculateIntersectionPoint(m1, b1, m2, b2);

    assertFalse(point.isPresent());
}
```

## 4. 总结

在本教程中，我们展示了如何计算两条线的交点。