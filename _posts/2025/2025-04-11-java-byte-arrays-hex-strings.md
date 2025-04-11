---
layout: post
title:  Java中字节数组和十六进制字符串之间的转换
category: algorithms
copyright: algorithms
excerpt: 字符串
---

## 1. 概述

在本教程中，我们将介绍将字节数组转换为十六进制字符串的不同方法，反之亦然。

## 2. 字节与十六进制之间的转换

首先我们来看一下字节和十六进制数的转换逻辑。

### 2.1 字节转十六进制

Java中的字节是8位有符号整数，因此，**我们需要将每个4位段分别转换为十六进制**，然后将它们拼接起来。转换后，我们将得到两个十六进制字符。

例如，我们可以将45以二进制写为0010 1101，其十六进制等价于“2d”：
```java
0010 = 2 (base 10) = 2 (base 16)
1101 = 13 (base 10) = d (base 16)

Therefore: 45 = 0010 1101 = 0x2d
```

让我们用Java实现这个简单的逻辑：
```java
public String byteToHex(byte num) {
    char[] hexDigits = new char[2];
    hexDigits[0] = Character.forDigit((num >> 4) & 0xF, 16);
    hexDigits[1] = Character.forDigit((num & 0xF), 16);
    return new String(hexDigits);
}
```

现在，让我们通过分析每个操作来理解上面的代码。首先，我们创建一个长度为2的char数组来存储输出：
```java
char[] hexDigits = new char[2];
```

