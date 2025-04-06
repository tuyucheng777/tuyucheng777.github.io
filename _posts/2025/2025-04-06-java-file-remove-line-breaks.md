---
layout: post
title:  如何在Java中删除文件内的换行符
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

有时，我们需要从文件中读取原始文本并通过删除换行符来清理杂乱的内容。

在本教程中，我们将探讨使用Java从文件中删除换行符的各种方法。

## 2. 关于换行符

在我们深入研究从文件读取并删除换行符的代码之前，让我们快速了解一下我们要删除的目标对象：换行符。

乍一看，这很简单，换行符就是一个字符打破一行。但是，换行符有很多种，如果我们处理不当，可能会陷入陷阱，下面举个例子来快速解释一下。

假设我们有两个文本文件，multiple-line-1.txt和multiple-line-2.txt，我们将它们称为file1和file2。如果我们在IDE的编辑器(例如[IntelliJ](https://www.baeldung.com/tag/intellij))中打开它们，则两个文件看起来相同：

```text
A,
 B,
 C,
 D,
 E,
 F
```

我们可以看到，**每个文件有6行，从第2行开始每行都有一个前导空格**。因此，我们认为file1和file2包含确切的文本。

但是，现在让我们**使用带有-n(显示行号)和-e(显示非打印字符)选项的[cat](https://www.baeldung.com/linux/files-cat-more-less#cat)命令打印文件内容**：

```shell
$ cat -ne multiple-line-1.txt
     1  A,$
     2   B,$
     3   C,$
     4   D,$
     5   E,$
     6   F$
```

file1的输出与我们在IntelliJ编辑器中看到的相同，但file2看起来完全不同：

```shell
$ cat -ne multiple-line-2.txt
     1  A,^M B,$
     2   C,$
     3   D,^M E,$
     4   F$
```

这是因为**有3种不同的换行符**：

- '\\r'：CR(回车符)，Mac OS中X之前的换行符
- '\\n'：LF(换行符)，\*nix和Mac OS中的换行符
- '\\r\\n'：CRLF，Windows中的换行符

cat -e将CRLF显示为'^M'，因此，我们看到file2包含CRLF，可能该文件是在Windows中创建的。根据需求，我们可能希望删除所有类型的换行符或仅删除当前系统的换行符。

接下来，我们将以这两个文件为例，了解如何从中读取内容并删除换行符。为简单起见，我们将创建两个辅助方法来返回每个文件的[Path](https://www.baeldung.com/java-path-vs-file#javaniofilepath-class)：

```java
Path file1Path() throws Exception {
    return Paths.get(this.getClass().getClassLoader().getResource("multiple-line-1.txt").toURI());
} 

Path file2Path() throws Exception {
    return Paths.get(this.getClass().getClassLoader().getResource("multiple-line-2.txt").toURI());
}
```

**请注意，本文中使用的方法需要将整个文本读入内存，因此请注意[非常大的文件](https://www.baeldung.com/java-read-lines-large-file)**。

## 3. 用空字符串替换line.separator

**系统属性line.separator存储特定于当前操作系统的行分隔符**，因此，如果我们只想删除特定于当前系统的换行符，我们可以用空字符串替换line.separator。例如，此方法从Linux机器上的file1中删除所有换行符：

```java
String content = Files.readString(file1Path(), StandardCharsets.UTF_8);

String result = content.replace(System.getProperty("line.separator"), "");
assertEquals("A, B, C, D, E, F", result);
```

我们使用Files类的[readString()](https://www.baeldung.com/java-11-new-features#2-new-file-methods)方法将文件内容加载到字符串中，然后，我们使用[replace()](https://www.baeldung.com/string/replace)进行替换。

但是，同样的方法不会从file2中删除所有换行符，因为它包含CRLF换行符：

```java
String content = Files.readString(file2Path(), StandardCharsets.UTF_8);

String result = content.replace(System.getProperty("line.separator"), "");
assertNotEquals("A, B, C, D, E, F", result); // <-- NOT equals assertion!
```

接下来，让我们看看是否可以独立于系统地删除所有换行符。

## 4. 用空字符串替换“\\n”和“\\r”

我们已经知道所有3种不同的换行符都涵盖“\\n”和“\\r”字符，因此，**如果我们想独立于系统删除所有换行符，我们可以用空字符串替换“\\n”和“\\r”**：

```java
String content1 = Files.readString(file1Path(), StandardCharsets.UTF_8);

// file contains CRLF
String content2 = Files.readString(file2Path(), StandardCharsets.UTF_8);

String result1 = content1.replace("\r", "").replace("\n", "");
String result2 = content2.replace("\r", "").replace("\n", "");

assertEquals("A, B, C, D, E, F", result1);
assertEquals("A, B, C, D, E, F", result2);
```

当然，**我们也可以使用基于正则表达式的[replaceAll()](https://www.baeldung.com/string/replace-all)方法来达到同样的目的**，我们以file2为例来看看它是如何工作的：

```java
String resultReplaceAll = content2.replaceAll("[\\n\\r]", "");
assertEquals("A, B, C, D, E, F", resultReplaceAll);
```

## 5. 使用readAllLines()然后join()

让我们回顾一下到目前为止学到的两种方法，我们首先从文件中读取整个内容，然后将line.separator系统属性或“\\n”和“\\r”字符替换为空，这些方法之间的一个共同点是我们自己手动管理换行符。

Files类提供[readAllLines()](https://www.baeldung.com/reading-file-in-java#read-file-with-path-readalllines)来将文件内容读成行并返回字符串列表。值得注意的是，**readAllLines()将上述三个换行符作为行分隔符**。换句话说，**此方法会从输入中删除所有换行符，我们需要做的是将返回列表中的元素合并起来**。

[join()](https://www.baeldung.com/java-strings-concatenation#3stringjoin-java-8)方法可以非常方便地拼接列表或字符串数组：

```java
List<String> lines1 = Files.readAllLines(file1Path(), StandardCharsets.UTF_8);

// file contains CRLF
List<String> lines2 = Files.readAllLines(file2Path(), StandardCharsets.UTF_8);

String result1 = String.join("", lines1);
String result2 = String.join("", lines2);

assertEquals("A, B, C, D, E, F", result1);
assertEquals("A, B, C, D, E, F", result2);
```

## 6. 总结

在本文中，我们首先讨论了不同类型的换行符。然后，我们探讨了从文件中删除换行符的各种方法。