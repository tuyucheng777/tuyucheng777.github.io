---
layout: post
title:  Java InputStream与InputStreamReader
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

在本文中，我们将讨论[InputStream](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/io/InputStream.html)类以及它如何处理来自各种来源的二进制信息，我们还将讨论[InputStreamReader](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/io/InputStreamReader.html)类及其与InputStream的区别。

## 2. 什么是InputStream？

**InputStream是一个从源中读取字节形式的二进制数据的类**，因为它是一个抽象类，所以我们只能通过它的子类([FileInputStream](https://www.baeldung.com/reading-file-in-java)和[ByteArrayInputStream](https://www.baeldung.com/convert-byte-array-to-input-stream)等)来实例化它。

## 3. 什么是InputStreamReader？

与InputStream类相比，InputStreamReader直接处理字符或文本。**它使用给定的InputStream读取字节，然后根据特定的[Charset](https://www.baeldung.com/java-char-encoding)将它们转换为字符**。我们可以显式设置Charset，其中一些是UTF-8、UTF-16等，或者依赖于JVM的默认字符集：

```java
@Test
public void givenAStringWrittenToAFile_whenReadByInputStreamReader_thenShouldMatchWhenRead(@TempDir Path tempDir) throws IOException {
    String sampleTxt = "Good day. This is just a test. Good bye.";
    Path sampleOut = tempDir.resolve("sample-out.txt");
    List<String> lines = Arrays.asList(sampleTxt);
    Files.write(sampleOut, lines);
    String absolutePath = String.valueOf(sampleOut.toAbsolutePath());
    try (InputStreamReader reader = new InputStreamReader(new FileInputStream(absolutePath), StandardCharsets.UTF_8)) {
        boolean isMatched = false;
        int b;
        StringBuilder sb = new StringBuilder();
        while ((b = reader.read()) != -1) {
            sb.append((char) b);
            if (sb.toString().contains(sampleTxt)) {
                isMatched = true;
                break;
            }
        }
        assertThat(isMatched).isTrue();
    }
}
```

上面的代码片段演示了如何使用StandardCharsets.UTF_8常量明确设置InputStreamReader的编码。

我们的FileInputStream是InputStream的一种类型，被InputStreamReader包裹着。因此，我们可以看到InputStreamReader将InputStream解释为文本而不是原始字节信息。

## 4. InputStreamReader与InputStream

**InputStreamReader是从字节流到字符流的桥梁**，此类接收一个InputStream实例，读取字节，并使用字符编码将其解码为字符。**它有一个read()方法，用于读取单个字符**。此方法通过读取底层InputStream中当前位置之前的一个或多个字节，将字节转换为字符，**当到达流的末尾时，它返回-1**。

相比之下，**InputStream是所有表示字节输入流的类的超类**，此类是InputStreamReader的主要构造函数参数，这意味着InputStream的任何子类都是InputStreamReader的有效字节源。

InputStream类也有一个read()方法，用于读取单个字节。**但是，InputStream.read()方法不会将字节解码为字符，而InputStreamReader.read()则会**。

## 5. 总结

在本文中，我们讨论了InputStream和InputStreamReader。**InputStream是一个抽象类，它有多个子类，这些子类倾向于特定形式的二进制数据，例如FileInputStream和ByteArrayInputStream等**。相比之下，InputStreamReader从InputStream读取字节并将其转换为指定编码的字符。

这两个类之间的区别很明显，当我们需要处理二进制数据时，我们应该使用InputStream；如果我们需要处理字符流，那么使用InputStreamReader会更好。

InputStream是创建InputStreamReader所需的主要构造函数参数。