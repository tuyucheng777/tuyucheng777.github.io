---
layout: post
title:  Java将PDF转换为Base64
category: libraries
copyright: libraries
excerpt: Apache Commons Codec
---

## 1. 概述

在这个简短的教程中，我们将了解**如何使用Java 8和Apache Commons Codec对PDF文件进行Base64编码和解码**。

首先，让我们快速了解一下Base64的基础知识。

## 2. Base64基础知识

在线上发送数据时，我们需要以二进制格式发送，但如果我们只发送[0和1](https://www.baeldung.com/cs/two-complement)，不同的传输层协议可能会对它们进行不同的解释，导致数据在传输过程中损坏。

因此，**为了在传输二进制数据时具有可移植性和通用标准，Base64应运而生**。

由于发送方和接收方都理解并同意使用该标准，因此我们的数据丢失或被误解的可能性大大降低。

现在让我们看看将其应用于PDF的几种方法。

## 3. 使用Java 8进行转换

从Java 8开始，引入了一个实用程序[java.util.Base64](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Base64.html)，它为Base64编码方案提供编码器和解码器。它支持[RFC 4648](https://www.ietf.org/rfc/rfc4648.txt)和[RFC 2045](https://www.ietf.org/rfc/rfc2045.txt)中指定的Basic、URL安全以及MIME类型。

### 3.1 编码

要将PDF转换为Base64，我们首先需要以字节为单位获取它，然后**通过java.util.Base64.Encoder的编码方法传递它**：

```java
byte[] inFileBytes = Files.readAllBytes(Paths.get(IN_FILE)); 
byte[] encoded = java.util.Base64.getEncoder().encode(inFileBytes);
```

这里，IN_FILE是我们输入PDF的路径。

### 3.2 流编码

对于较大的文件或内存有限的系统，**使用流执行编码比读取内存中的所有数据效率更高**，让我们看看如何实现这一点：

```java
try (OutputStream os = java.util.Base64.getEncoder().wrap(new FileOutputStream(OUT_FILE)); FileInputStream fis = new FileInputStream(IN_FILE)) {
    byte[] bytes = new byte[1024];
    int read;
    while ((read = fis.read(bytes)) > -1) {
        os.write(bytes, 0, read);
    }
}
```

这里，IN_FILE是输入PDF的路径，OUT_FILE是包含Base64编码文档的文件路径。我们不是将整个PDF读入内存，然后在内存中对整个文档进行编码，而是一次读取最多1KB的数据，并将该数据通过编码器传递到OutputStream中。

### 3.3 解码

在接收端，我们得到编码的文件。

因此我们**现在需要对其进行解码以取回原始字节，并将它们写入FileOutputStream以获取解码后的PDF**：

```java
byte[] decoded = java.util.Base64.getDecoder().decode(encoded);

FileOutputStream fos = new FileOutputStream(OUT_FILE);
fos.write(decoded);
fos.flush();
fos.close();
```

这里，OUT_FILE是我们要创建的PDF的路径。

## 4. 使用Apache Commons进行转换

接下来，我们将使用Apache Commons Codec包来实现相同的功能；它基于[RFC 2045](https://www.ietf.org/rfc/rfc2045.txt)，并且早于我们之前讨论过的Java 8实现。因此，当我们需要支持多个JDK版本(包括旧版本)或供应商时，它作为第三方API非常方便。

### 4.1 Maven

为了能够使用Apache库，我们需要在pom.xml中添加依赖：

```xml
<dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
    <version>1.16.0</version>
</dependency>
```

上述内容的最新版本可以在[Maven Central](https://mvnrepository.com/artifact/commons-codec)上找到。

### 4.2 编码

步骤与Java 8相同，只是这次我们将原始字节传递给[org.apache.commons.codec.binary.Base64](https://commons.apache.org/proper/commons-codec/apidocs/org/apache/commons/codec/binary/Base64.html)类的encodeBase64方法：

```java
byte[] inFileBytes = Files.readAllBytes(Paths.get(IN_FILE));
byte[] encoded = org.apache.commons.codec.binary.Base64.encodeBase64(inFileBytes);
```

### 4.3 流编码

该库不支持流编码。

### 4.4 解码

再次，我们只需调用decodeBase64方法并将结果写入文件：

```java
byte[] decoded = org.apache.commons.codec.binary.Base64.decodeBase64(encoded);

FileOutputStream fos = new FileOutputStream(OUT_FILE);
fos.write(decoded);
fos.flush();
fos.close();
```

## 5. 测试

现在我们将使用一个简单的JUnit测试来测试我们的编码和解码：

```java
public class EncodeDecodeUnitTest {

    private static final String IN_FILE = // path to file to be encoded from;
    private static final String OUT_FILE = // path to file to be decoded into;
    private static byte[] inFileBytes;

    @BeforeClass
    public static void fileToByteArray() throws IOException {
        inFileBytes = Files.readAllBytes(Paths.get(IN_FILE));
    }

    @Test
    public void givenJavaBase64_whenEncoded_thenDecodedOK() throws IOException {
        byte[] encoded = java.util.Base64.getEncoder().encode(inFileBytes);
        byte[] decoded = java.util.Base64.getDecoder().decode(encoded);
        writeToFile(OUT_FILE, decoded);

        assertNotEquals(encoded.length, decoded.length);
        assertEquals(inFileBytes.length, decoded.length);
        assertArrayEquals(decoded, inFileBytes);
    }

    @Test
    public void givenJavaBase64_whenEncodedStream_thenDecodedStreamOK() throws IOException {
        try (OutputStream os = java.util.Base64.getEncoder().wrap(new FileOutputStream(OUT_FILE));
             FileInputStream fis = new FileInputStream(IN_FILE)) {
            byte[] bytes = new byte[1024];
            int read;
            while ((read = fis.read(bytes)) > -1) {
                os.write(bytes, 0, read);
            }
        }

        byte[] encoded = java.util.Base64.getEncoder().encode(inFileBytes);
        byte[] encodedOnDisk = Files.readAllBytes(Paths.get(OUT_FILE));
        assertArrayEquals(encoded, encodedOnDisk);

        byte[] decoded = java.util.Base64.getDecoder().decode(encoded);
        byte[] decodedOnDisk = java.util.Base64.getDecoder().decode(encodedOnDisk);
        assertArrayEquals(decoded, decodedOnDisk);
    }

    @Test
    public void givenApacheCommons_givenJavaBase64_whenEncoded_thenDecodedOK() throws IOException {
        byte[] encoded = org.apache.commons.codec.binary.Base64.encodeBase64(inFileBytes);
        byte[] decoded = org.apache.commons.codec.binary.Base64.decodeBase64(encoded);

        writeToFile(OUT_FILE, decoded);

        assertNotEquals(encoded.length, decoded.length);
        assertEquals(inFileBytes.length, decoded.length);

        assertArrayEquals(decoded, inFileBytes);
    }

    private void writeToFile(String fileName, byte[] bytes) throws IOException {
        FileOutputStream fos = new FileOutputStream(fileName);
        fos.write(bytes);
        fos.flush();
        fos.close();
    }
}
```

可以看到，我们首先在@BeforeClass方法中读取输入字节，并在两个@Test方法中验证：

- 编码和解码的字节数组的长度不同
- inFileBytes和解码后的字节数组具有相同的长度和相同的内容

当然，我们也可以打开我们创建的解码后的PDF文件，看看其内容是否与我们输入的文件相同。

## 6. 总结

在本快速教程中，我们了解了有关[Java的Base64实用程序](https://www.baeldung.com/java-base64-encode-and-decode)的更多信息。

我们还介绍了使用Java 8和Apache Commons Codec将PDF转换为Base64或从Base64转换为PDF的代码示例，有趣的是，JDK实现比Apache实现快得多。