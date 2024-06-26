---
layout: post
title:  Java - Path与File
category: java-nio
copyright: java-nio
excerpt: Java NIO
---

## 1. 概述

在Java中，Path和File是负责文件I/O操作的类。它们执行相同的功能，但属于不同的包。

在本教程中，我们将讨论这两个类之间的区别。我们将回顾一些基础知识。然后，我们将讨论一些遗留缺陷。最后，我们将学习如何在两个API之间迁移功能。

## 2. java.io.File类

从第一个版本开始，Java就提供了自己的java.io包，其中几乎包含我们执行输入和输出操作可能需要的所有类。**[File类](https://www.baeldung.com/java-io-file)是文件和目录路径名的抽象表示**：

```java
File file = new File("tuyucheng/tutorial.txt");
```

File类的实例是不可变的-一旦创建，此对象表示的抽象路径名将永远不会改变。

## 3. java.nio.file.Path类

[Path类](https://www.baeldung.com/java-nio-2-file-api)是[NIO2](https://www.baeldung.com/java-nio-2-file-api)更新的一部分，它随版本7一起出现在Java中。它提供了一个**全新的API来处理I/O**。此外，与遗留的File类一样，**Path也创建一个对象，可用于在文件系统中定位文**件。

同样，它可以执行File类可以完成的所有操作：

```java
Path path = Paths.get("tuyucheng/tutorial.txt");
```

我们没有像使用File API那样使用构造函数，而是使用静态[java.nio.file.Paths.get()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Paths.html#get(java.lang.String,java.lang.String...))方法创建Path实例。

## 4. File类的缺点

在对这两个类进行简短回顾之后，现在让我们讨论这两个API并回答这个问题：**如果它们提供相同的功能，为什么Oracle决定引入一个新的API，我应该使用哪一个**？

正如我们所知，java.io包随Java JDK的第一个版本一起提供，允许我们执行I/O操作。从那时起，许多开发人员报告了它的许多缺点、功能缺失以及某些功能的问题。

### 4.1 错误处理

最常见的问题是糟糕的错误处理。**许多方法不会告诉我们遇到问题的任何细节，甚至根本不会抛出异常**。

假设我们有一个删除文件的简单程序：

```java
File file = new File("tuyucheng/tutorial.txt");
boolean result = file.delete();
```

此代码成功编译并运行，没有任何错误。当然，我们有一个包含false值的result标志，但我们不知道失败的原因。该文件可能不存在，或者程序可能没有删除它的权限。

我们现在可以使用更新的NIO2 API重写相同的功能：

```java
Path path = Paths.get("tuyucheng/tutorial.txt");
Files.delete(path);
```

现在，编译器要求我们处理IOException。此外，抛出的异常具有有关其失败的详细信息，例如，文件是否不存在。

### 4.2 元数据支持

java.io包中的File类对元数据的支持很差，这导致跨不同平台的问题，I/O操作需要有关文件的元信息。

元数据还可能包括权限、文件所有者和安全属性。因此，File类根本**不支持符号链接**，并且**rename()方法不能在不同平台上一致地工作**。

### 4.3 方法性能

还有一个性能问题，因为File类的方法不能缩放。它会导致某些包含大量文件的目录出现问题。**列出目录的内容可能会导致挂起，从而导致内存资源问题**。最后，它可能导致拒绝服务。

由于其中一些缺点，Oracle开发了改进的NIO2 API。**在可能的情况下，开发人员应该使用这个新的java.nio包而不是遗留类来启动新项目**。

## 5. 映射功能

为了修复java.io包中的一些漏洞，Oracle准备了自己的[缺点总结](https://docs.oracle.com/javase/tutorial/essential/io/legacy.html)，帮助开发人员在API之间迁移。

NIO2包提供了所有遗留功能，包括针对上述缺点的改进。由于大量应用程序可能仍在使用此遗留API，因此**Oracle目前不打算在未来版本中弃用或删除旧API**。

在新的API中，**我们使用[java.nio.file.Files](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Files.html)类中的静态方法，而不是实例方法**。现在让我们快速比较这些API。

### 5.1 File和Path实例

当然，主要区别在于包名和类名：

```java
java.io.File file = new java.io.File("tuyucheng/tutorial.txt");
java.nio.file.Path path = java.nio.file.Paths.get("tuyucheng/tutorial.txt");
```

在这里，我们通过构造函数构建一个File对象，同时我们使用静态方法获取Path。我们还可以使用多个参数解析复杂路径：

```java
File file = new File("tuyucheng", "tutorial.txt");
Path path = Paths.get("tuyucheng", "tutorial.txt");
```

而且，我们可以通过链接resolve()方法来实现相同的结果：

```java
Path path2 = Paths.get("tuyucheng").resolve("tutorial.txt");
```

此外，我们可以使用[toPath()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/File.html#toPath())和[toFile()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Path.html#toFile())方法在API之间转换对象：

```java
Path pathFromFile = file.toPath();
File fileFromPath = path.toFile();
```

### 5.2 管理文件和目录

这两个API都提供了管理文件和目录的方法。我们将使用之前创建的实例对象对此进行演示。

要创建文件，我们可以使用createNewFile()和Files.createFile()方法：

```java
boolean result = file.createNewFile();
Path newPath = Files.createFile(path);
```

要创建目录，我们需要使用mkdir()或Files.createDirectory()：

```vhdl
boolean result = file.mkdir();
File newPath = Files.createDirectory(path);
```

这些方法还有其他变体，可以通过mkdirs()和Files.createDirectories()方法包含所有不存在的子目录：

```java
boolean result = file.mkdirs();
File newPath = Files.createDirectories(path);
```

当我们想要重命名或移动文件时，我们需要创建另一个实例对象并使用renameTo()或Files.move()：

```java
boolean result = file.renameTo(new File("tuyucheng/tutorial2.txt"));
Path newPath = Files.move(path, Paths.get("tuyucheng/tutorial2.txt"));
```

要执行删除操作，我们使用delete()或Files.delete()：

```java
boolean result = file.delete();
Files.delete(Paths.get(path));
```

请注意，如果出现任何错误，遗留方法会返回一个结果设置为false的标志。NIO2方法返回一个新的Path实例，但删除操作除外，它会在发生错误时抛出IOException。

### 5.3 读取元数据

我们还可以获取文件的一些基本信息，例如权限或类型。和以前一样，我们需要一个实例对象：

```java
// java.io API
boolean fileExists = file.exists();
boolean fileIsFile = file.isFile();
boolean fileIsDir = file.isDirectory();
boolean fileReadable = file.canRead();
boolean fileWritable = file.canWrite();
boolean fileExecutable = file.canExecute();
boolean fileHidden = file.isHidden();

// java.nio API
boolean pathExists = Files.exists(path);
boolean pathIsFile = Files.isRegularFile(path);
boolean pathIsDir = Files.isDirectory(path);
boolean pathReadable = Files.isReadable(path);
boolean pathWritable = Files.isWritable(path);
boolean pathExecutable = Files.isExecutable(path);
boolean pathHidden = Files.isHidden(path);
```

### 5.4 路径名方法

最后，让我们快速看一下[File类中用于获取文件系统路径的方法](https://www.baeldung.com/java-path)。请注意，与前面的示例不同，它们中的大多数是直接在对象实例上执行的。

要获得绝对路径或规范路径，我们可以使用：

```java
// java.io API
String absolutePathStr = file.getAbsolutePath();
String canonicalPathStr = file.getCanonicalPath();

// java.nio API
Path absolutePath = path.toAbsolutePath();
Path canonicalPath = path.toRealPath().normalize();
```

虽然Path对象是不可变的，但它会返回一个新实例。此外，NIO2 API具有toRealPath()和normalize()方法，我们可以使用它们来删除冗余。

转换为URI可以通过使用toUri()方法完成：

```java
URI fileUri = file.toURI();
URI pathUri = path.toUri();
```

此外，我们可以列出目录内容：

```java
// java.io API
String[] list = file.list();
File[] files = file.listFiles();

// java.nio API
DirectoryStream<Path> paths = Files.newDirectoryStream(path);
```

NIO2 API返回自己的[DirectoryStream](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/DirectoryStream.html)对象，该对象实现了Iterable接口。

## 6. 总结

从Java 7开始，开发人员现在可以在两个API之间进行选择来处理文件。在本文中，我们讨论了与java.io.File类相关的一些不同缺点和问题。

为了解决这些问题，Oracle决定提供NIO包，它带来了相同的功能并进行了大量改进。

然后，我们审查了这两个API。通过示例，我们学习了如何在它们之间迁移。我们还了解到java.io.File现在被认为是遗留的，不推荐用于新项目。但是，也没有计划弃用和删除它。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-nio-2)上获得。
