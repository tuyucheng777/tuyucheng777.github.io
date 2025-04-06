---
layout: post
title:  使用Java检查目录是否为空
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

在本快速教程中，我们将熟悉几种判断目录是否为空的方法。

## 2. 使用Files.newDirectoryStream

**从Java 7开始，[Files.newDirectoryStream](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Files.html#newDirectoryStream(java.nio.file.Path))方法返回一个[DirectoryStream](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/DirectoryStream.html)<Path\>以迭代目录中的所有条目**，因此我们可以使用这个API来检查给定的目录是否为空：

```java
public boolean isEmpty(Path path) throws IOException {
    if (Files.isDirectory(path)) {
        try (DirectoryStream<Path> directory = Files.newDirectoryStream(path)) {
            return !directory.iterator().hasNext();
        }
    }

    return false;
}
```

对于非目录输入，我们甚至不会尝试加载目录条目，而是返回false：

```java
Path aFile = Paths.get(getClass().getResource("/notDir.txt").toURI());
assertThat(isEmpty(aFile)).isFalse();
```

另一方面，如果输入是目录，我们将尝试打开指向该目录的DirectoryStream。然后，**当且仅当第一个hasNext()方法调用返回false时，我们才会认为该目录为空**。否则，它不为空：

```java
Path currentDir = new File("").toPath().toAbsolutePath();
assertThat(isEmpty(currentDir)).isFalse();
```

DirectoryStream是一个Closeable资源，因此我们将其包装在一个[try-with-resources](https://www.baeldung.com/java-try-with-resources)块中。正如我们所料，isEmpty方法对空目录返回true：

```java
Path path = Files.createTempDirectory("tuyucheng-empty");
assertThat(isEmpty(path)).isTrue();
```

在这里，我们使用[Files.createTempDirectory](https://www.baeldung.com/java-nio-2-file-api#creating-temporary-files)创建一个空的临时目录。

## 3. 使用Files.list

**从JDK 8开始，[Files.list](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Files.html#list(java.nio.file.Path))方法在内部使用Files.newDirectoryStream API来公开Stream<Path\>**。每个Path都是给定父目录中的一个条目，因此，我们也可以使用这个API来达到同样的目的：

```java
public boolean isEmpty(Path path) throws IOException {
    if (Files.isDirectory(path)) {
        try (Stream<Path> entries = Files.list(path)) {
            return !entries.findFirst().isPresent();
        }
    }

    return false;
}
```

同样，我们只使用findFirst方法接触第一个条目。如果返回的[Optional](https://www.baeldung.com/java-optional)为空，则目录也为空。

[Stream由I/O资源支持](https://www.baeldung.com/java-stream-close)，因此我们确保使用try-with-resources块适当地释放它。

## 4. 低效的解决方案

**Files.list和Files.newDirectoryStream都会惰性地迭代目录条目，因此，它们可以非常高效地处理大型目录**。但是，像这样的解决方案在这种情况下效果不佳：

```java
public boolean isEmpty(Path path) {
    return path.toFile().listFiles().length == 0;
}
```

这将急切地加载目录中的所有条目，这在处理大目录时效率非常低。

## 5. 总结

在这个简短的教程中，我们熟悉了一些检查目录是否为空的有效方法。