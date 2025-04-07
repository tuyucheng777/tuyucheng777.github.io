---
layout: post
title:  读取文件时删除BOM字符
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 简介

[字节顺序标记](https://www.baeldung.com/linux/file-utf8-remove-byte-order-mark)(BOM)表示文件的编码，但如果我们处理不当，可能会导致问题，尤其是在处理文本数据时。此外，在读取文本文件时遇到以BOM字符开头的文件并不罕见。

**在本教程中，我们将探讨如何在Java中读取文件时检测和删除BOM字符，特别关注UTF-8编码**。

## 2. 了解BOM字符

BOM字符是一种特殊的Unicode字符，用于指示文本文件或流的字节顺序(字节顺序)。对于UTF-8，BOM为EF BB BF(0xEF 0xBB 0xBF)。

**虽然BOM字符对于编码检测很有用，但如果没有正确删除，它可能会干扰文本处理**。

## 3. 使用InputStream和Reader

处理BOM的传统方法涉及使用Java中的[InputStream](https://www.baeldung.com/convert-input-stream-to-string)和[Reader](https://www.baeldung.com/reading-file-in-java)，这种方法让我们可以在处理文件内容之前手动检测并从输入流中删除BOM。

首先我们要完整地读取一个文件的内容，如下：

```java
private String readFully(Reader reader) throws IOException {
    StringBuilder content = new StringBuilder();
    char[] buffer = new char[1024];
    int numRead;
    while ((numRead = reader.read(buffer)) != -1) {
        content.append(buffer, 0, numRead);
    }
    return content.toString();
}
```

这里，我们利用StringBuilder来累积从Reader读取的内容，通过反复将字符块读入缓冲区数组并将它们追加到StringBuilder，我们确保捕获了文件的全部内容。最后，将累积的内容作为字符串返回。

现在，让我们在测试用例中应用readFully()方法来演示如何使用InputStream和Reader有效地处理BOM：

```java
@Test
public void givenFileWithBOM_whenUsingInputStreamAndReader_thenRemoveBOM() throws IOException {
    try (InputStream is = new FileInputStream(filePath)) {
        byte[] bom = new byte[3];
        int n = is.read(bom, 0, bom.length);

        Reader reader;
        if (n == 3 && (bom[0] & 0xFF) == 0xEF && (bom[1] & 0xFF) == 0xBB && (bom[2] & 0xFF) == 0xBF) {
            reader = new InputStreamReader(is, StandardCharsets.UTF_8);
        } else {
            reader = new InputStreamReader(new FileInputStream(filePath), StandardCharsets.UTF_8);
        }
        assertEquals(expectedContent, readFully(reader));
    }
}
```

在此方法中，我们首先使用类加载器的资源设置文件路径并处理潜在的URI语法异常。然后，我们利用FileInputStream打开文件的InputStream，并使用InputStreamReader创建具有UTF-8编码的Reader。

此外，我们利用输入流的read()方法将前3个字节读入字节数组，以检查是否存在BOM。

**如果系统检测到UTF-8 BOM(0xEF、0xBB、0xBF)，它会跳过它并使用我们之前定义的readFully()方法断言内容。否则，我们通过创建一个具有UTF-8编码的新InputStreamReader并执行相同的断言来重置流**。

## 4. 使用Apache Commons IO

[Apache Commons IO](https://www.baeldung.com/apache-commons-io)提供了一种替代手动检测和删除BOM的方法，该库提供了用于常见I/O操作的各种实用程序。这些实用程序之一是[BOMInputStream](https://www.baeldung.com/convert-string-to-input-stream)类，它通过自动检测和从输入流中删除BOM来简化BOM的处理。

以下是我们应用此方法的示例：

```java
@Test
public void givenFileWithBOM_whenUsingApacheCommonsIO_thenRemoveBOM() throws IOException {
    try (BOMInputStream bomInputStream = new BOMInputStream(new FileInputStream(filePath));
         Reader reader = new InputStreamReader(bomInputStream, StandardCharsets.UTF_8)) {

        assertTrue(bomInputStream.hasBOM());
        assertEquals(expectedContent, readFully(reader));
    }
}
```

在此测试用例中，我们用BOMInputStream包装FileInputStream，自动检测并删除输入流中的任何BOM。此外，我们使用assertTrue()方法检查是否已使用hasBOM()方法成功检测到并删除了BOM。

然后，我们使用BOMInputStream创建一个Reader，并使用readFully()方法断言内容，以确保内容与预期内容匹配，而不受BOM的影响。

## 5. 使用NIO(新I/O)

Java的[NIO](https://www.baeldung.com/java-nio-vs-nio-2)(新I/O)包提供了高效的文件处理功能，包括支持将文件内容读入内存缓冲区。利用NIO，我们可以使用[ByteBuffer](https://www.baeldung.com/java-bytebuffer)和[Files](https://www.baeldung.com/java-io-file)类检测并删除文件中的BOM。

下面我们来看一下如何使用NIO处理BOM来实现测试用例：

```java
@Test
public void givenFileWithBOM_whenUsingNIO_thenRemoveBOM() throws IOException, URISyntaxException {
    byte[] fileBytes = Files.readAllBytes(Paths.get(filePath));
    ByteBuffer buffer = ByteBuffer.wrap(fileBytes);

    if (buffer.remaining() >= 3) {
        byte b0 = buffer.get();
        byte b1 = buffer.get();
        byte b2 = buffer.get();

        if ((b0 & 0xFF) == 0xEF && (b1 & 0xFF) == 0xBB && (b2 & 0xFF) == 0xBF) {
            assertEquals(expectedContent, StandardCharsets.UTF_8.decode(buffer).toString());
        } else {
            buffer.position(0);
            assertEquals(expectedContent, StandardCharsets.UTF_8.decode(buffer).toString());
        }
    } else {
        assertEquals(expectedContent, StandardCharsets.UTF_8.decode(buffer).toString());
    }
}
```

在此测试用例中，我们使用readAllBytes()方法将文件内容读入ByteBuffer。然后，我们通过检查缓冲区的前3个字节来检查是否存在BOM，如果检测到UTF-8 BOM，我们将跳过它；否则，我们将重置缓冲区位置。

## 6. 总结

总之，通过使用不同的Java库和技术，处理文件读取操作中的BOM变得简单，并确保文本处理的流畅。