---
layout: post
title:  创建文件
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

在这个快速教程中，我们将学习如何在Java中创建一个新文件-首先使用NIO的Files和Path类，然后使用Java File和FileOutputStream类，[Google Guava](https://github.com/google/guava)，最后使用[Apache Commons IO](https://commons.apache.org/proper/commons-io/)库。

## 2. 设置

在示例中，我们将为文件名定义一个常量：

```java
private final String FILE_NAME = "src/test/resources/fileToCreate.txt";
```

我们还将添加一个清理步骤，以确保该文件在每次测试之前不存在，并在每次测试运行后将其删除：

```java
@AfterEach
@BeforeEach
public void cleanUpFiles() {
    File targetFile = new File(FILE_NAME);
    targetFile.delete();
}
```

## 3. 使用NIO Files.createFile()

让我们首先使用Java NIO包中的Files.createFile()方法：

```java
@Test
public void givenUsingNio_whenCreatingFile_thenCorrect() throws IOException {
    Path newFilePath = Paths.get(FILE_NAME);
    Files.createFile(newFilePath);
}
```

如你所见，代码仍然非常简单；我们现在使用新的Path接口而不是旧的File。

这里需要注意的一点是，新API充分利用了异常。如果文件已经存在，我们不再需要检查返回码。相反，我们会得到一个FileAlreadyExistsException：

```text
java.nio.file.FileAlreadyExistsException: <code class="language-java">src/test/resources/fileToCreate.txt在 sun.nfWindowsException.translateToIOException (WindowsException.java:81)
```

## 4. 使用File.createNewFile()

现在让我们看看如何使用java.io.File类来做同样的事情：

```java
@Test
public void givenUsingFile_whenCreatingFile_thenCorrect() throws IOException {
    File newFile = new File(FILE_NAME);
    boolean success = newFile.createNewFile();
    assertTrue(success);
}
```

请注意，该文件必须不存在才能使此操作成功。如果文件存在，则createNewFile()操作将返回false。

## 5. 使用FileOutputStream

创建新文件的另一种方法是使用java.io.FileOutputStream：

```java
@Test
public void givenUsingFileOutputStream_whenCreatingFile_thenCorrect() throws IOException {
    try(FileOutputStream fileOutputStream = new FileOutputStream(FILE_NAME)){
    }
}
```

在这种情况下，当我们实例化FileOutputStream对象时会创建一个新文件。如果给定名称的文件已经存在，则会覆盖该文件。但是，如果现有文件是一个目录或由于任何原因无法创建新文件，那么我们将得到FileNotFoundException。

此外，请注意我们使用了try-with-resources语句以确保流已正确关闭。

## 6. 使用Guava

Guava创建新文件的解决方案同样也很简单：

```java
@Test
public void givenUsingGuava_whenCreatingFile_thenCorrect() throws IOException {
    com.google.common.io.Files.touch(new File(FILE_NAME));
}
```

## 7. 使用Apache Commons IO

**Apache Commons库提供了FileUtils.touch()方法，它实现了与Linux中的”touch”实用程序相同的行为**。

因此，它会在文件系统中创建一个新的空文件甚至一个文件及其完整路径：

```java
@Test
public void givenUsingCommonsIo_whenCreatingFile_thenCorrect() throws IOException {
    FileUtils.touch(new File(FILE_NAME));
}
```

请注意，这与前面的示例略有不同：如果文件已经存在，操作不会失败，它只是不会执行任何操作。

以上就是在Java中创建新文件的4种快速方法。

## 8. 总结

在本文中，我们研究了用Java创建文件的不同解决方案，我们使用了属于JDK和外部库的类。