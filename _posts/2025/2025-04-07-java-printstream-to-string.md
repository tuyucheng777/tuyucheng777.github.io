---
layout: post
title:  PrintStream转String
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

在这个简短的教程中，我们将阐明如何在Java中**将PrintStream转换为String**。

我们将从使用核心Java方法开始，然后，我们将了解如何使用外部库(例如Apache Commons IO)实现相同的目标。

## 2. 什么是PrintStream

在Java中，[PrintStream](https://www.baeldung.com/java-printstream-printf)是一种[输出流](https://www.baeldung.com/java-outputstream)，它提供了一种方便的方法来打印和格式化数据。它带有一组用于打印和格式化不同类型数据的方法，例如println()和printf()。

与其他输出流不同，它从不抛出IOException。但是，如果出现错误，它会设置一个标志，可以通过checkError()方法进行测试。

现在我们知道了什么是PrintStream，让我们看看如何将其转换为字符串。

## 3. 使用ByteArrayOutputStream

简而言之，ByteArrayOutputStream是一个将数据写入字节数组的输出流。

通常，**我们可以使用它来捕获PrintStream的输出，然后将捕获的字节转换为字符串**。那么，让我们看看实际效果：

```java
public static String usingByteArrayOutputStreamClass(String input) throws IOException {
    if (input == null) {
        return null;
    }

    String output;
    try (ByteArrayOutputStream outputStream = new ByteArrayOutputStream(); PrintStream printStream = new PrintStream(outputStream)) {
        printStream.print(input);

        output = outputStream.toString();
    }

    return output;
}
```

我们可以看到，我们创建了一个PrintStream对象，并将ByteArrayOutputStream传递到构造函数中。

然后，我们使用print()方法将输入字符串写入PrintStream。

最后，我们使用ByteArrayOutputStream类的toString()方法将输入转换为String对象。

现在，让我们使用测试用例来确认这一点：

```java
@Test
public void whenUsingByteArrayOutputStreamClass_thenConvert() throws IOException {
    assertEquals("test", PrintStreamToStringUtil.usingByteArrayOutputStreamClass("test"));
    assertEquals("", PrintStreamToStringUtil.usingByteArrayOutputStreamClass(""));
    assertNull(PrintStreamToStringUtil.usingByteArrayOutputStreamClass(null));
}
```

如上所示，我们的方法将PrintStream转换为字符串。

## 4. 使用自定义输出流

另一个解决方案是使用OutputStream类的自定义实现。

基本上，OutputStream是表示字节输出流的所有类的超类，包括ByteArrayOutputStream。

首先，让我们考虑一下CustomOutputStream[静态内部类](https://www.baeldung.com/java-static#1-example-of-static-class)：

```java
private static class CustomOutputStream extends OutputStream {

    private StringBuilder string = new StringBuilder();

    @Override
    public void write(int b) throws IOException {
        this.string.append((char) b);
    }

    @Override
    public String toString() {
        return this.string.toString();
    }
}
```

这里，我们使用StringBuilder实例逐字节写入给定的数据。此外，我们重写了toString()方法来获取StringBuilder对象的字符串表示形式。

接下来，让我们重用上一节中的相同示例。**但是，我们将使用自定义实现而不是ByteArrayOutputStream**：

```java
public static String usingCustomOutputStream(String input) throws IOException {
    if (input == null) {
        return null;
    }

    String output;
    try (CustomOutputStream outputStream = new CustomOutputStream(); PrintStream printStream = new PrintStream(outputStream)) {
        printStream.print(input);

        output = outputStream.toString();
    }

    return output;
}
```

现在，让我们添加另一个测试用例来确认一切是否按预期工作：

```java
@Test
public void whenCustomOutputStream_thenConvert() throws IOException {
    assertEquals("world", PrintStreamToStringUtil.usingCustomOutputStream("world"));
    assertEquals("", PrintStreamToStringUtil.usingCustomOutputStream(""));
    assertNull(PrintStreamToStringUtil.usingCustomOutputStream(null));
}
```

## 5. 使用Apache Commons IO

或者，我们可以使用[Apache Commons IO](https://www.baeldung.com/apache-commons-io)库来实现相同的目标。

首先，让我们将[Apache Commons IO依赖](https://mvnrepository.com/artifact/commons-io/commons-io/2.11.0)添加到pom.xml中：

```xml
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.15.1</version>
</dependency>
```

Apache Commons IO提供了自己的[ByteArrayOutputStream](https://commons.apache.org/proper/commons-io/javadocs/api-2.5/org/apache/commons/io/output/ByteArrayOutputStream.html)版本，**该类附带toByteArray()方法，用于将数据检索为字节数组**。

让我们在实践中看看：

```java
public static String usingApacheCommonsIO(String input) {
    if (input == null) {
        return null;
    }

    org.apache.commons.io.output.ByteArrayOutputStream outputStream = new org.apache.commons.io.output.ByteArrayOutputStream();
    try (PrintStream printStream = new PrintStream(outputStream)) {
        printStream.print(input);
    }

    return new String(outputStream.toByteArray());
}
```

简而言之，我们使用toByteArray()从输出流中获取字节数组。然后，我们将返回的数组传递给String构造函数。

这里的一个重要警告是，与Java相反，我们不需要关闭ByteArrayOutputStream。

该解决方案也能正常工作，如单元测试所示：

```java
@Test
public void whenUsingApacheCommonsIO_thenConvert() {
    assertEquals("hello", PrintStreamToStringUtil.usingApacheCommonsIO("hello"));
    assertEquals("", PrintStreamToStringUtil.usingApacheCommonsIO(""));
    assertNull(PrintStreamToStringUtil.usingApacheCommonsIO(null));
}
```

## 6. 总结

在本文中，我们学习了如何将PrintStream转换为String。

在此过程中，我们解释了如何使用核心Java方法来实现这一点。然后，我们说明了如何使用Apache Commons IO等外部库。