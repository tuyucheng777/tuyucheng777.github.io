---
layout: post
title:  用Java计算圆的面积
category: algorithms
copyright: algorithms
excerpt: 圆
---

## 1. 概述

在本快速教程中，我们将说明如何在Java中计算圆的面积。

我们将使用众所周知的数学公式：r^2 * PI。

## 2. 圆面积计算方法

让我们首先创建一个执行计算的方法：

```java
private void calculateArea(double radius) {
    double area = radius * radius * Math.PI;
    System.out.println("The area of the circle [radius = " + radius + "]: " + area);
}
```

### 2.1 将半径作为命令行参数传递

现在我们可以读取命令行参数并计算面积：

```java
double radius = Double.parseDouble(args[0]);
calculateArea(radius);
```

当我们编译并运行该程序时：

```text
java CircleArea.java
javac CircleArea 7
```

我们将得到以下输出：

```text
The area of the circle [radius = 7.0]: 153.93804002589985
```

### 2.2 从键盘读取半径

获取半径值的另一种方法是使用来自用户的输入数据：

```java
Scanner sc = new Scanner(System.in);
System.out.println("Please enter radius value: ");
double radius = sc.nextDouble();
calculateArea(radius);
```

输出与前面的示例相同。

## 3. Circle类

除了像第2节中看到的那样调用方法来计算面积之外，我们还可以创建一个表示圆的类：

```java
public class Circle {

    private double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    // standard getter and setter

    private double calculateArea() {
        return radius * radius * Math.PI;
    }

    public String toString() {
        return "The area of the circle [radius = " + radius + "]: " + calculateArea();
    }
}
```

我们需要注意几点，首先，我们没有将面积保存为变量，因为它直接依赖于半径，因此我们可以轻松计算。其次，计算面积的方法是私有的，因为我们在toString()方法中使用它。**toString()方法不应该调用类中的任何公共方法，因为这些方法可能会被重写，并且其行为可能与预期不同**。

现在我们可以实例化我们的Circle对象：

```java
Circle circle = new Circle(7);
```

当然，输出将与以前相同。

## 4. 总结

在这篇简短的文章中，我们展示了使用Java计算圆面积的不同方法。