接下来，我们通过右移4位来隔离高阶位；然后，我们应用掩码来隔离低阶4位。需要掩码是因为负数在内部表示为正数的[二进制补码](https://www.baeldung.com/cs/two-complement)：
```java
hexDigits[0] = Character.forDigit((num >> 4) & 0xF, 16);
```

然后我们将剩下的4位转换为十六进制：
```java
hexDigits[1] = Character.forDigit((num & 0xF), 16);
```

最后，我们从字符数组创建一个String对象。然后，将此对象作为转换后的十六进制数组返回。

现在，让我们理解一下这对于负字节-4是如何工作的：
```java
hexDigits[0]:
1111 1100 >> 4 = 1111 1111 1111 1111 1111 1111 1111 1111
1111 1111 1111 1111 1111 1111 1111 1111 & 0xF = 0000 0000 0000 0000 0000 0000 0000 1111 = 0xf

hexDigits[1]:
1111 1100 & 0xF = 0000 1100 = 0xc

Therefore: -4 (base 10) = 1111 1100 (base 2) = fc (base 16)
```

还值得注意的是，Character.forDigit()方法始终返回小写字符。

### 2.2 十六进制转字节

现在，让我们将十六进制数字转换为字节。众所周知，一个字节包含8位，因此，**我们需要两个十六进制数字来创建一个字节**。

首先，我们将分别将每个十六进制数字转换为二进制数字。

然后，我们需要拼接这两个四位段来获得等效的字节：
```java
Hexadecimal: 2d
2 = 0010 (base 2)
d = 1101 (base 2)

Therefore: 2d = 0010 1101 (base 2) = 45
```

现在，让我们用Java编写该操作：
```java
public byte hexToByte(String hexString) {
    int firstDigit = toDigit(hexString.charAt(0));
    int secondDigit = toDigit(hexString.charAt(1));
    return (byte) ((firstDigit << 4) + secondDigit);
}

private int toDigit(char hexChar) {
    int digit = Character.digit(hexChar, 16);
    if(digit == -1) {
        throw new IllegalArgumentException("Invalid Hexadecimal Character: "+ hexChar);
    }
    return digit;
}
```

让我们逐一了解一下这一点。

首先，我们将十六进制字符转换为整数：
```java
int firstDigit = toDigit(hexString.charAt(0));
int secondDigit = toDigit(hexString.charAt(1));
```

然后我们将最高有效位左移4位，因此，二进制表示法在最低有效位的4位上都有0。

然后，我们将最低有效数字相加到其中：
```java
return (byte) ((firstDigit << 4) + secondDigit);
```

现在，让我们仔细检查一下toDigit()方法。我们使用Character.digit()方法进行转换，**如果传递给此方法的字符值不是指定基数中的有效数字，则返回-1**。

我们验证返回值，如果传递了无效值，则会引发异常。

## 3. 字节数组和十六进制字符串之间的转换

至此，我们知道如何将字节转换为十六进制，让我们扩展此算法，并将字节数组转换为十六进制字符串。

### 3.1 字节数组转十六进制字符串

我们需要循环遍历数组并为每个字节生成十六进制对：
```java
public String encodeHexString(byte[] byteArray) {
    StringBuffer hexStringBuffer = new StringBuffer();
    for (int i = 0; i < byteArray.length; i++) {
        hexStringBuffer.append(byteToHex(byteArray[i]));
    }
    return hexStringBuffer.toString();
}
```

我们已经知道，输出始终是小写的。

### 3.2 十六进制字符串转字节数组

首先，我们需要检查十六进制字符串的长度是否为偶数，这是因为奇数长度的十六进制字符串会导致错误的字节表示。

现在，我们将遍历数组并将每个十六进制对转换为一个字节：
```java
public byte[] decodeHexString(String hexString) {
    if (hexString.length() % 2 == 1) {
        throw new IllegalArgumentException("Invalid hexadecimal String supplied.");
    }
    
    byte[] bytes = new byte[hexString.length() / 2];
    for (int i = 0; i < hexString.length(); i += 2) {
        bytes[i / 2] = hexToByte(hexString.substring(i, i + 2));
    }
    return bytes;
}
```

## 4. 使用BigInteger类

我们可以通过传递符号和字节数组来创建BigInteger类型的对象。

现在，我们可以借助String类中定义的静态方法format生成十六进制字符串：
```java
public String encodeUsingBigIntegerStringFormat(byte[] bytes) {
    BigInteger bigInteger = new BigInteger(1, bytes);
    return String.format("%0" + (bytes.length << 1) + "x", bigInteger);
}
```

提供的format将生成一个以零填充的小写十六进制字符串，我们还可以通过将“x”替换为“X”来生成大写字符串。

或者，我们可以使用BigInteger中的toString()方法，**使用toString()方法的细微差别在于输出不会用前导0填充**：
```java
public String encodeUsingBigIntegerToString(byte[] bytes) {
    BigInteger bigInteger = new BigInteger(1, bytes);
    return bigInteger.toString(16);
}
```

现在，让我们看一下十六进制字符串到字节数组的转换：
```java
public byte[] decodeUsingBigInteger(String hexString) {
    byte[] byteArray = new BigInteger(hexString, 16).toByteArray();
    if (byteArray[0] == 0) {
        byte[] output = new byte[byteArray.length - 1];
        System.arraycopy(byteArray, 1, output, 0, output.length);
        return output;
    }
    return byteArray;
}
```

**toByteArray()方法会产生一个额外的符号位**，我们编写了专门的代码来处理这个额外的位。

因此，在使用BigInteger类进行转换之前，我们应该了解这些细节。

## 5. 使用DataTypeConverter类

DataTypeConverter类随JAXB库提供，这是Java 8之前标准库的一部分；从Java 9开始，我们需要将java.xml.bind模块明确添加到运行时。

让我们看一下使用DataTypeConverter类的实现：
```java
public String encodeUsingDataTypeConverter(byte[] bytes) {
    return DatatypeConverter.printHexBinary(bytes);
}

public byte[] decodeUsingDataTypeConverter(String hexString) {
    return DatatypeConverter.parseHexBinary(hexString);
}
```

如上所示，使用DataTypeConverter类非常方便，**printHexBinary()方法的输出始终为大写**，该类提供了一组用于数据类型转换的打印和解析方法。

在选择这种方法之前，我们需要确保该类在运行时可用。

## 6. 使用Apache的Commons-Codec库

我们可以使用Apache commons-codec库提供的[Hex](https://commons.apache.org/proper/commons-codec/apidocs/org/apache/commons/codec/binary/Hex.html)类：
```java
public String encodeUsingApacheCommons(byte[] bytes) throws DecoderException {
    return Hex.encodeHexString(bytes);
}

public byte[] decodeUsingApacheCommons(String hexString) throws DecoderException {
    return Hex.decodeHex(hexString);
}
```

**encodeHexString的输出始终是小写的**。

## 7. 使用Google Guava库

让我们看看如何使用[BaseEncoding](https://google.github.io/guava/releases/16.0/api/docs/com/google/common/io/BaseEncoding.html)类将字节数组编码和解码为十六进制字符串：
```java
public String encodeUsingGuava(byte[] bytes) {
    return BaseEncoding.base16().encode(bytes);
}

public byte[] decodeUsingGuava(String hexString) {
    return BaseEncoding.base16().decode(hexString.toUpperCase());
}
```

**BaseEncoding默认使用大写字母进行编码和解码**，如果需要使用小写字母，则需要使用静态方法[lowercase](https://google.github.io/guava/releases/16.0/api/docs/com/google/common/io/BaseEncoding.html#lowerCase())创建一个新的编码实例。

## 8. 使用Java 17 HexFormat

除了前面讨论的在字节数组和十六进制字符串之间转换的方法之外，Java 17还通过java.util.HexFormat类引入了一种方便的替代方法。[HexFormat](https://www.baeldung.com/java-hexformat)提供了一种执行十六进制编码和解码操作的标准化方法，与自定义实现相比，它提供了更强大、更简洁的解决方案。

让我们探索如何使用HexFormat在字节数组和十六进制字符串之间进行转换。

### 8.1 字节数组转十六进制字符串

要使用HexFormat将字节数组转换为十六进制字符串，我们可以简单地使用HexFormat.of().formatHex()方法：
```java
public String encodeUsingHexFormat(byte[] bytes) {
    HexFormat hexFormat = HexFormat.of();
    return hexFormat.formatHex(bytes);
}
```

### 8.2 十六进制字符串到字节数组

类似地，要将十六进制字符串转换为字节数组，我们可以使用HexFormat.of().parseHex()方法：
```java
public byte[] decodeUsingHexFormat(String hexString) {
    HexFormat hexFormat = HexFormat.of();
    return hexFormat.parseHex(hexString);
}
```

## 9. 总结

在本文中，我们学习了字节数组到十六进制字符串之间的转换算法，我们还讨论了将字节数组编码为十六进制字符串以及反之的各种方法。

不建议添加库来仅使用几个实用方法，因此，如果我们还没有使用外部库，我们应该使用讨论的算法。DataTypeConverter类是另一种在各种数据类型之间进行编码/解码的方法。