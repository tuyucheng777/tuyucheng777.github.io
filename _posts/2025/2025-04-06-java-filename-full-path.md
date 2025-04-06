---
layout: post
title:  从包含绝对文件路径的字符串获取文件名
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

当我们用Java处理文件时，我们经常需要从给定的绝对路径中提取文件名。

在本教程中，我们将探讨如何提取文件名。

## 2. 问题简介

问题很简单，假设我们得到了一个绝对文件路径字符串，我们想从中提取文件名。下面几个例子可以快速解释这个问题：

```java
String PATH_LINUX = "/root/with space/subDir/myFile.linux";
String EXPECTED_FILENAME_LINUX = "myFile.linux";

String PATH_WIN = "C:\\root\\with space\\subDir\\myFile.win";
String EXPECTED_FILENAME_WIN = "myFile.win";
```

正如我们所见，**不同的文件系统可能有不同的文件分隔符**。因此，在本教程中，我们将介绍一些独立于平台的解决方案。换句话说，相同的实现将适用于\*nix和Windows系统。

为简单起见，我们将使用单元测试断言来验证解决方案是否按预期工作。

接下来，让我们看看它们的实际效果。

## 3. 将绝对路径解析为字符串

首先，**文件系统不允许文件名包含文件分隔符**。因此，例如，我们不能在Linux的Ext2、Ext3或Ext4文件系统上创建名称包含“/”的文件：

```bash
$ touch "a/b.txt"
touch: cannot touch 'a/b.txt': No such file or directory
```

在上面的示例中，文件系统将“a/”视为一个目录。基于这个规则，解决问题的一个思路是**从最后一个文件分隔符开始取出直到字符串末尾的子字符串**。

