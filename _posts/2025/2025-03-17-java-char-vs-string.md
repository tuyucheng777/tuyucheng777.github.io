---
layout: post
title:  Java的char和String之间的区别
category: java
copyright: java
excerpt: Java Char
---

## 1. 概述

在Java中，char和String类型都处理字符，但它们具有不同的属性和用途。

在本教程中，我们将探讨Java编程中char和String之间的区别。

## 2. char和String

char表示Java中的单个16位Unicode字符，例如字母、数字或符号。因此，char类型的范围是'\u0000'到'\uffff'(包含)。我们也可以说**char是一个从0到65535(2<sup>16</sup>-1)的无符号16位整数**。

然而String是Java中必不可少的类，**字符串由单个或者多个字符组成**。

我们看到，char和String都和字符有关，接下来我们来仔细看看这两种数据类型，并讨论一下它们的区别。

为了简单起见，我们将在示例中使用单元测试断言查看结果。

## 3. char是原始类型，而String是类

Java中char和String的第一个区别是char是[原始类型](https://www.baeldung.com/java-primitives)，而String是类。换句话说，它是[引用类型](https://www.baeldung.com/java-primitives-vs-objects#java-type-system)。

因此，char不需要对象的开销。因此，**它在性能和内存占用方面具有优势**。但是，原始类型的功能有限，因为**它们没有任何成员方法**。

此外，[Java泛型](https://www.baeldung.com/java-generics)也不支持原始类型。

## 4. char只能表示一个字符，但字符串是字符序列

现在，让我们看一下String类的签名：

```java
public final class String implements Serializable, Comparable<String>, CharSequence, Constable, ConstantDesc{ ... }
```

如代码所示，String类实现了[CharSequence接口](https://www.baeldung.com/java-char-sequence-string#charsequence)。也就是说，**String对象是一个字符序列**。

但是**char变量只能携带一个字符**。

让我们通过一个例子来理解：

```java
char h = 'h';
char e = 'e';
char l = 'l';
char o = 'o';

String hello = "hello";
assertEquals(h, hello.charAt(0));
assertEquals(e, hello.charAt(1));
assertEquals(l, hello.charAt(2));
assertEquals(l, hello.charAt(3));
assertEquals(o, hello.charAt(4));
```

我们可以看到，上面的测试中有四个char变量(h、e、l、o)，字符串“hello”由五个字符组成。我们可以通过将每个char变量与CharSequence的charAt()方法的结果进行比较来验证这一点。

另外，我们可以将字符串视为字符数组。**String类为我们提供了toCharArray()方法，用于将字符串拆分为数组中的字符**：

```java
char[] chars = new char[] { h, e, l, l, o };
char[] charsFromString = hello.toCharArray();
assertArrayEquals(chars, charsFromString);
```

## 5. 加法(+)运算符

当我们将加法运算符应用于两个字符串时，它会将它们拼接起来：

```java
String h = "H";
String i = "i";
assertEquals("Hi", h + i);
```

但是，我们应该注意，**当我们执行char + char时，结果是一个int，该值是这两个字符的总和**。一个例子可以直接解决这个问题：

```java
char h = 'H'; // the value is 72
char i = 'i'; // the value is 105
assertEquals(177, h + i);
assertInstanceOf(Integer.class, h + i);
```

此外，当我们“相加”一个char和一个String时，char将被转换为字符串并与给定的字符串拼接。例如，**我们经常将一个空字符串“相加”到char中，以将char变量转换为单字符String**：

```java
char c = 'C'; 
assertEquals("C", "" + c);
```

但是，**当我们用字符串“相加”多个字符变量时，需要注意字符串的位置**。接下来我们看另一个例子：

```java
char h = 'H'; // the value is 72
char i = 'i'; // the value is 105
assertEquals("Hi", "" + h + i); //(1)
assertEquals("Hi", h + "" + i); //(2)
assertEquals("177", h + i + "");//(3)
```

在这个例子中，前两个加法运算产生字符串“Hi”，但最后一个加法运算的结果是字符串“177”，这是因为加法表达式是从左到右执行的。

前两个表达式中，无论是“” + h还是h + “”，Java都会将char变量h转换为字符串，并将其与空字符串拼接起来。然而，**在最后一个表达式中，正如我们之前所了解的，h + i会产生一个int结果(72 + 105 = 177)**。然后，将int结果转换为字符串，以便与空字符串拼接起来。

## 6. 总结

在本文中，我们讨论了Java中char和String之间的区别，让我们在这里列出它们作为摘要：

- char是原始类型，但String是引用类型。
- char仅代表一个字符，而String可以包含多个字符。
- char + char = 两个字符的总和为int，String + String拼接两个字符串。