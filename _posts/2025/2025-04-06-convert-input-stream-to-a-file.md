---
layout: post
title:  将InputStream写入文件
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

在本快速教程中，我们将说明如何将InputStream写入文件。首先我们将使用纯Java，然后是Guava，最后是Apache Commons IO库。

## 2. 使用普通Java进行转换

让我们从Java解决方案开始：

```java
@Test
public void whenConvertingToFile_thenCorrect() throws IOException {
    Path path = Paths.get("src/test/resources/sample.txt");
    byte[] buffer = java.nio.file.Files.readAllBytes(path);

    File targetFile = new File("src/test/resources/targetFile.tmp");
    OutputStream outStream = new FileOutputStream(targetFile);
    outStream.write(buffer);

    IOUtils.closeQuietly(outStream);
}
```

请注意，在此示例中，输入流具有已知和预先确定的数据，例如磁盘上的文件或内存中的流。因此，**我们不需要进行任何边界检查**，如果内存允许，我们可以简单地一次性读取和写入。

如果输入流链接到持续的数据流，例如来自持续连接的HTTP响应，则一次读取整个流是不可行的。在这种情况下，我们需要确保一直读取直到到达流的末尾：

```java
@Test
public void whenConvertingInProgressToFile_thenCorrect() throws IOException {
    InputStream initialStream = new FileInputStream(new File("src/main/resources/sample.txt"));
    File targetFile = new File("src/main/resources/targetFile.tmp");
    OutputStream outStream = new FileOutputStream(targetFile);

    byte[] buffer = new byte[8 * 1024];
    int bytesRead;
    while ((bytesRead = initialStream.read(buffer)) != -1) {
        outStream.write(buffer, 0, bytesRead);
    }
    IOUtils.closeQuietly(initialStream);
    IOUtils.closeQuietly(outStream);
}
```

最后，这是我们可以使用Java 8执行相同操作的另一种简单方法：

```java
@Test
public void whenConvertingAnInProgressInputStreamToFile_thenCorrect2() throws IOException {
    InputStream initialStream = new FileInputStream(new File("src/main/resources/sample.txt"));
    File targetFile = new File("src/main/resources/targetFile.tmp");

    java.nio.file.Files.copy(
        initialStream, 
        targetFile.toPath(), 
        StandardCopyOption.REPLACE_EXISTING);

    IOUtils.closeQuietly(initialStream);
}
```

## 3. 使用Guava转换

接下来我们看一个更简单的基于Guava的解决方案：

```java
@Test
public void whenConvertingInputStreamToFile_thenCorrect3() throws IOException {
    InputStream initialStream = new FileInputStream(new File("src/main/resources/sample.txt"));
    byte[] buffer = new byte[initialStream.available()];
    initialStream.read(buffer);

    File targetFile = new File("src/main/resources/targetFile.tmp");
    Files.write(buffer, targetFile);
}
```

## 4. 使用Commons IO转换

最后，这是使用Apache Commons IO的更快解决方案：

```java
@Test
public void whenConvertingInputStreamToFile_thenCorrect4() throws IOException {
    InputStream initialStream = FileUtils.openInputStream(new File("src/main/resources/sample.txt"));

    File targetFile = new File("src/main/resources/targetFile.tmp");

    FileUtils.copyInputStreamToFile(initialStream, targetFile);
}
```

以上就是将InputStream写入文件的3种快捷方法。