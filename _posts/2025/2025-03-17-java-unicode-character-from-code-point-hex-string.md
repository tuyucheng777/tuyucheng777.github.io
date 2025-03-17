---
layout: post
title:  根据代码点十六进制字符串创建Unicode字符
category: java
copyright: java
excerpt: Java Char
---

## 1. 概述

Java对[Unicode](https://www.baeldung.com/java-char-encoding#unicode)的支持使得处理来自不同语言和脚本的字符变得非常简单。

在本教程中，我们将探索和学习如何从Java中的代码点获取Unicode字符。

## 2.   问题介绍

Java的Unicode支持使我们能够快速构建国际化应用程序，让我们看几个例子：

```java
static final String U_CHECK = "✅"; // U+2705
static final String U_STRONG = "强"; // U+5F3A
```

在上面的例子中，[复选标记](https://www.compart.com/en/unicode/U+2705)“✅”和“[强](https://www.compart.com/en/unicode/U+5F3A)”(中文“Strong”)都是Unicode字符。

我们知道，如果我们的字符串遵循转义的“u”和十六进制数的模式，Unicode字符可以在Java中正确表示，例如：

```java
String check = "\u2705";
assertEquals(U_CHECK, check);

String strong = "\u5F3A";
assertEquals(U_STRONG, strong);
```

在一些场景中，我们收到“\u”后面的十六进制数，需要得到对应的Unicode字符。例如，当我们收到字符串格式的数字“2705”时，应该产生复选标记“✅”。

我们首先想到的办法可能是将“\\\u”和数字连接起来。然而，这样做不行：

```java
String check = "\\u" + "2705";
assertEquals("\\u2705", check);

String strong = "\\u" + "5F3A";
assertEquals("\\u5F3A", strong);
```

测试显示，**将“\\\u”和数字(例如“2705”)拼接起来，会生成字符串“\\\u2705”**，而不是复选标记“✅”。

接下来，让我们探索如何将给定的数字转换为Unicode字符串。

## 3. 理解“\u”后的十六进制数

**Unicode为每个字符分配一个唯一的[代码点](https://www.baeldung.com/java-char-encoding#3-code-point)**，从而提供一种跨不同语言和文字表示文本的通用方法。代码点是Unicode标准中唯一标识字符的数值。

要在Java中创建Unicode字符，我们需要了解所需字符的代码点。一旦我们有了代码点，我们就可以使用Java的char数据类型和转义序列'\u'来表示Unicode字符。

在“\uxxxx”表示法中，**“xxxx”是字符在十六进制表示中的代码点**。例如，“A”的十六进制[ASCII](https://www.baeldung.com/java-character-ascii-value)码为41(十进制：65)。因此，如果我们使用Unicode表示法“\u0041”，则可以得到字符串“A”：

```java
assertEquals("A", "\u0041");
```

那么接下来我们看看如何从“\u”后面的十六进制数中获取所需的Unicode字符。

## 4. 使用Character.toChars()方法

现在我们明白了“\u”后面的十六进制数表示什么，当我们收到“2705”时，**它就是字符代码点的十六进制表示**。

如果我们获取了代码点整数，**Character.toChars(int codePoint)方法可以返回保存代码点Unicode表示的char数组**。最后，我们可以调用[String.valueOf()](https://www.baeldung.com/string/value-of)来获取目标字符串：

```text
Given "2705"
 |_ Hex(codePoint) = 0x2705
     |_ codePoint = 9989 (decimal)
         |_ char[] chars = Character.toChars(9989) 
            |_ String.valueOf(chars)
               |_"✅"
```

我们可以看到，为了获得目标字符，我们必须首先找到代码点。

**可以通过使用[Integer.parseInt()](https://www.baeldung.com/java-convert-string-to-int-or-integer#Integer)方法以十六进制(基数为16)基数解析提供的字符串来获取代码点整数**。 

接下来，让我们把所有内容放在一起：

```java
int codePoint = Integer.parseInt("2705", 16); // Decimal int: 9989
char[] checkChar = Character.toChars(codePoint);
String check = String.valueOf(checkChar);
assertEquals(U_CHECK, check);

codePoint = Integer.parseInt("5F3A", 16); // Decimal int: 24378
char[] strongChar = Character.toChars(codePoint);
String strong = String.valueOf(strongChar);
assertEquals(U_STRONG, strong);
```

值得注意的是，**如果我们使用Java 11或更高版本，我们可以使用Character.toString()方法直接从代码点整数获取字符串**，例如：

```java
// ForJava11 and later versions:
assertEquals(U_STRONG, Character.toString(codePoint));
```

当然，我们可以将上面的实现包装在一个方法中：

```java
String stringFromCodePointHex(String codePointHex) {
    int codePoint = Integer.parseInt(codePointHex, 16);
    // ForJava11 and later versions: return Character.toString(codePoint)
    char[] chars = Character.toChars(codePoint);
    return String.valueOf(chars);
}
```

最后，让我们测试该方法以确保它产生预期的结果：

```java
assertEquals("A", stringFromCodePointHex("0041"));
assertEquals(U_CHECK, stringFromCodePointHex("2705"));
assertEquals(U_STRONG, stringFromCodePointHex("5F3A"));
```

## 5. 总结

在本文中，我们首先了解了“\uxxxx”符号中“xxxx”的含义，然后探讨了如何从给定代码点的十六进制表示中获取目标Unicode字符串。