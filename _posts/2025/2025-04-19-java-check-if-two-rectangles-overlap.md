---
layout: post
title:  在Java中检查两个矩形是否重叠
category: algorithms
copyright: algorithms
excerpt: 矩形
---

## 1. 概述

在本快速教程中，我们将学习解决检查两个给定矩形是否重叠的算法问题。

我们将首先查看问题定义，然后逐步建立解决方案。

最后，我们将用Java实现它。

## 2. 问题定义

假设我们有两个给定的矩形r1和r2，我们需要检查r1和r2之间是否至少有一个公共点，如果是，则仅表示这两个矩形重叠。

让我们看一些例子：

![](/assets/images/2025/algorithms/javacheckiftworectanglesoverlap01.png)

如果我们注意到最后一种情况，矩形r1和r2没有相交的边界。尽管如此，它们仍然是重叠的矩形，因为r1中的每个点也是r2中的一个点。

## 3. 初始设置

为了解决这个问题，我们首先应该以编程方式定义一个矩形，**矩形可以很容易地通过其左下角和右上角坐标来表示**：

```java
public class Rectangle {
    private Point bottomLeft;
    private Point topRight;

    // constructor, getters and setters

    boolean isOverlapping(Rectangle other) {
        // ...
    }
}
```

其中Point是表示二维空间中点(x,y)的类：

```java
public class Point {
    private int x;
    private int y;

    // constructor, getters and setters
}
```

我们稍后会在Rectangle类中定义.isOverlapping()方法来检查它是否与比较的矩形重叠。

## 4. 解决方案

如果以下任一条件为true，则给定的两个矩形将不会重叠：

1. 两个矩形中的一个位于另一个矩形的顶边上方
2. 两个矩形中的一个位于另一个矩形左边缘的左侧

![](/assets/images/2025/algorithms/javacheckiftworectanglesoverlap02.png)

对于所有其他情况，两个矩形将相互重叠。为了验证这一点，我们可以举几个例子。

## 5. Java实现

现在我们了解了解决方案，让我们实现我们的.isOverlapping()方法：

```java
public boolean isOverlapping(Rectangle comparedRectangle) {
    if (this.topRight.getY() < comparedRectangle.bottomLeft.getY() || this.bottomLeft.getY() > comparedRectangle.topRight.getY()) {
        return false;
    }
    if (this.topRight.getX() < comparedRectangle.bottomLeft.getX() || this.bottomLeft.getX() > comparedRectangle.topRight.getX()) {
        return false;
    }
    return true;
}
```

如果其中一个矩形位于另一个矩形的上方或左侧，则Rectangle类中的.isOverlapping()方法返回false，否则返回true。

为了确定一个矩形是否位于另一个矩形的上方，我们比较它们的y坐标。同样，我们比较x坐标来检查一个矩形是否位于另一个矩形的左侧。

### 5.1 不考虑边界的比较

在问题描述中，我们没有定义是否应该考虑矩形的边界。.isOverlapping()方法将矩形定义为闭[区间](https://en.wikipedia.org/wiki/Interval_(mathematics))，因此，两个相切的矩形至少共享一个边界点，因此被认为是重叠的。或者，我们可以定义一个新的解决方案来忽略边界，并将矩形视为开区间：

```java
public boolean isOverlappingWithoutBorders(Rectangle comparedRectangle) {
    if (this.topRight.getY() <= comparedRectangle.bottomLeft.getY() || this.bottomLeft.getY() >= comparedRectangle.topRight.getY()) {
        return false;
    }

    if (this.topRight.getX() <= comparedRectangle.bottomLeft.getX() || this.bottomLeft.getX() >= comparedRectangle.topRight.getX()) {
        return false;
    }
    return true;
}
```

我们可以轻松测试这两种方法是否按预期运行：

```java
Rectangle rectangle1 = new Rectangle(new Point(0, 0), new Point(5, 14));
Rectangle rectangle2 = new Rectangle(new Point(5, 0), new Point(17, 14));
assertTrue(rectangle1.isOverlapping(rectangle2));
assertFalse(rectangle1.isOverlappingWithoutBorders(rectangle2));
```

两种方法均按预期工作。

## 6. 总结

在这篇短文中，我们学习了如何解决一个算法问题：判断两个给定的矩形是否有边界重叠，该算法可用作两个矩形物体的碰撞检测策略。