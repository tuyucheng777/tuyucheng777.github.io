---
layout: post
title:  使用GZIPInputStream逐行读取.gz文件
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

我们可能希望在Java中使用[压缩文件](https://www.baeldung.com/java-read-zip-files)，一种常见格式是.gz，由[GZIP](https://www.baeldung.com/linux/gzip-and-gunzip)实用程序生成。

Java有一个内置的用于读取.gz文件的库，这些文件常用于日志。

**在本教程中，我们将探索使用GZIPInputStream类在Java中逐行读取压缩(.gz)文件**。

## 2. 读取GZipped文件

假设我们想将文件的内容读入[List](https://www.baeldung.com/java-collections)，首先，我们需要在路径上找到该文件：

```java
String filePath = Objects.requireNonNull(Main.class.getClassLoader().getResource("myFile.gz")).getFile();
```

接下来，让我们准备从这个文件读入一个空列表：

```java
List<String> lines = new ArrayList<>();
try (FileInputStream fileInputStream = new FileInputStream(filePath);
     GZIPInputStream gzipInputStream = new GZIPInputStream(fileInputStream);
     InputStreamReader inputStreamReader = new InputStreamReader(gzipInputStream);
     BufferedReader bufferedReader = new BufferedReader(inputStreamReader)) {

    //...
}
```

在[try-with-resources](https://www.baeldung.com/java-try-with-resources)块中，我们定义了一个FileInputStream对象来读取GZIP文件。然后，我们有一个GZIPInputStream来解压GZIP文件中的数据。最后，有一个BufferedReader来读取文件中的每一行。

现在，我们可以循环遍历文件逐行读取：

```java
String line;
while ((line = bufferedReader.readLine()) != null) {
    lines.add(line);
}
```

## 3. 使用Java Stream API处理大型GZipped文件

**当面对大型GZIP压缩文件时，我们可能没有足够的内存来加载整个文件**。但是，[流式](https://www.baeldung.com/java-streams)方法允许我们在从流中读取内容时逐行处理内容。

### 3.1 独立方法

让我们构建一个例程来从文件中收集与特定子字符串匹配的行：

```java
try (InputStream inputStream = new FileInputStream(filePath);
     GZIPInputStream gzipInputStream = new GZIPInputStream(inputStream);
     InputStreamReader inputStreamReader = new InputStreamReader(gzipInputStream);
     BufferedReader bufferedReader = new BufferedReader(inputStreamReader)) {

     return bufferedReader.lines().filter(line -> line.contains(toFind)).collect(toList());
}
```

此方法利用lines()方法从文件中创建行流，然后，后续的filter()操作选择感兴趣的行，并使用collect()将它们收集到列表中。

**使用try-with-resources可确保当所有操作完成时，各种文件和输入流都能正确关闭**。

### 3.2 使用Consumer<Stream<String\>\>

在前面的例子中，我们受益于周围的try-with-resources来管理我们的.gz流资源。然而，我们可能希望概括出一种用于动态操作从.gz文件读取的Stream<String\>的方法：

```java
try (InputStream inputStream = new FileInputStream(filePath);
     GZIPInputStream gzipInputStream = new GZIPInputStream(inputStream);
     InputStreamReader inputStreamReader = new InputStreamReader(gzipInputStream);
     BufferedReader bufferedReader = new BufferedReader(inputStreamReader)) {

    consumer.accept(bufferedReader.lines());
}
```

**这种方法允许调用者传入Consumer<Stream<String\>\>来对未压缩的行流进行操作**。此外，代码会调用该Consumer上的accept()来提供Stream，这允许我们传入任何我们想对行进行操作的内容：

```java
useContentsOfZipFile(testFilePath, linesStream -> {
    linesStream.filter(line -> line.length() > 10).forEach(line -> count.incrementAndGet());
});
```

在这个例子中，我们为Consumer提供了一个计算所有超过一定长度的行的Lambda。

## 4. 总结

在这篇短文中，我们研究了如何用Java读取.gz文件。

首先，我们研究了如何使用BufferedReader和readLine()将文件读入列表。然后，我们研究了将文件视为Stream<String\>来处理行的方法，而不必一次将它们全部加载到内存中。