---
layout: post
title:  使用Java将byte数组写入文件
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

在本快速教程中，我们将学习几种将Java字节数组写入文件的不同方法。我们将从头开始，使用Java IO包。接下来，我们看一个使用Java NIO的示例。之后，我们将使用Google Guava和Apache Commons IO。

## 2. Java IO

Java的IO包从JDK 1.0开始就有了，它提供了一组用于读写数据的类和接口。

让我们使用FileOutputStream将图像写入文件：

```java
File outputFile = tempFolder.newFile("outputFile.jpg");
try (FileOutputStream outputStream = new FileOutputStream(outputFile)) {
    outputStream.write(dataForWriting);
}
```

我们打开一个输出流到目标文件，然后我们可以简单地将byte[] dataForWriting传递给write方法。请注意，我们在这里使用[try-with-resources](https://www.baeldung.com/java-try-with-resources)块来确保在抛出IOException时关闭OutputStream。

## 3. Java NIO

Java NIO包在Java 1.4中引入，[NIO的文件系统API](https://www.baeldung.com/java-nio-2-file-api)作为扩展在Java 7中引入。**Java NIO使用缓冲且是非阻塞的，而Java IO使用阻塞流**。在java.nio.file包中，创建文件资源的语法更简洁。

我们可以使用Files类在单行中写入byte[]：

```java
Files.write(outputFile.toPath(), dataForWriting);
```

我们的示例要么创建一个文件，要么截断现有文件并打开它进行写入。我们还可以使用Paths.get("path/to/file")或Paths.get("path", "to", "file")来构造描述文件存储位置的路径，Path是表示路径的Java NIO原生方式。

如果我们需要覆盖文件打开行为，我们也可以向write方法提供[OpenOption](https://www.baeldung.com/java-file-options)。

## 4. Google Guava

[Guava](https://www.baeldung.com/guava-write-to-file-read-from-file)是Google的一个库，它提供了多种类型来执行Java中的常见操作，包括IO。

让我们将[Guava](https://mvnrepository.com/artifact/com.google.guava/guava)导入到pom.xml文件中：

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

### 4.1 Guava Files

与Java NIO包一样，我们可以在单行中写入byte[]：

```java
Files.write(dataForWriting, outputFile);
```

Guava的Files.write方法也接收可选的OptionOptions并使用与java.nio.Files.write相同的默认值。

不过这里有一个问题：**Guava Files.write方法标有@Beta注解，[根据文档](https://github.com/google/guava#important-warnings)，这意味着它可能随时更改，因此不建议在库中使用**。

所以，如果我们正在编写一个库项目，我们应该考虑使用ByteSink。

### 4.2 ByteSink

我们还可以创建一个ByteSink来写入我们的byte[]：

```java
ByteSink byteSink = Files.asByteSink(outputFile);
byteSink.write(dataForWriting);
```

**ByteSink是我们可以向其写入字节的目的地**，它向目的地提供一个OutputStream。

如果我们需要使用java.nio.files.Path或提供特殊的OpenOption，我们可以使用MoreFiles类获取我们的ByteSink：

```java
ByteSink byteSink = MoreFiles.asByteSink(outputFile.toPath(), 
    StandardOpenOption.CREATE, 
    StandardOpenOption.WRITE);
byteSink.write(dataForWriting);
```

## 5. Apache Commons IO

[Apache Commons IO](https://www.baeldung.com/apache-commons-io)提供了一些常见的文件任务。

让我们导入最新版本的[commons-io](https://mvnrepository.com/artifact/commons-io/commons-io)：

```xml
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.11.0</version>
</dependency>

```

现在，让我们使用FileUtils类写入byte[]：

```java
FileUtils.writeByteArrayToFile(outputFile, dataForWriting);
```

FileUtils.writeByteArrayToFile方法类似于我们使用的其他方法，我们给它一个File，表示我们想要的目标和我们要写入的二进制数据。**如果我们的目标文件或任何父目录不存在，它们将被创建**。

## 6. 总结

在这个简短的教程中，我们学习了如何使用普通Java和两个流行的Java实用程序库：Google Guava和Apache Commons IO将二进制数据从byte[]写入文件。