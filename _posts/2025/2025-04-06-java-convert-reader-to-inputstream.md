---
layout: post
title:  Reader转为InputStream
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

在本快速教程中，我们将研究从Reader到InputStream的转换-首先使用纯Java，然后使用Guava，最后使用Apache Commons IO库。

## 2. 普通Java

让我们从Java解决方案开始：

```java
@Test
public void givenUsingPlainJava_whenConvertingReaderIntoInputStream_thenCorrect() throws IOException {
    Reader initialReader = new StringReader("With Java");

    char[] charBuffer = new char[8 * 1024];
    StringBuilder builder = new StringBuilder();
    int numCharsRead;
    while ((numCharsRead = initialReader.read(charBuffer, 0, charBuffer.length)) != -1) {
        builder.append(charBuffer, 0, numCharsRead);
    }
    InputStream targetStream = new ByteArrayInputStream(
            builder.toString().getBytes(StandardCharsets.UTF_8));

    initialReader.close();
    targetStream.close();
}
```

请注意，我们一次读取(和写入)数据块。

## 3. 使用Guava

接下来让我们看看更简单的Guava解决方案：

```java
@Test
public void givenUsingGuava_whenConvertingReaderIntoInputStream_thenCorrect() throws IOException {
    Reader initialReader = new StringReader("With Guava");

    InputStream targetStream = new ByteArrayInputStream(CharStreams.toString(initialReader)
        .getBytes(Charsets.UTF_8));

    initialReader.close();
    targetStream.close();
}
```

请注意，我们使用的是开箱即用的输入流，它将整个转换过程转变为一行代码。

## 4. 使用Commons IO

最后，让我们看几个Commons IO解决方案，也是简单的一行代码。

首先，使用[ReaderInputStream](https://commons.apache.org/proper/commons-io/apidocs/org/apache/commons/io/input/ReaderInputStream.html)：

```java
@Test
public void givenUsingCommonsIOReaderInputStream_whenConvertingReaderIntoInputStream() throws IOException {
    Reader initialReader = new StringReader("With Commons IO");

    InputStream targetStream = new ReaderInputStream(initialReader, Charsets.UTF_8);

    initialReader.close();
    targetStream.close();
}
```

最后，使用[IOUtils](https://commons.apache.org/proper/commons-io/apidocs/org/apache/commons/io/IOUtils.html#toString-java.io.Reader-)进行相同的转换：

```java
@Test
public void givenUsingCommonsIOUtils_whenConvertingReaderIntoInputStream() throws IOException {
    Reader initialReader = new StringReader("With Commons IO");

    InputStream targetStream = IOUtils.toInputStream(IOUtils.toString(initialReader), Charsets.UTF_8);

    initialReader.close();
    targetStream.close();
}
```

请注意，我们在这里处理任何类型的读取器-但如果你专门处理文本数据，那么明确指定字符集而不是使用JVM默认值始终是个好主意。

## 5. 总结

以上就是将Reader转换为InputStream的3种简单方法。