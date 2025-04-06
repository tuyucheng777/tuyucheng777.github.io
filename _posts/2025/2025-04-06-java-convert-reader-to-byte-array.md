---
layout: post
title:  Reader转为字节数组
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

本快速教程将展示如何使用纯Java、Guava和Apache Commons IO库将Reader转换为byte[\]。

## 2. 普通Java

让我们从简单的Java解决方案开始-通过一个中间字符串：

```java
@Test
public void givenUsingPlainJava_whenConvertingReaderIntoByteArray_thenCorrect() throws IOException {
    Reader initialReader = new StringReader("With Java");

    char[] charArray = new char[8 * 1024];
    StringBuilder builder = new StringBuilder();
    int numCharsRead;
    while ((numCharsRead = initialReader.read(charArray, 0, charArray.length)) != -1) {
        builder.append(charArray, 0, numCharsRead);
    }
    byte[] targetArray = builder.toString().getBytes();

    initialReader.close();
}
```

请注意，读取是分块进行的，而不是一次一个字符。

## 3. Guava

接下来让我们看看Guava解决方案-同样使用中间字符串：

```java
@Test
public void givenUsingGuava_whenConvertingReaderIntoByteArray_thenCorrect() throws IOException {
    Reader initialReader = CharSource.wrap("With Google Guava").openStream();

    byte[] targetArray = CharStreams.toString(initialReader).getBytes();

    initialReader.close();
}
```

请注意，我们正在使用内置的实用程序API，而不必执行任何普通Java示例的低级转换。

## 4. Commons IO

最后，这里有一个通过Commons IO开箱即用的直接解决方案：

```java
@Test
public void givenUsingCommonsIO_whenConvertingReaderIntoByteArray_thenCorrect() throws IOException {
    StringReader initialReader = new StringReader("With Commons IO");

    byte[] targetArray = IOUtils.toByteArray(initialReader);

    initialReader.close();
}
```

以上就是将Java Reader转换为字节数组的3种快速方法。