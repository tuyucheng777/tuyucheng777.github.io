---
layout: post
title:  使用Java创建临时目录
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

当我们需要创建一组稍后可以丢弃的文件时，临时目录会派上用场。创建临时目录时，我们可以委托操作系统来决定将它们存放在何处，也可以自行指定将它们存放在何处。

在这个简短的教程中，我们将学习**如何使用不同的API和方法在Java中创建临时目录**。本教程中的所有示例都将使用纯Java 7+、[Guava](https://search.maven.org/artifact/com.google.guava/guava/29.0-jre/bundle)和[Apache Commons IO](https://search.maven.org/artifact/org.checkerframework.annotatedlib/commons-io/2.7/jar)执行。

## 2. 委托给操作系统

用于创建临时目录的最常用的方法之一是将目标委托给底层操作系统，**该位置由java.io.tmpdir属性指定，每个操作系统都有自己的结构和清理例程**。

在普通Java中，我们通过指定我们希望目录采用的前缀来创建目录：

```java
String tmpdir = Files.createTempDirectory("tmpDirPrefix").toFile().getAbsolutePath();
String tmpDirsLocation = System.getProperty("java.io.tmpdir");
assertThat(tmpdir).startsWith(tmpDirsLocation);
```

使用Guava，过程类似，但我们不能指定如何为目录添加前缀：

```java
String tmpdir = Files.createTempDir().getAbsolutePath();
String tmpDirsLocation = System.getProperty("java.io.tmpdir");
assertThat(tmpdir).startsWith(tmpDirsLocation);
```

Apache Commons IO不提供创建临时目录的方法，它提供了一个包装器来获取操作系统临时目录，然后，由我们来完成剩下的工作：

```java
String tmpDirsLocation = System.getProperty("java.io.tmpdir");
Path path = Paths.get(FileUtils.getTempDirectory().getAbsolutePath(), UUID.randomUUID().toString());
String tmpdir = Files.createDirectories(path).toFile().getAbsolutePath();
assertThat(tmpdir).startsWith(tmpDirsLocation);
```

为了避免与现有目录发生名称冲突，我们使用UUID.randomUUID()创建一个具有随机名称的目录。

## 3. 指定位置

有时我们需要指定要创建临时目录的位置，一个很好的例子是在Maven构建期间。由于我们已经有了一个“临时”构建target目录，我们可以利用该目录来放置构建可能需要的临时目录：

```java
Path tmpdir = Files.createTempDirectory(Paths.get("target"), "tmpDirPrefix");
assertThat(tmpdir.toFile().getPath()).startsWith("target");
```

Guava和Apache Commons IO都少在特定位置创建临时目录的方法。

值得注意的是，**target目录可能因构建配置而异**，一种确保万无一失的方法是将目标目录位置传递给运行测试的JVM。

由于操作系统不负责清理，我们可以使用File.deleteOnExit()：

```java
tmpdir.toFile().deleteOnExit();
```

这样，**一旦JVM终止，文件就会被删除，但前提是终止是正常终止**。

## 4. 使用不同的文件属性

与任何其他文件或目录一样，可以在创建临时目录时指定文件属性。所以，如果我们想创建一个只能由创建它的用户读取的临时目录，我们可以指定一组属性来实现这一点：

```java
FileAttribute<Set> attrs = PosixFilePermissions.asFileAttribute(PosixFilePermissions.fromString("r--------"));
Path tmpdir = Files.createTempDirectory(Paths.get("target"), "tmpDirPrefix", attrs);
assertThat(tmpdir.toFile().getPath()).startsWith("target");
assertThat(tmpdir.toFile().canWrite()).isFalse();
```

正如预期的那样，Guava和Apache Commons IO没有提供在创建临时目录时指定属性的方法。

还值得注意的是，前面的例子假设我们处于Posix兼容文件系统下，例如Unix或macOS。

有关文件属性的更多信息，请参阅我们的[NIO2文件属性API指南](https://www.baeldung.com/java-nio2-file-attribute)。

## 5. 总结

在这个简短的教程中，我们探讨了如何在纯Java 7+、Guava和Apache Commons IO中创建临时目录。我们看到纯Java是创建临时目录最灵活的方式，因为它提供了更广泛的可能性，同时将冗长程度降至最低。