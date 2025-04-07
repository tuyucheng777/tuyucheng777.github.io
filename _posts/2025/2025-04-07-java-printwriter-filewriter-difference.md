---
layout: post
title:  Java中的PrintWriter与FileWriter
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

Java标准库提供了文件操作API，[PrintWriter](https://www.baeldung.com/java-write-to-file#write-with-printwriter)和[FileWriter](https://www.baeldung.com/java-filewriter)类可帮助将字符写入文件。但是，这两个类适用于不同的用例。

在本教程中，我们将根据它们的用例探索有关PrintWriter和FileWriter的详细信息。此外，我们还将看到这两个类之间的差异和相似之处。

## 2. PrintWriter

**PrintWriter类有助于将格式化的文本写入文件和控制台等输出流**。

此外，PrintWriter类中的方法不会抛出[IOException](https://www.baeldung.com/java-checked-unchecked-exceptions)。相反，**它有checkError()方法来了解写入操作的状态**。如果写入操作通过，checkError()方法将返回false，如果由于错误而失败，则返回true。

此外，如果在检查错误状态之前流尚未关闭，则checkError()会刷新该流。

此外，PrintWriter还提供了一个名为flush()的方法，用于在写入操作后显式刷新流。但是，当与[try-with-resources](https://www.baeldung.com/java-try-with-resources)块一起使用时，无需显式刷新流。

### 2.1 PrintWriter.println()

println()方法将字符串写入输出流并以新行结尾，它不能将格式化的文本写入输出流。

此外，如果我们决定不使用新行来终止字符串，PrintWriter还提供print()方法。

下面是使用println()方法将字符串写入文件的示例：

```java
@Test
void whenWritingToTextFileUsingPrintWriterPrintln_thenTextMatches() throws IOException {
    String result = "I'm going to Alabama\nAlabama is a state in the US\n";
    try (PrintWriter pw = new PrintWriter("alabama.txt");) {
        pw.println("I'm going to Alabama");
        pw.println("Alabama is a state in the US");
    }
    Path path = Paths.get("alabama.txt");
    String actualData = new String(Files.readAllBytes(path));
    assertEquals(result, actualData);
}
```

这里，我们创建一个PrintWriter对象，以文件路径作为参数。接下来，我们调用PrintWriter对象上的println()方法将字符写入文件。

最后，我们断言预期结果等于文件内容。

**值得注意的是，PrintWriter还提供了一个write()方法将文本写入文件，我们可以用它来代替print()方法**。

### 2.2 PrintWriter.printf()

**[printf()](https://www.baeldung.com/java-printstream-printf#syntax)方法有助于将格式化的文本写入输出流**，我们可以使用[格式说明符](https://www.baeldung.com/java-printstream-printf#conversion_chars)(如%s、%d、.2f等)将不同类型的数据类型写入输出流。

让我们看一些使用printf()将格式化数据写入文件的示例代码：

```java
@Test
void whenWritingToTextFileUsingPrintWriterPrintf_thenTextMatches() throws IOException {
    String result = "Dreams from My Father by Barack Obama";
    File file = new File("dream.txt");
    try (PrintWriter pw = new PrintWriter(file);) {
        String author = "Barack Obama";
        pw.printf("Dreams from My Father by %s", author);
        assertTrue(!pw.checkError());
    }
    try (BufferedReader reader = new BufferedReader(new FileReader(file));) {
        String actualData = reader.readLine();
        assertEquals(result, actualData);
    }
}
```

在上面的代码中，我们将格式化的文本写入文件，我们使用%s标识符直接向文本添加字符串数据类型。

另外，我们创建一个[BufferedReader](https://www.baeldung.com/java-buffered-reader)实例，它以[FileReader](https://www.baeldung.com/java-filereader)对象作为参数来读取文件的内容。

由于该方法不会抛出IOException，因此我们调用checkError()方法来了解写入操作的状态。在本例中，checkError()返回false，表示没有错误。

## 3. FileWriter

FileWriter类扩展了Writer类，**它提供了一种使用预设缓冲区大小将字符写入文件的便捷方法**。

FileWriter不会自动刷新缓冲区，我们需要调用flush()方法。但是，当FileWriter与try-with-resources块一起使用时，它会在退出该块时自动刷新并关闭流。

**此外，当文件丢失或无法打开等情况下，它会抛出IOException**。

与PrintWriter不同，它不能将格式化的文本写入文件。

让我们看一个使用FileWriter类中的write()方法将字符写入文件的示例：

```java
@Test
void whenWritingToTextFileUsingFileWriter_thenTextMatches() throws IOException {
    String result = "Harry Potter and the Chamber of Secrets";
    File file = new File("potter.txt");
    try (FileWriter fw = new FileWriter(file);) {
        fw.write("Harry Potter and the Chamber of Secrets");
    }
    try (BufferedReader reader = new BufferedReader(new FileReader(file));) {
        String actualData = reader.readLine();
        assertEquals(result, actualData);
    }
}
```

在上面的代码中，我们创建了一个[File](https://www.baeldung.com/java-io-file)实例并将其传递给FileWriter对象。接下来，我们调用FileWriter对象上的write()方法将一串字符写入文件。

最后，我们断言预期结果等于文件的内容。

## 4. 总结

在本文中，我们通过示例代码学习了FileWriter和PrintWriter的基本用法。

FileWriter的主要用途是将字符写入文件；但是，PrintWriter具有更多功能，除了文件之外，它还可以写入其他输出流。此外，它还提供了一种将格式化文本写入文件或控制台的方法。