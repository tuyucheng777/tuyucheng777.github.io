---
layout: post
title:  Java中的基本计算器
category: algorithms
copyright: algorithms
excerpt: 计算器
---

## 1. 概述

在本教程中，我们将用Java实现一个支持加法、减法、乘法和除法运算的基本计算器。

我们还将运算符和操作数作为输入并基于它们进行计算。

## 2. 基本设置

首先，让我们展示一些有关计算器的信息：

```java
System.out.println("---------------------------------- \n" +
    "Welcome to Basic Calculator \n" +
    "----------------------------------");
System.out.println("Following operations are supported : \n" +
    "1. Addition (+) \n" +
    "2. Subtraction (-) \n" +
    "3. Multiplication (*) \n" +
    "4. Division (/) \n");
```

现在，让我们使用[java.util.Scanner](https://www.baeldung.com/java-scanner)来获取用户输入：

```java
Scanner scanner = new Scanner(System.in);

System.out.println("Enter an operator: (+ OR - OR * OR /) ");
char operation = scanner.next().charAt(0);

System.out.println("Enter the first number: ");
double num1 = scanner.nextDouble();

System.out.println("Enter the second number: ");
double num2 = scanner.nextDouble();
```

当我们将输入获取到系统中时，我们需要对其进行验证。例如，如果运算符不是+、-、\*或/，那么我们的计算器应该会识别出错误的输入。同样，如果我们在除法运算中输入第二个数字0，结果也不会理想。

那么，让我们实现这些验证。

首先我们来关注一下运算符无效的情况：

```java
if (!(operation == '+' || operation == '-' || operation == '*' || operation == '/')) {
    System.err.println("Invalid Operator. Please use only + or - or * or /");
}
```

然后我们可以显示无效操作的错误：

```java
if (operation == '/' && num2 == 0.0) {
    System.err.println("The second number cannot be zero for division operation.");
}
```

首先验证用户输入，之后，计算结果将显示为：

<数字1\> <操作\> <数字2\> = <结果\>

## 3. 处理计算

首先，我们可以使用[if-else](https://www.baeldung.com/java-if-else)结构来处理计算：

```java
if (operation == '+') {
    System.out.println(num1 + " + " + num2 + " = " + (num1 + num2));
} else if (operation == '-') {
    System.out.println(num1 + " - " + num2 + " = " + (num1 - num2));
} else if (operation == '*') {
    System.out.println(num1 + " x " + num2 + " = " + (num1 * num2));
} else if (operation == '/') {
    System.out.println(num1 + " / " + num2 + " = " + (num1 / num2));
} else {
    System.err.println("Invalid Operator Specified.");
}
```

类似地，我们可以使用Java [switch](https://www.baeldung.com/java-switch)语句：

```java
switch (operation) {
    case '+':
        System.out.println(num1 + " + " + num2 + " = " + (num1 + num2));
        break;
    case '-':
        System.out.println(num1 + " - " + num2 + " = " + (num1 - num2));
        break;
    case '*':
        System.out.println(num1 + " x " + num2 + " = " + (num1 * num2));
        break;
    case '/':
        System.out.println(num1 + " / " + num2 + " = " + (num1 / num2));
        break;
    default:
        System.err.println("Invalid Operator Specified.");
        break;
}
```

我们可以使用变量来存储计算结果，这样，就可以在最后打印出来。在这种情况下，System.out.println只会使用一次。

另外，计算的最大范围是2147483647。因此，如果超出该范围，int数据类型就会溢出。因此，应该将其存储在[更大数据类型](https://www.baeldung.com/java-primitives)的变量中，例如double数据类型。

## 4. 总结

在本教程中，我们用Java实现了一个基本计算器，使用了两种不同的结构，我们还确保在进一步处理输入之前对其进行了验证。