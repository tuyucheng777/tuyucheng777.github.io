---
layout: post
title:  在Java中将相对路径转换为绝对路径
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

在Java中处理文件路径是一项常见任务，有时，出于各种原因，我们需要将相对路径转换为绝对路径。无论我们是在处理文件操作、访问资源还是浏览目录，知道如何将相对路径转换为绝对路径都是必不可少的。

在本教程中，我们将探讨在Java中实现这种转换的不同方法。

## 2. 解决方案

### 2.1 使用Paths类

Java 7中引入的java.nio.file包提供了Paths类，提供了一种操作文件和目录路径的便捷方法。

让我们使用Paths类将相对路径转换为绝对路径：

```java
String relativePath = "myFolder/myFile.txt";

Path absolutePath = Paths.get(relativePath).toAbsolutePath();
```

### 2.2 使用File类

在Java 7之前，[java.io.File](https://www.baeldung.com/java-io-file)类提供了一种将相对路径转换为绝对路径的方法。

下面是如何使用File类转换相对路径的示例：

```java
String relativePath = "myFolder/myFile.txt";

File file = new File(relativePath);

String absolutePath = file.getAbsolutePath();
```

**虽然建议对新项目使用较新的Paths类，但File类对于旧代码仍然可用**。

### 2.3 使用FileSystem类

另一种方法是使用java.nio.file.FileSystem类，它提供了转换路径的方法：

```java
String relativePath = "myFolder/myFile.txt";

Path absolutePath = FileSystems.getDefault().getPath(relativePath).toAbsolutePath();
```

## 3. 示例

让我们用相对路径测试我们的解决方案：

```java
String relativePath1 = "data/sample.txt";
System.out.println(convertToAbsoluteUsePathsClass(relativePath1));
System.out.println(convertToAbsoluteUseFileClass(relativePath1));
System.out.println(convertToAbsoluteUseFileSystemsClass(relativePath1));
```

结果将如下所示(**结果可能因所使用的操作系统而异**-此示例使用Windows)：

```text
D:\SourceCode\tutorials\core-java-modules\core-java-20\data\sample.txt
D:\SourceCode\tutorials\core-java-modules\core-java-20\data\sample.txt
D:\SourceCode\tutorials\core-java-modules\core-java-20\data\sample.txt
```

我们再试一次：

```java
String relativePath2 = "../data/sample.txt";
System.out.println(convertToAbsoluteUsePathsClass(relativePath2));
System.out.println(convertToAbsoluteUseFileClass(relativePath2));
System.out.println(convertToAbsoluteUseFileSystemsClass(relativePath2));
```

这次的结果将会是这样的：

```text
D:\SourceCode\tutorials\core-java-modules\core-java-20\..\data\sample.txt
D:\SourceCode\tutorials\core-java-modules\core-java-20\..\data\sample.txt
D:\SourceCode\tutorials\core-java-modules\core-java-20\..\data\sample.txt
```

**如果我们想从路径中删除任何冗余元素(例如“.”或“..”)，我们可以使用Path类的normalize()方法**。

## 4. 总结

将相对路径转换为绝对路径对于Java中的文件操作、资源访问或目录导航至关重要。

在本教程中，我们探索了实现这种转换的不同方法。