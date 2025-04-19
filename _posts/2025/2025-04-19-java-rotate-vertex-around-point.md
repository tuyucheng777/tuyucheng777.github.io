---
layout: post
title:  在Java中围绕某个点旋转顶点
category: algorithms
copyright: algorithms
excerpt: 点
---

## 1. 概述

在进行计算机图形和游戏开发时，围绕某个点旋转顶点的能力是一项基本技能。

在本快速教程中，我们将探索在Java中围绕某个点旋转顶点的不同方法。

## 2. 理解问题陈述

假设我们在二维平面上有两个点：点A的坐标为(x1,y1)，点B的坐标为(x2, y2)。我们希望将点A绕点B旋转一定角度，**如果旋转角度为正，则旋转方向为逆时针；如果旋转角度为负，则旋转方向为顺时针**：

![](/assets/images/2025/algorithms/javarotatevertexaroundpoint01.png)

在上图中，点A'是执行旋转后的新点。从A到A'的旋转是逆时针的，表示旋转角度为正45度。

## 3. 使用原点作为旋转点

**在这个方法中，我们首先将顶点和旋转点平移到原点。平移完成后，我们将围绕原点旋转所需的角度。旋转完成后，我们再将它们平移回原始位置**。

### 3.1 绕原点旋转点P

首先，让我们了解如何绕原点旋转点P。对于旋转，我们将使用涉及[三角函数](https://www.baeldung.com/java-lang-math#trigonometric)的公式。计算点P(x,y)绕原点(0,0)逆时针旋转后新坐标的公式为：

```java
rotatedXPoint = x * cos(angle) - y * sin(angle)
rotatedYPoint = x * sin(angle) + x * cos(angle)
```

rotatedXPoint和rotatedYPoint表示点P旋转后的新坐标，如果我们需要顺时针旋转该点，则需要使用负的旋转角度。

### 3.2 绕给定点旋转

我们将通过从顶点的x坐标中减去旋转点的x坐标，以及从顶点的y坐标中减去旋转点的y坐标来将旋转点移动到原点。

这些平移后的坐标表示相对于新原点的顶点位置，之后，我们将按照前面描述的方式进行旋转，并通过重新添加x和y坐标来实现反向平移。

让我们使用这种方法围绕一个点旋转一个顶点：

```java
public Point2D.Double usingOriginAsRotationPoint(Point2D.Double vertex, Point2D.Double rotationPoint, double angle) {
    double translatedToOriginX = vertex.x - rotationPoint.x;
    double translatedToOriginY = vertex.y - rotationPoint.y;

    double rotatedX = translatedToOriginX * Math.cos(angle) - translatedToOriginY * Math.sin(angle);
    double rotatedY = translatedToOriginX * Math.sin(angle) + translatedToOriginY * Math.cos(angle);

    double reverseTranslatedX = rotatedX + rotationPoint.x;
    double reverseTranslatedY = rotatedY + rotationPoint.y;

    return new Point2D.Double(reverseTranslatedX, reverseTranslatedY);
}
```

让我们测试一下这种旋转顶点的方法：

```java
void givenRotationPoint_whenUseOrigin_thenRotateVertex() {
    Point2D.Double vertex = new Point2D.Double(2.0, 2.0);
    Point2D.Double rotationPoint = new Point2D.Double(0.0, 1.0);
    double angle = Math.toRadians(45.0);
    Point2D.Double rotatedVertex = VertexRotation.usingOriginAsRotationPoint(vertex, rotationPoint, angle);

    assertEquals(0.707, rotatedVertex.getX(), 0.001);
    assertEquals(3.121, rotatedVertex.getY(), 0.001);
}
```

## 4. 使用AffineTransform类

在这种方法中，我们将利用[AffineTransform](https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/java/awt/geom/AffineTransform.html)类，该类用于执行平移、缩放、旋转和翻转等几何变换。

**我们首先使用getRotateInstance()根据指定的角度和旋转点创建一个旋转变换矩阵，随后，我们将使用transform()方法将变换应用于顶点并执行旋转**。让我们来看看这种方法：

```java
public Point2D.Double usingAffineTransform(Point2D.Double vertex, Point2D.Double rotationPoint, double angle) {
    AffineTransform affineTransform = AffineTransform.getRotateInstance(angle, rotationPoint.x, rotationPoint.y);
    Point2D.Double rotatedVertex = new Point2D.Double();
    affineTransform.transform(vertex, rotatedVertex);
    return rotatedVertex;
}
```

让我们测试一下这种旋转顶点的方法：

```java
void givenRotationPoint_whenUseAffineTransform_thenRotateVertex() {
    Point2D.Double vertex = new Point2D.Double(2.0, 2.0);
    Point2D.Double rotationPoint = new Point2D.Double(0.0, 1.0);
    double angle = Math.toRadians(45.0);
    Point2D.Double rotatedVertex = VertexRotation.usingAffineTransform(vertex, rotationPoint, angle);

    assertEquals(0.707, rotatedVertex.getX(), 0.001);
    assertEquals(3.121, rotatedVertex.getY(), 0.001);
}
```

## 5. 总结

在本教程中，我们讨论了围绕某个点旋转顶点的方法。