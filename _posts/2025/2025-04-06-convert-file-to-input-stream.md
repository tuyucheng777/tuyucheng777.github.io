---
layout: post
title:  File转为InputStream
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

在本快速教程中，我们将展示如何将File转换为InputStream-首先使用纯Java，然后使用Guava和Apache Commons IO库。

## 2. 使用Java转换

**我们可以使用Java的[IO包](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/package-summary.html)将一个File转换为不同的InputStream**。

### 2.1 FileInputStream

让我们从第一个也是最简单的开始-使用FileInputStream：

```java
@Test
public void givenUsingPlainJava_whenConvertingFileToInputStream_thenCorrect() throws IOException {
    File initialFile = new File("src/main/resources/sample.txt");
    InputStream targetStream = new FileInputStream(initialFile);
}
```

### 2.2 DataInputStream

让我们看看另一种方法，我们可以使用DataInputStream从文件中读取二进制或原始数据：

```java
@Test
public final void givenUsingPlainJava_whenConvertingFileToDataInputStream_thenCorrect() throws IOException {
      final File initialFile = new File("src/test/resources/sample.txt");
      final InputStream targetStream = new DataInputStream(new FileInputStream(initialFile));
}
```

### 2.3 SequenceInputStream

最后，让我们看看如何**使用SequenceInputStream将两个文件的输入流拼接到单个InputStream**：

```java
@Test
public final void givenUsingPlainJava_whenConvertingFileToSequenceInputStream_thenCorrect() throws IOException {
      final File initialFile = new File("src/test/resources/sample.txt");
      final File anotherFile = new File("src/test/resources/anothersample.txt");
      final InputStream targetStream = new FileInputStream(initialFile);
      final InputStream anotherTargetStream = new FileInputStream(anotherFile);
    
      InputStream sequenceTargetStream = new SequenceInputStream(targetStream, anotherTargetStream);
}
```

请注意，为了清晰起见，我们没有关闭这些示例中的结果流。

## 3. 使用Guava转换

接下来，让我们看看使用中间ByteSource的Guava解决方案：

```java
@Test
public void givenUsingGuava_whenConvertingFileToInputStream_thenCorrect() throws IOException {
    File initialFile = new File("src/main/resources/sample.txt");
    InputStream targetStream = Files.asByteSource(initialFile).openStream();
}
```

## 4. 使用Commons IO进行转换

最后，让我们看一下使用Apache Commons IO的解决方案：

```java
@Test
public void givenUsingCommonsIO_whenConvertingFileToInputStream_thenCorrect() 
  throws IOException {
    File initialFile = new File("src/main/resources/sample.txt");
    InputStream targetStream = FileUtils.openInputStream(initialFile);
}
```

就这样，我们有了从Java文件打开流的3种简单干净的解决方案。

## 5. 总结

在本文中，我们探讨了使用不同的库将File转换为InputStream的各种方法。