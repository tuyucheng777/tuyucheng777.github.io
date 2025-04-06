---
layout: post
title:  使用GZip压缩并创建字节数组
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

**[GZIP](https://www.baeldung.com/linux/gzip-and-gunzip)格式是数据[压缩](https://www.baeldung.com/cs/zlib-vs-gzip-vs-zip)中使用的一种文件格式**，Java语言的GZipInputStream和GZipOutputStream类实现了该文件格式。

在本教程中，我们将学习如何使用Java中的GZIP压缩数据。此外，我们还将了解如何将压缩数据写入字节数组。

## 2. GZipOutputStream类

GZipOutputStream类压缩数据并将其写入底层输出流。

### 2.1 对象实例化

**我们可以使用构造函数来创建该类的对象**：

```java
ByteArrayOutputStream os = new ByteArrayOutputStream();
GZIPOutputStream gzipOs = new GZIPOutputStream(os);
```

这里，我们将一个ByteArrayOutputStream对象传递给构造函数。这样，我们稍后就可以使用toByteArray()方法获取字节[数组](https://www.baeldung.com/java-arrays-tutorial)中的压缩数据。

除了ByteArrayOutputStream之外，我们还可以提供其他[OutputStream](https://www.baeldung.com/java-outputstream)实例：

- FileOutputStream：用于将数据存储在文件中
- ServletOutputStream：通过网络传输数据

在这两种情况下，数据都会在进入时被发送到目的地。

### 2.2 压缩数据

**write()方法执行数据压缩**：

```java
byte[] buffer = "Sample Text".getBytes();
gzipOs.write(buffer, 0, buffer.length);
```

write()方法压缩缓冲区字节数组的内容并将其写入包装的输出流。

**除了buffer字节数组之外，write()还包含两个参数，offset和length，它们定义了字节数组内的字节范围**。因此，我们可以使用这些来指定要写入的字节范围，而不是整个缓冲区。

最后，为了完成数据压缩，我们调用close()：

```java
gzipOs.close();
```

**close()方法写入所有剩余数据并关闭流**，因此，调用close()非常重要，否则我们将丢失数据。

## 3. 获取字节数组中的压缩数据

**我们将创建一个使用GZIP进行数据压缩的实用方法**，我们还将了解如何获取包含压缩数据的字节数组。

### 3.1 压缩数据

**让我们创建以GZIP格式压缩数据的gzip()方法**：

```java
private static final int BUFFER_SIZE = 512;

public static void gzip(InputStream is, OutputStream os) throws IOException {
    GZIPOutputStream gzipOs = new GZIPOutputStream(os);
    byte[] buffer = new byte[BUFFER_SIZE];
    int bytesRead = 0;
    while ((bytesRead = is.read(buffer)) > -1) {
        gzipOs.write(buffer, 0, bytesRead);
    }
    gzipOs.close();
}
```

在上述方法中，我们首先创建一个新的GZIPOutputStream实例。然后，我们开始使用缓冲区字节数组从输入流数据。

值得注意的是，我们持续读取字节，直到获得-1返回值。**当我们到达流的末尾时，read()方法返回-1**。

### 3.2 获取包含压缩数据的字节数组

**让我们压缩一个字符串并将结果写入字节数组**，我们将使用之前创建的gzip()方法：

```java
String payload = "This is a sample text to test the gzip method. Have a nice day!";
ByteArrayOutputStream os = new ByteArrayOutputStream();
gzip(new ByteArrayInputStream(payload.getBytes()), os);
byte[] compressed = os.toByteArray();
```

在这里，我们为gzip()方法提供输入和输出流，我们将payload值包装在ByteArrayInputStream对象中。之后，我们创建一个空的ByteArrayOutputStream，gzip()将压缩数据写入其中。

最后，调用gzip()后，我们使用toByteArray()方法获取压缩数据。

## 4. 测试

在测试代码之前，让我们在GZip类中添加gzip()方法。**现在，我们准备用[单元测试](https://www.baeldung.com/java-unit-testing-best-practices)来测试我们的代码**：

```java
@Test
void whenCompressingUsingGZip_thenGetCompressedByteArray() throws IOException {
    String payload = "This is a sample text to test method gzip. The gzip algorithm will compress this string. "
        + "The result will be smaller than this string.";
    ByteArrayOutputStream os = new ByteArrayOutputStream();
    GZip.gzip(new ByteArrayInputStream(payload.getBytes()), os);
    byte[] compressed = os.toByteArray();
    assertTrue(payload.getBytes().length > compressed.length);
    assertEquals("1f", Integer.toHexString(compressed[0] & 0xFF));
    assertEquals("8b", Integer.toHexString(compressed[1] & 0xFF));
}
```

在此测试中，我们压缩一个字符串值，**我们将该字符串转换为ByteArrayInputStream并将其提供给gzip()方法。此外，输出数据将写入ByteArrayOutputStream**。

此外，如果两个条件成立，则测试成功：

1. 压缩后的数据比未压缩后的数据小
2. 压缩的字节数组以1f 8b值开头

对于第二个条件，**GZIP文件以固定值1f 8b开头，以符合GZIP文件格式**。

因此，如果我们运行单元测试，我们将验证两个条件是否都正确。

## 5. 总结

在本文中，我们学习了在Java语言中使用GZIP文件格式时如何在字节数组中获取压缩数据。为此，我们创建了一个用于压缩的实用方法。