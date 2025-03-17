---
layout: post
title:  在Java中将十六进制字符串转换为整数
category: java
copyright: java
excerpt: Java Hex
---

## 1. 简介

将[十六进制(Hex)](https://www.baeldung.com/java-convert-hex-to-ascii)字符串转换为整数是编程过程中的常见任务，特别是在处理使用十六进制表示法的数据类型时。

**在本教程中，我们将深入研究在Java中将十六进制字符串转换为int的各种方法**。

## 2. 理解十六进制表示法

十六进制采用16进制，因此每个数字可以取从0到9，后跟(A)到(F)的16个可能值：

![](/assets/images/2025/java/javaconverthexstringtointeger01.png)

**还要注意，在大多数情况下，十六进制字符串以“0x”开头以表示其基数**。

## 3. 使用Integer.parseInt()

在Java中将十六进制字符串转换为整数的最简单方法是通过[Integer.parseInt()](https://www.baeldung.com/java-scanner-integer)方法，该方法将字符串转换为整数，并假设其写入的基数。对于我们来说，基数是16：

```java
@Test
public void givenValidHexString_whenUsingParseInt_thenExpectCorrectDecimalValue() {
    String hexString = "0x00FF00";
    int expectedDecimalValue = 65280;

    int decimalValue = Integer.parseInt(hexString.substring(2), 16);

    assertEquals(expectedDecimalValue, decimalValue);
}
```

在上面的代码中，使用Integer.parseInt将十六进制字符串“0x00FF00”转换为其对应的十进制值65280，并且测试断言结果与预期的十进制值匹配。**请注意，我们使用substring(2)方法从hexString中删除“ox”部分**。

## 4. 使用BigInteger

为了在处理非常大或无符号的十六进制值时获得更大的灵活性，我们可以考虑使用[BigInteger](https://www.baeldung.com/java-bigdecimal-biginteger)。它可以对任意精度的整数进行操作，因此可以在无数种情况下使用。

下面介绍如何将十六进制字符串转换为BigInteger，然后提取整数值：

```java
@Test
public void givenValidHexString_whenUsingBigInteger_thenExpectCorrectDecimalValue() {
    String hexString = "0x00FF00";
    int expectedDecimalValue = 65280;

    BigInteger bigIntegerValue = new BigInteger(hexString.substring(2), 16);
    int decimalValue = bigIntegerValue.intValue();
    assertEquals(expectedDecimalValue, decimalValue);
}
```

## 5. 使用Integer.decode()

将十六进制字符串转换为整数的另一种方法是使用[Integer.decode()](https://www.baeldung.com/java-encapsulation-convert-string-to-int)方法，此方法既处理十六进制字符串，也处理十进制字符串。

在这里，我们使用Integer.decode()而不声明进制，因为它是由字符串本身确定的：

```java
@Test
public void givenValidHexString_whenUsingIntegerDecode_thenExpectCorrectDecimalValue() {
    String hexString = "0x00FF00";
    int expectedDecimalValue = 65280;

    int decimalValue = Integer.decode(hexString);

    assertEquals(expectedDecimalValue, decimalValue);
}
```

**因为Integer.decode()方法可以处理字符串中的“0x”前缀，所以我们不需要像以前的方法那样使用substring(2)手动将其删除**。

## 6. 总结

总之，我们讨论了十六进制表示的意义，并深入研究了三种不同的方法：Integer.parseInt()用于直接转换，BigInteger用于处理大值或无符号值，Integer.decode()用于灵活处理十六进制和十进制字符串，包括“0x”前缀。