String的[lastIndexOf()](https://www.baeldung.com/string/last-index-of)方法返回子字符串在该字符串中的最后索引，然后，我们可以通过调用absolutePath.substring(lastIndex + 1)来简单地获取文件名。

正如我们所见，实现很简单。但是，我们应该注意，为了使我们的解决方案独立于系统，我们不应该将文件分隔符硬编码为Windows的“\\\\”或\*nix系统的“/”。相反，**让我们在代码中使用[File.separator](https://www.baeldung.com/java-file-vs-file-path-separator)以便我们的程序自动适应其运行的系统**：

```java
int index = PATH_LINUX.lastIndexOf(File.separator);
String filenameLinux = PATH_LINUX.substring(index + 1);
assertEquals(EXPECTED_FILENAME_LINUX, filenameLinux);
```

如果我们在Linux机器上运行上面的测试，它就会通过。同样，下面的测试在Windows机器上通过：

```java
int index = PATH_WIN.lastIndexOf(File.pathSeparator);
String filenameWin = PATH_WIN.substring(index + 1);
assertEquals(EXPECTED_FILENAME_WIN, filenameWin);
```

正如我们所见，相同的实现在两个系统上都有效。

除了将绝对路径解析为字符串外，我们还可以使用标准的[File](https://www.baeldung.com/java-io-file)类来解决这个问题。

## 4. 使用File.getName()方法

File类提供[getName()](https://www.baeldung.com/java-io-file#2-getting-metadata-about-file-instances)方法直接获取文件名，此外，我们可以从给定的绝对路径字符串构造一个File对象。

我们首先在Linux系统上进行测试：

```java
File fileLinux = new File(PATH_LINUX);
assertEquals(EXPECTED_FILENAME_LINUX, fileLinux.getName());
```

如果我们运行测试，它就会通过。由于File内部使用File.separator，如果我们在Windows系统上测试相同的解决方案，它也会通过：

```java
File fileWin = new File(PATH_WIN);
assertEquals(EXPECTED_FILENAME_WIN, fileWin.getName());
```

## 5. 使用Path.getFileName()方法

File是java.io包中的标准类，从Java 1.7开始，较新的[java.nio](https://www.baeldung.com/java-io-vs-nio)库附带了[Path](https://www.baeldung.com/java-path-vs-file)接口。

一旦我们有了Path对象，我们就可以通过调用[Path.getFileName()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Path.html#getFileName())方法来获取文件名。与File类不同，**我们可以使用静态[Paths.get()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Paths.html#get(java.lang.String,java.lang.String...))方法创建Path实例**。

接下来，让我们从给定的PATH_LINUX字符串创建一个Path实例并在Linux上测试该解决方案：

```java
Path pathLinux = Paths.get(PATH_LINUX);
assertEquals(EXPECTED_FILENAME_LINUX, pathLinux.getFileName().toString());
```

当我们执行测试时，它通过了。值得一提的是**Path.getFileName()返回一个Path对象**，因此，我们显式调用toString()方法将其转换为字符串。

同样的实现也适用于以PATH_WIN作为路径字符串的Windows系统，这是因为Path可以检测到它正在运行的当前[FileSystem](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/FileSystem.html)：

```java
Path pathWin = Paths.get(PATH_WIN);
assertEquals(EXPECTED_FILENAME_WIN, pathWin.getFileName().toString());
```

## 6. 使用Apache Commons IO中的FilenameUtils.getName()

到目前为止，我们已经讨论了3种从绝对路径中提取文件名的解决方案。正如我们所提到的，它们与平台无关。但是，只有当给定的绝对路径与程序运行的系统相匹配时，这3种解决方案才能正常工作。例如，我们的程序只有在Windows上运行时才能处理Windows路径。

### 6.1 智能FilenameUtils.getName()方法

那么在实践中，解析不同系统的路径格式的可能性是比较低的。但是，**[Apache Commons IO](https://www.baeldung.com/apache-commons-io)的[FilenameUtils](https://www.baeldung.com/apache-commons-io#2-filenameutils)类可以“智能地”从不同的路径格式中提取文件名**。因此，如果我们的程序在Windows上运行，它也可以用于Linux文件路径，反之亦然。

接下来，让我们创建一个测试：

```java
String filenameLinux = FilenameUtils.getName(PATH_LINUX);
assertEquals(EXPECTED_FILENAME_LINUX, filenameLinux);
                                                         
String filenameWin = FilenameUtils.getName(PATH_WIN);
assertEquals(EXPECTED_FILENAME_WIN, filenameWin);
```

我们可以看到，上面的测试同时解析了PATH_LINUX和PATH_WIN，无论我们在Linux还是Windows上运行，测试都会通过。

那么接下来，我们可能想知道FilenameUtils是如何自动处理不同系统的路径的。

### 6.2 FilenameUtils.getName()的工作原理

如果我们看一下FilenameUtils.getName()的实现，它的逻辑类似于我们的“lastIndexOf”文件分隔符方法。不同之处在于FilenameUtils调用lastIndexOf()方法两次，一次使用\*nix分隔符(/)，然后使用Windows文件分隔符(\\\\)。最后，它将较大的索引作为“lastIndex”：

```java
...
final int lastUnixPos = fileName.lastIndexOf(UNIX_SEPARATOR); // UNIX_SEPARATOR = '/'
final int lastWindowsPos = fileName.lastIndexOf(WINDOWS_SEPARATOR); // WINDOWS_SEPARATOR = '\\'
return Math.max(lastUnixPos, lastWindowsPos);
```

因此，**FilenameUtils.getName()不检查当前文件系统或系统的文件分隔符**，而是找到最后一个文件分隔符的索引(无论它属于哪个系统)，然后从此索引中提取子字符串，直到字符串末尾作为最终结果。

### 6.3 导致FilenameUtils.getName()失败的极端情况

现在我们了解了FilenameUtils.getName() 的工作原理，这确实是一个聪明的解决方案，并且在大多数情况下都有效。但是，**许多Linux支持的文件系统允许文件名包含反斜杠('\\')**：

```shell
$ echo 'Hi there!' > 'my\file.txt'

$ ls -l my*
-rw-r--r-- 1 kent kent 10 Sep 13 23:55 'my\file.txt'

$ cat 'my\file.txt'
Hi there!
```

如果给定的Linux文件路径中的文件名包含反斜杠，则FilenameUtils.getName()将失败，测试可以清楚地解释这一点：

```java
String filenameToBreak = FilenameUtils.getName("/root/somedir/magic\\file.txt");
assertNotEquals("magic\\file.txt", filenameToBreak); // <-- filenameToBreak = "file.txt", but we expect: magic\file.txt
```

当使用此方法时，我们应该牢记这种情况。

## 7. 总结

在本文中，我们学习了如何从给定的绝对路径字符串中提取文件名。