---
layout: post
title:  使用Java创建BMI计算器
category: algorithms
copyright: algorithms
excerpt: BMI
---

## 1. 概述

在本教程中，我们将使用Java创建一个BMI计算器。

在开始实现之前，首先我们来了解一下BMI的概念。

## 2. 什么是BMI？

BMI是“身体质量指数”的缩写，是根据个人身高和体重得出的数值。

借助BMI，我们可以了解一个人的体重是否健康。

我们先来看看BMI的计算公式：

**BMI= 体重(公斤) / (身高(米) * 身高(米))**

根据BMI范围，一个人可分为体重过轻、正常、超重或肥胖：

|  体重指数范围   | 类别 |
|:---------:|:--:|
|   <18.5   | 偏轻 |
| 18.5 - 25 | 普通 |
|  25 - 30  | 超重 |
|    >30    | 肥胖 |

例如，让我们计算体重为100kg(公斤)、身高为1.524m(米)的个人的BMI。

BMI = 100 / (1.524 * 1.524)

BMI = 43.056

**由于BMI大于30，因此该人被归类为“超重”**。

## 3. Java程序计算BMI

该Java程序包含计算BMI的公式和简单的if– else语句，利用公式和上表，我们可以找出个体所属的类别：

```java
static String calculateBMI(double weight, double height) {

    double bmi = weight / (height * height);

    if (bmi < 18.5) {
        return "Underweight";
    }
    else if (bmi < 25) {
        return "Normal";
    }
    else if (bmi < 30) {
        return "Overweight";
    }
    else {
        return "Obese";
    }
}
```

## 4. 测试

让我们通过提供“肥胖”个体的身高和体重来测试代码：

```java
@Test
public void whenBMIIsGreaterThanThirty_thenObese() {
    double weight = 50;
    double height = 1.524;
    String actual = BMICalculator.calculateBMI(weight, height);
    String expected = "Obese";

    assertThat(actual).isEqualTo(expected);
}
```

运行测试后，我们可以看到实际结果与预期相同。

## 5. 总结

在本文中，我们学习了如何使用Java创建BMI计算器。