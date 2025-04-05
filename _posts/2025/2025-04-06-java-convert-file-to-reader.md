---
layout: post
title:  转化File为Reader
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

在本快速教程中，我们将说明如何使用普通Java、Guava或Apache Commons IO将File转换为Reader。

## 2. 使用纯Java

我们首先看一下简单的Java解决方案：

```java
@Test
public void givenUsingPlainJava_whenConvertingFileIntoReader_thenCorrect() throws IOException {
    File initialFile = new File("src/test/resources/initialFile.txt");
    initialFile.createNewFile();
    Reader targetReader = new FileReader(initialFile);
    targetReader.close();
}
```

## 2. Guava

现在让我们看看同样的转换，这次使用Guava库：

```java
@Test
public void givenUsingGuava_whenConvertingFileIntoReader_thenCorrect() throws IOException {
    File initialFile = new File("src/test/resources/initialFile.txt");
    com.google.common.io.Files.touch(initialFile);
    Reader targetReader = Files.newReader(initialFile, Charset.defaultCharset());
    targetReader.close();
}
```

## 3. Commons IO

最后，让我们使用Commons IO通过中间字节数组进行转换：

```java
@Test
public void givenUsingCommonsIO_whenConvertingFileIntoReader_thenCorrect() throws IOException {
    File initialFile = new File("src/test/resources/initialFile.txt");
    FileUtils.touch(initialFile);
    FileUtils.write(initialFile, "With Commons IO");
    byte[] buffer = FileUtils.readFileToByteArray(initialFile);
    Reader targetReader = new CharSequenceReader(new String(buffer));
    targetReader.close();
}
```

以上就是将File转换为Reader的三种方法：首先使用纯Java，然后使用Guava，最后使用Apache Commons IO库。