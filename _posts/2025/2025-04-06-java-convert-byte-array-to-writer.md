---
layout: post
title:  字节数组转Writer
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

在这个非常简短的教程中，我们将讨论如何使用纯Java、Guava和Commons IO将byte[]转换为Writer。

## 2. 使用纯Java

让我们从一个简单的Java解决方案开始：

```java
@Test
public void givenPlainJava_whenConvertingByteArrayIntoWriter_thenCorrect() throws IOException {
    byte[] initialArray = "With Java".getBytes();
    Writer targetWriter = new StringWriter().append(new String(initialArray));

    targetWriter.close();
    
    assertEquals("With Java", targetWriter.toString());
}
```

请注意，我们通过中间字符串将byte[]转换为Writer。

## 3. Guava

接下来让我们研究一个更复杂的Guava解决方案：

```java
@Test
public void givenUsingGuava_whenConvertingByteArrayIntoWriter_thenCorrect() throws IOException {
    byte[] initialArray = "With Guava".getBytes();

    String buffer = new String(initialArray);
    StringWriter stringWriter = new StringWriter();
    CharSink charSink = new CharSink() {
        @Override
        public Writer openStream() throws IOException {
            return stringWriter;
        }
    };
    charSink.write(buffer);

    stringWriter.close();

    assertEquals("With Guava", stringWriter.toString());
}
```

请注意，这里我们使用CharSink将byte[]转换为Writer。

## 4. Commons IO

最后，让我们看看Commons IO解决方案：

```java
@Test
public void givenUsingCommonsIO_whenConvertingByteArrayIntoWriter_thenCorrect() throws IOException {
    byte[] initialArray = "With Commons IO".getBytes();
    
    Writer targetWriter = new StringBuilderWriter(new StringBuilder(new String(initialArray)));

    targetWriter.close();

    assertEquals("With Commons IO", targetWriter.toString());
}
```

注意：我们使用StringBuilder将byte[]转换为StringBuilderWriter。

## 5. 总结

在这个简短而切中要点的教程中，我们展示了将byte[]转换为Writer的3种不同方法。