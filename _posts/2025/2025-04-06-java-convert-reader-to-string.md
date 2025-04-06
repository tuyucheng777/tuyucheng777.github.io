---
layout: post
title:  Reader转为String
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

在本快速教程中，我们将使用纯Java、Guava和Apache Commons IO库将Reader转换为String。

## 2. 普通Java

让我们从一个简单的Java解决方案开始，该解决方案从Reader顺序读取字符：

```java
@Test
public void givenUsingPlainJava_whenConvertingReaderIntoStringV1_thenCorrect() throws IOException {
    StringReader reader = new StringReader("text");
    int intValueOfChar;
    String targetString = "";
    while ((intValueOfChar = reader.read()) != -1) {
        targetString += (char) intValueOfChar;
    }
    reader.close();
}
```

如果要读取的内容很多，批量读取的解决方案会更好：

```java
@Test
public void givenUsingPlainJava_whenConvertingReaderIntoStringV2_thenCorrect() throws IOException {
    Reader initialReader = new StringReader("text");
    char[] arr = new char[8 * 1024];
    StringBuilder buffer = new StringBuilder();
    int numCharsRead;
    while ((numCharsRead = initialReader.read(arr, 0, arr.length)) != -1) {
        buffer.append(arr, 0, numCharsRead);
    }
    initialReader.close();
    String targetString = buffer.toString();
}
```

## 3. Guava

Guava提供了一个可以直接进行转换的实用程序：

```java
@Test
public void givenUsingGuava_whenConvertingReaderIntoString_thenCorrect() throws IOException {
    Reader initialReader = CharSource.wrap("With Google Guava").openStream();
    String targetString = CharStreams.toString(initialReader);
    initialReader.close();
}
```

## 4. 使用Commons IO

Apache Commons IO相同-有一个能够执行直接转换的IO实用程序：

```java
@Test
public void givenUsingCommonsIO_whenConvertingReaderIntoString_thenCorrect() throws IOException {
    Reader initialReader = new StringReader("With Apache Commons");
    String targetString = IOUtils.toString(initialReader);
    initialReader.close();
}
```

以上就是将Reader转换为纯字符串的4种方法。