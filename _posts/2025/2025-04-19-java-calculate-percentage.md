---
layout: post
title:  在Java中计算百分比
category: algorithms
copyright: algorithms
excerpt: 百分比
---

## 1. 简介

在本快速教程中，我们将实现一个CLI程序来用Java计算百分比。

但首先，让我们定义如何从数学上计算百分比。

## 2. 数学公式

在数学中，百分比是用100的分数表示的数字或比率，它通常用百分号“%”表示。

假设有一名学生获得了x分，总分为y分，计算该学生成绩百分比的公式如下：

> 百分比 = (x / y) * 100

## 3. 一个简单的Java程序

现在我们已经清楚了如何用数学方法计算百分比，让我们用Java构建一个程序来计算它：

```java
public class PercentageCalculator {

    public double calculatePercentage(double obtained, double total) {
        return obtained * 100 / total;
    }

    public static void main(String[] args) {
        PercentageCalculator pc = new PercentageCalculator();
        Scanner in = new Scanner(System.in);
        System.out.println("Enter obtained marks:");
        double obtained = in.nextDouble();
        System.out.println("Enter total marks:");
        double total = in.nextDouble();
        System.out.println("Percentage obtained: " + pc.calculatePercentage(obtained, total));
    }
}
```

该程序从CLI获取学生的分数(获得分数和总分数)，然后调用calculatePercentage()方法计算其百分比。

这里我们选择double作为输入和输出的数据类型，因为它可以存储精度高达16位的十进制数。因此，它应该足以满足我们的用例。

让我们运行这个程序并看看结果：

```text
Enter obtained marks:
87
Enter total marks:
100
Percentage obtained: 87.0

Process finished with exit code 0
```

## 4. 使用BigDecimal的版本

Java中的[BigDecimal](https://www.baeldung.com/java-bigdecimal-biginteger)是一个用于高精度十进制运算的类，它比原生的double和float类型有所改进，允许对小数位数和舍入进行精确控制，这使得它们对于精度至关重要的应用程序至关重要。

BigDecimal广泛应用于货币和税务计算等金融计算，在科学、工程和统计领域也至关重要，因为在这些领域，即使是微小的数字错误也可能导致不良后果。在传输数据的应用程序中，它们通常与[字符串](https://www.baeldung.com/java-string-to-bigdecimal)相互转换。

### 4.1 使用BigDecimal实现实用程序类

首先，让我们设置一个常量`ONE_HUNDRED`，这在计算中会很方便。现在，对于方法toPercentageOf，其思路很简单：我们想要找出特定值占总数的百分比。这是一个标准的计算，因此拥有一个简洁的方法是有意义的：

```java
public class BigDecimalPercentages {

    private static final BigDecimal ONE_HUNDRED = new BigDecimal("100");

    public BigDecimal toPercentageOf(BigDecimal value, BigDecimal total) {
        return value.divide(total, 4, RoundingMode.HALF_UP).multiply(ONE_HUNDRED);
    }

    public BigDecimal percentOf(BigDecimal percentage, BigDecimal total) {
        return percentage.multiply(total).divide(ONE_HUNDRED, 2, RoundingMode.HALF_UP);
    }
}
```

在toPercentageOf()方法中，我们通过将总数除以百分比来计算该值占总数的百分比，对于标准百分比计算，四舍五入到小数点后4位，然后乘以100。

另一种方法则相反，它计算出一个相当于传递的总数百分比的值。

但让我们稍微加点料，我们知道有些[方法](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/math/BigDecimal.html)可以更快地乘以或除以10的幂，例如movePointLeft()，它会将小数点向右移动。尽管如此，它的实现是为了防止结果出现负的标度值，这在某些情况下是可行的，但并非总是如此。例如，如果结果小于1，则除法将无法得到正确的结果。

另一方面，scaleByPowerOfTen()的功能更加丰富。它按十的幂对数字进行缩放，就像移动小数点一样，但不会因为数字过小而导致精度损失。如果我们输入一个负数，它将等同于除法。例如，使用-2相当于除以10，让我们看看这种方法的具体实现：

```java
import java.math.BigDecimal;
import java.math.RoundingMode;

public class FastBigDecimalPercentage {

    public static BigDecimal toPercentageOf(BigDecimal value, BigDecimal total) {
        return value.divide(total, 4, RoundingMode.HALF_UP).scaleByPowerOfTen(2);
    }

    public static BigDecimal percentOf(BigDecimal percentage, BigDecimal total) {
        return percentage.multiply(total).scaleByPowerOfTen(-2);
    }
}
```

### 4.2 使用Lombok的@Extensionmethod提高可用性

FastBigDecimalPercentage和BigDecimalPercentageCalculator类可用于计算BigDecimal百分比，但它们传统的Java语法需要更加优雅和流式。这时Lombok的@ExtensionMethod就派上用场了，它提供了一种增强代码可读性和可用性的方法：

```java
import lombok.experimental.ExtensionMethod;
import java.math.BigDecimal;
import java.util.Scanner;

@ExtensionMethod(FastBigDecimalPercentage.class)
public class BigDecimalPercentageCalculator {
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        System.out.println("Enter obtained marks:");
        BigDecimal obtained = new BigDecimal(in.nextDouble());
        System.out.println("Enter total marks:");
        BigDecimal total = new BigDecimal(in.nextDouble());

        System.out.println("Percentage obtained :" + obtained.toPercentageOf(total));
    }
}
```

通过应用Lombok的扩展，我们可以将标准方法转换为更直观的接口，我们可以像直接调用BigDecimal中声明的方法一样调用我们创建的方法。

## 5. 总结

在本文中，我们研究了如何用数学方法计算百分比，然后编写了一个Java CLI程序来计算它。