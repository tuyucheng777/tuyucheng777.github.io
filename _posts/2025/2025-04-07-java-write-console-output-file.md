---
layout: post
title:  使用Java将控制台输出写入文本文件
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

将输出打印到控制台以进行调试或向用户显示信息是很常见的，但是，有时可能需要将控制台输出保存到文本文件以供进一步分析或记录。

在本教程中，我们将探讨如何将控制台输出重定向到Java中的文本文件。

## 2. 准备

当我们谈论将控制台输出写入文本文件时，可能有两种情况：

- 仅文件：将所有输出重定向到文件，不会将任何输出打印到控制台
- 控制台和文件：输出写入控制台和文件

我们将在本教程中介绍这两种情况。

在进入编码部分之前，让我们准备一些文本写入控制台。为了更容易测试，让我们将三行数据放在字符串列表中：

```java
final static List<String> OUTPUT_LINES = Lists.newArrayList(
    "I came",
    "I saw",
    "I conquered");
```

稍后，我们将讨论如何将控制台的输出重定向到文件。因此，我们将使用单元测试断言来验证文件是否包含预期的内容。

**JUnit 5的[临时目录](https://www.baeldung.com/junit-5-temporary-directory)功能允许我们创建文件、向文件写入数据，并在验证后自动删除文件**。因此，我们将在测试中使用它。

接下来，让我们看看如何实现重定向。

## 3. 输出仅到文件

通常，我们使用System.out.println()将文本打印到控制台。**System.out是一个[PrintStream](https://www.baeldung.com/java-printstream-vs-printwriter)，默认情况下是标准输出**。System类提供了setOut()方法，允许我们用PrintStream对象之一替换默认的“out”。

**由于我们要将数据写入文件，因此我们可以从[FileOutputStream](https://www.baeldung.com/java-outputstream#1-fileoutputstream)创建一个PrintStream**。

接下来我们创建一个测试方法来看看这个想法是否可行：

```java
@Test
void whenReplacingSystemOutPrintStreamWithFileOutputStream_thenOutputsGoToFile(@TempDir Path tempDir) throws IOException {
    PrintStream originalOut = System.out;
    Path outputFilePath = tempDir.resolve("file-output.txt");
    PrintStream out = new PrintStream(Files.newOutputStream(outputFilePath), true);
    System.setOut(out);

    OUTPUT_LINES.forEach(line -> System.out.println(line));
    assertTrue(outputFilePath.toFile().exists(), "The file exists");
    assertLinesMatch(OUTPUT_LINES, Files.readAllLines(outputFilePath));
    System.setOut(originalOut);
}
```

在测试中，我们首先备份默认的System.out。然后，我们构造一个PrintStream out，它包装了一个FileOutputStream。接下来，我们用自己的“out”替换默认的System.out。

这些操作使System.out.println()将数据打印到文件file-output.txt而不是控制台，我们使用assertLinesMatch()方法验证了文件内容。

最后，我们将System.out恢复为默认值。

如果我们运行测试，它就会通过。此外，控制台上不会打印任何输出。

## 4. 创建DualPrintStream类

现在，让我们看看如何将数据打印到控制台和文件。换句话说，**我们需要两个PrintStream对象**。

由于System.setOut()仅接收一个PrintStream参数，因此我们不能传递两个PrintStream。但是，我们可以**创建一个新的PrintStream子类来携带一个额外的PrintStream对象**：

```java
class DualPrintStream extends PrintStream {
    private final PrintStream second;

    public DualPrintStream(OutputStream main, PrintStream second) {
        super(main);
        this.second = second;
    }

    // ...
}
```

DualPrintStream扩展了PrintStream；此外，我们可以将一个额外的PrintStream对象(第二个)传递给构造函数。然后，**我们必须重写PrintStream的write()方法，以便第二个PrintStream可以搭载相同的方法并应用相同的操作**：

```java
class DualPrintStream extends PrintStream {
    // ...
    
    @Override
    public void write(byte[] b) throws IOException {
        super.write(b);
        second.write(b);
    }
}
```

现在，让我们检查它是否按预期工作：

```java
@Test
void whenUsingDualPrintStream_thenOutputsGoToConsoleAndFile(@TempDir Path tempDir) throws IOException {
    PrintStream originalOut = System.out;
    Path outputFilePath = tempDir.resolve("dual-output.txt");
    DualPrintStream dualOut = new DualPrintStream(Files.newOutputStream(outputFilePath), System.out);
    System.setOut(dualOut);
    OUTPUT_LINES.forEach(line -> System.out.println(line));
    assertTrue(outputFilePath.toFile().exists(), "The file exists");
    assertLinesMatch(OUTPUT_LINES, Files.readAllLines(outputFilePath));
    System.setOut(originalOut);
}
```

运行后，程序通过了，这意味着文本已写入文件。此外，我们也可以在控制台中看到这三行数据。

最后，值得注意的是，PrintStream还有其他方法需要我们重写，以保持文件和控制台流同步，例如close()、flush()和write()的其他变体。我们还应该重写checkError()方法来妥善管理IOException。

## 5. 总结

在本文中，我们学习了如何通过替换默认的System.out来使System.out.println()将数据打印到文件。