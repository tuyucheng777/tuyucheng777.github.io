---
layout: post
title:  如何在Java中将字符串写入OutputStream
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

当我们需要将数据传输到外部目标(例如文件和网络)时，我们经常使用[OutputStream](https://www.baeldung.com/java-outputstream)，数据可以是二进制或字符串格式。**我们使用OutputStream(字节流)来处理二进制数据，使用Writer(字符流)来处理字符串数据**。

但是，由于所选API的限制，在某些情况下我们必须将字符串写入OutputStream。在某些情况下，API可能仅提供OutputStream而不是Writer。在本教程中，我们将探讨在这种情况下将字符串写入OutputStream的不同方法。

## 2. 字节转换

**最直观的方法是将字符串转换为字节，然后将转换后的字节写入OutputStream**：

```java
String str = "Hello";
byte[] bytes = str.getBytes(StandardCharsets.UTF_8);
outputStream.write(bytes);
```

这种方法很简单，但有一个很大的缺点，我们需要事先将字符串显式转换为字节，并在每次调用getBytes()时指定[字符编码](https://www.baeldung.com/java-char-encoding)，这使我们的代码变得繁琐。

## 3. OutputStreamWriter

**更好的方法是利用[OutputStreamWriter](https://www.baeldung.com/java-outputstream#writing-text-with-outputstreamwriter)包装我们的OutputStream**，OutputStreamWriter充当将字符流转换为字节流的包装器，写入其中的字符串使用所选的字符编码编码为字节。

我们通过write()方法写入字符串，而无需指定字符编码。每次调用OutputStreamWriter上的write()都会隐式将字符串转换为编码字节，从而提供更简化的过程：

```java
try (OutputStreamWriter writer = new OutputStreamWriter(outputStream, StandardCharsets.UTF_8)) {
    writer.write("Hello");
}
```

如果包装的OutputStream尚未缓冲，我们建议使用[BufferedWriter](https://www.baeldung.com/java-list-to-text-file#using-bufferedwriter)来包装OutputStreamWriter，以使写入过程更加高效：

```java
BufferedWriter bufferedWriter = new BufferedWriter(writer);
```

通过首先将数据写入缓冲区，然后将缓冲的数据以更大的块形式刷新到流中，可以减少IO操作。

## 4. PrintStream

**另一个选择是利用PrintStream，它提供与OutputStreamWriter类似的功能**。与OutputStreamWriter类似，我们可以在实例化PrintStream时指定编码：

```java
try (PrintStream printStream = new PrintStream(outputStream, true, StandardCharsets.UTF_8)) {
    printStream.print("Hello");
}
```

如果我们没有在构造函数中明确定义字符编码，则默认编码为[UTF-8](https://www.baeldung.com/java-char-encoding#2-utf-8)：

```java
PrintStream printStream = new PrintStream(outputStream);
```

PrintStream和OutputStreamWriter之间的区别在于PrintStream提供了额外的print()方法，用于将不同类型的数据类型写入OutputStream。此外，PrintWriter的print()和write()方法从不抛出IOException：

```java
printStream.print(100); // integer
printStream.print(true); // boolean
```

## 5. PrintWriter

[PrintWriter](https://www.baeldung.com/java-printstream-vs-printwriter)的用途与PrintStream类似，提供将格式化的数据表示写入OutputStream的功能：

```java
try (PrintWriter writer = new PrintWriter(outputStream)) {
    writer.print("Hello");
}
```

除了包装OutputStream之外，PrintWriter还提供了其他构造函数来包装Writer。它们之间的另一个区别是PrintWriter提供了write()方法来写入字符数组，而PrintStream提供了write()方法来写入字节数组：

```java
char[] c = new char[] {'H', 'e', 'l' ,'l', 'o'};
try (PrintWriter writer = new PrintWriter(new StringWriter())) {
    writer.write(c);
}
```

## 6. 总结

在本文中，我们探讨了在Java中将字符串写入OutputStream的几种方法。

我们首先将字符串直接转换为字节，这需要对每个写入操作进行显式编码，并且会影响代码的可维护性。随后，我们探索了3种不同的Java类，它们环绕OutputStream以无缝地将字符串编码为字节。