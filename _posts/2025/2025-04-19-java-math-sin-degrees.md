---
layout: post
title:  使用Math.sin和度数
category: algorithms
copyright: algorithms
excerpt: Sin
---

## 1. 简介

在这个简短的教程中，我们将研究如何使用Java的Math.sin()函数计算正弦值以及如何在度数和弧度之间转换角度值。

## 2. 弧度与角度

默认情况下，**Java数学库要求其三角函数值以弧度为单位**。

提醒一下，**弧度只是表达角度测量的另一种方式，其转换公式为**：

```java
double inRadians = inDegrees * PI / 180;
inDegrees = inRadians * 180 / PI;
```

Java使用toRadians和toDegrees使这变得简单：

```java
double inRadians = Math.toRadians(inDegrees);
double inDegrees = Math.toDegrees(inRadians);
```

每当我们使用Java的任何三角函数时，**我们应该首先考虑输入的单位是什么**。

## 3. 使用Math.sin

我们可以通过查看Math.sin方法来了解这一原则的实际作用，这是Java提供的众多方法之一：

```java
public static double sin(double a)
```

它相当于数学中的正弦函数，**其输入以弧度为单位**。因此，假设我们有一个已知角度的度数：

```java
double inDegrees = 30;
```

我们首先需要将其转换为弧度：

```java
double inRadians = Math.toRadians(inDegrees);
```

然后我们可以计算正弦值：

```java
double sine = Math.sin(inRadians);
```

但是，**如果我们知道它已经是弧度，那么我们就不需要进行转换**：

```java
@Test
public void givenAnAngleInDegrees_whenUsingToRadians_thenResultIsInRadians() {
    double angleInDegrees = 30;
    double sinForDegrees = Math.sin(Math.toRadians(angleInDegrees)); // 0.5

    double thirtyDegreesInRadians = 1/6 * Math.PI;
    double sinForRadians = Math.sin(thirtyDegreesInRadians); // 0.5

    assertTrue(sinForDegrees == sinForRadians);
}
```

由于twentyDegreesInRadians已经是弧度了，我们不需要先转换它就可以得到相同的结果。

## 4. 总结

在这篇简短的文章中，我们回顾了弧度和度数，然后看到了如何使用Math.sin处理它们的示例。