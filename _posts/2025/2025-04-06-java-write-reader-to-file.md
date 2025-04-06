---
layout: post
title:  将Reader写入文件
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

在本快速教程中，我们将使用普通Java将Reader的内容写入文件，然后是Guava，最后是Apache Commons IO库。

## 2. 普通Java

让我们从简单的Java解决方案开始：

```java
@Test
public void givenUsingPlainJava_whenWritingReaderContentsToFile_thenCorrect() throws IOException {
    Reader initialReader = new StringReader("Some text");

    int intValueOfChar;
    StringBuilder buffer = new StringBuilder();
    while ((intValueOfChar = initialReader.read()) != -1) {
        buffer.append((char) intValueOfChar);
    }
    initialReader.close();

    File targetFile = new File("src/test/resources/targetFile.txt");
    targetFile.createNewFile();

    Writer targetFileWriter = new FileWriter(targetFile);
    targetFileWriter.write(buffer.toString());
    targetFileWriter.close();
}
```

首先我们将Reader的内容读入一个字符串；然后我们只是将字符串写入文件。

## 3. Guava

Guava解决方案更简单=-我们现在有API来处理将Reader写入文件：

```java
@Test
public void givenUsingGuava_whenWritingReaderContentsToFile_thenCorrect() throws IOException {
    Reader initialReader = new StringReader("Some text");

    File targetFile = new File("src/test/resources/targetFile.txt");
    com.google.common.io.Files.touch(targetFile);
    CharSink charSink = com.google.common.io.Files.asCharSink(targetFile, Charset.defaultCharset(), FileWriteMode.APPEND);
    charSink.writeFrom(initialReader);

    initialReader.close();
}
```

## 4. 使用Apache Commons IO

最后，Commons IO解决方案也使用更高级别的API从Reader读取数据并将该数据写入文件：

```java
@Test
public void givenUsingCommonsIO_whenWritingReaderContentsToFile_thenCorrect() throws IOException {
    Reader initialReader = new CharSequenceReader("CharSequenceReader extends Reader");

    File targetFile = new File("src/test/resources/targetFile.txt");
    FileUtils.touch(targetFile);
    byte[] buffer = IOUtils.toByteArray(initialReader);
    FileUtils.writeByteArrayToFile(targetFile, buffer);

    initialReader.close();
}
```

以上就是将Reader的内容写入File的3个简单解决方案。