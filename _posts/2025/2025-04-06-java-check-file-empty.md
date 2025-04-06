---
layout: post
title:  使用Java检查文件是否为空
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

我们经常会遇到需要在Java应用程序中处理[文件](https://www.baeldung.com/java-io-file)的场景，有时，我们想在继续进一步的操作之前确定文件是否为空。

在本教程中，我们将探讨一些使用Java检查文件是否为空的有效而直接的方法。

## 2. 问题介绍

在深入实现之前，让我们先了解文件为空的含义。

在文件操作中，**空文件是指不包含任何数据或大小为0字节的文件**。

验证文件是否为空在处理输入或输出操作(例如读取或解析文件)时特别有用。

**Java标准库提供了获取文件大小的方法，但是，我们需要注意一些陷阱**。

为简单起见，我们将使用单元测试断言来验证我们的方法是否按预期工作。此外，[JUnit 5的TempDirectory扩展](https://www.baeldung.com/junit-5-temporary-directory)允许我们轻松地在单元测试中创建和清理临时目录。由于我们的测试与文件相关，我们将使用此扩展来支持我们的验证。

## 3. 使用File.length()方法

我们知道，我们可以通过检查文件的大小来确定文件是否为空，**Java标准IO库提供了File.length()方法来计算文件的大小(以字节为单位)**。

因此，我们可以通过检查File.length()是否返回0来解决问题：

```java
@Test
void whenTheFileIsEmpty_thenFileLengthIsZero(@TempDir Path tempDir) throws IOException {
    File emptyFile = tempDir.resolve("an-empty-file.txt").toFile();
    emptyFile.createNewFile();
    assertTrue(emptyFile.exists());
    assertEquals(0, emptyFile.length());
}
```

调用File.length()来检查文件是否为空很方便，但这有一个陷阱，让我们通过另一个测试来理解它：

```java
@Test
void whenFileDoesNotExist_thenFileLengthIsZero(@TempDir Path tempDir) {
    File aNewFile = tempDir.resolve("a-new-file.txt").toFile();
    assertFalse(aNewFile.exists());
    assertEquals(0, aNewFile.length());
}
```

在上面的测试中，我们像往常一样初始化了一个File对象。但是，我们并没有创建该文件。换句话说，**文件tempDir/a-new-file.txt不存在**。

因此，测试表明，当我们对不存在的文件调用File.length()时，它也会返回0。通常，当我们说文件为空时，该文件一定存在。因此，仅通过File.length()进行空检查是不可靠的。

接下来，让我们创建一个方法来解决这个问题：

```java
boolean isFileEmpty(File file) {
    if (!file.exists()) {
        throw new IllegalArgumentException("Cannot check the file length. The file is not found: " + file.getAbsolutePath());
    }
    return file.length() == 0;
}
```

在上述方法中，如果文件不存在，我们会引发IllegalArgumentException。有些人可能认为[FileNotFoundException](https://www.baeldung.com/java-filenotfound-exception)更合适，在这里，我们没有选择FileNotFoundException，因为它是一个[受检](https://www.baeldung.com/java-checked-unchecked-exceptions)，如果我们在调用isFileEmpty()方法时抛出此异常，我们必须处理该异常。另一方面，**IllegalArgumentException是一个非受检异常**，表示文件参数无效。

现在，无论文件是否存在，isFileEmpty()方法都会执行该任务：

```java
@Test
void whenTheFileDoesNotExist_thenIsFilesEmptyThrowsException(@TempDir Path tempDir) {
    File aNewFile = tempDir.resolve("a-new-file.txt")
            .toFile();
    IllegalArgumentException ex = assertThrows(IllegalArgumentException.class, () -> isFileEmpty(aNewFile));
    assertEquals(ex.getMessage(), "Cannot check the file length. The file is not found: " + aNewFile.getAbsolutePath());
}

@Test
void whenTheFileIsEmpty_thenIsFilesEmptyReturnsTrue(@TempDir Path tempDir) throws IOException {
    File emptyFile = tempDir.resolve("an-empty-file.txt")
            .toFile();
    emptyFile.createNewFile();
    assertTrue(isFileEmpty(emptyFile));
}
```

## 4. 使用NIO Files.size()方法

我们已经使用File.length()解决了这个问题。

File.length()来自标准Java IO；或者，如果我们使用的是Java版本7或更高版本，我们可以使用[Java NIO](https://www.baeldung.com/tag/java-nio) API解决问题。例如，java.nio.file.Files.size()返回文件的大小，这也可以帮助我们检查文件是否为空。

值得一提的是，如果文件不存在，**Java NIO的Files.size()会抛出NoSuchFileException**：

```java
@Test
void whenTheFileIsEmpty_thenFilesSizeReturnsTrue(@TempDir Path tempDir) throws IOException {
    Path emptyFilePath = tempDir.resolve("an-empty-file.txt");
    Files.createFile(emptyFilePath);
    assertEquals(0, Files.size(emptyFilePath));
}

@Test
void whenTheFileDoesNotExist_thenFilesSizeThrowsException(@TempDir Path tempDir) {
    Path aNewFilePath = tempDir.resolve("a-new-file.txt");
    assertThrows(NoSuchFileException.class, () -> Files.size(aNewFilePath));
}
```

## 5. 总结

在本文中，我们探讨了两种使用Java检查文件是否为空的方法：

- 使用Java标准IO中的File.length()
- 使用Java NIO中的Files.size()