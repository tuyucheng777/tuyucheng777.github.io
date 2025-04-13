---
layout: post
title:  使用Apache Tika进行内容分析
category: libraries
copyright: libraries
excerpt: Apache Tika
---

## 1. 概述

[Apache Tika](https://tika.apache.org/index.html)是一个工具包，用于从各种类型的文档(如Word、Excel和PDF甚至JPEG和MP4等多媒体文件)中提取内容和元数据。

所有基于文本和多媒体的文件都可以使用通用接口进行解析，这使得Tika成为一个强大且多功能的内容分析库。

本文将介绍Apache Tika，包括其解析API以及如何自动检测文档的内容类型，此外，我们还将提供一些示例来演示该库的具体操作。

## 2. 入门

为了使用Apache Tika解析文档，我们只需要一个Maven依赖：

```xml
<dependency>
    <groupId>org.apache.tika</groupId>
    <artifactId>tika-parsers</artifactId>
    <version>1.17</version>
</dependency>
```

该工件的最新版本可以在[这里](https://mvnrepository.com/artifact/org.apache.tika/tika-parsers)找到。

## 3. Parser API

Parser API是 Apache Tika的核心，它抽象出了解析操作的复杂性，此API依赖于一个方法：

```java
void parse(InputStream stream, ContentHandler handler, Metadata metadata, ParseContext context) 
        throws IOException, SAXException, TikaException
```

该方法的参数含义为：

- stream：从要解析的文档创建的InputStream实例
- handler：ContentHandler对象，用于接收从输入文档解析出的一系列XHTML SAX事件；然后，该处理程序将处理事件并以特定形式导出结果
- metadata：Metadata对象，用于在解析器内外传递元数据属性
- context：ParseContext实例，携带特定于上下文的信息，用于定制解析过程

如果parse方法无法从输入流中读取，则会抛出IOException异常；如果无法解析从流中获取的文档，则会抛出TikaException异常；如果处理程序无法处理事件，则会抛出SAXException异常。

在解析文档时，Tika会尽可能地重用现有的解析器库，例如Apache POI或PDFBox。因此，大多数解析器实现类只是这些外部库的适配器。

在第5节中，我们将看到如何使handler和metadata参数来提取文档的内容和元数据。

为了方便起见，我们可以使用门面类Tika来访问Parser API的功能。

## 4. 自动检测

Apache Tika可以根据文档本身而不是附加信息自动检测文档的类型及其语言。

### 4.1 文档类型检测

可以使用Detector接口的实现类来检测文档类型，该接口具有一个方法：

```java
MediaType detect(java.io.InputStream input, Metadata metadata) throws IOException
```

此方法获取文档及其相关元数据-然后返回一个MediaType对象，该对象描述了有关文档类型的最佳猜测。

元数据并非检测器依赖的唯一信息来源。检测器还可以利用“魔法字节”(magic bytes)，即文件开头附近的一种特殊模式，或者将检测过程委托给更合适的检测器。

事实上，检测器所使用的算法是依赖于实现的。

例如，默认检测器首先处理魔法字节，然后处理元数据属性。如果此时尚未找到内容类型，它将使用服务加载器发现所有可用的检测器并依次尝试。

### 4.2 语言检测

除了文档类型之外，Tika还可以在没有元数据信息帮助的情况下识别其语言。

在Tika的早期版本中，使用LanguageIdentifier实例来检测文档的语言。

然而，LanguageIdentifier已被弃用，取而代之的是Web服务，这一点在[入门](https://tika.apache.org/1.17/detection.html#Language_Detection)文档中并未明确说明。

语言检测服务现在通过抽象类LanguageDetector的子类型提供，使用Web服务，你还可以访问功能齐全的在线翻译服务，例如Google Translate或Microsoft Translator。

为了简洁起见，我们不会详细介绍这些服务。

## 5. Tika实践

本节使用工作示例说明Apache Tika的功能。

说明方法将被包装在一个类中：

```java
public class TikaAnalysis {
    // illustration methods
}
```

### 5.1 检测文档类型

下面是我们可以用来检测从InputStream读取的文档类型的代码：

```java
public static String detectDocTypeUsingDetector(InputStream stream) throws IOException {
    Detector detector = new DefaultDetector();
    Metadata metadata = new Metadata();

    MediaType mediaType = detector.detect(stream, metadata);
    return mediaType.toString();
}
```

假设我们在类路径中有一个名为tika.txt的PDF文件，该文件的扩展名已被更改，以试图欺骗我们的分析工具，该文档的真实类型仍然可以通过测试找到并确认：

```java
@Test
public void whenUsingDetector_thenDocumentTypeIsReturned() throws IOException {
    InputStream stream = this.getClass().getClassLoader()
            .getResourceAsStream("tika.txt");
    String mediaType = TikaAnalysis.detectDocTypeUsingDetector(stream);

    assertEquals("application/pdf", mediaType);

    stream.close();
}
```

很明显，由于文件开头的魔法字节%PDF，错误的文件扩展名无法阻止Tika找到正确的媒体类型。

为了方便起见，我们可以使用Tika门面类重写检测代码，结果相同：

```java
public static String detectDocTypeUsingFacade(InputStream stream) throws IOException {
    Tika tika = new Tika();
    String mediaType = tika.detect(stream);
    return mediaType;
}
```

### 5.2 提取内容

现在让我们提取文件的内容并将结果作为字符串返回-使用Parser API：

```java
public static String extractContentUsingParser(InputStream stream) throws IOException, TikaException, SAXException {
    Parser parser = new AutoDetectParser();
    ContentHandler handler = new BodyContentHandler();
    Metadata metadata = new Metadata();
    ParseContext context = new ParseContext();

    parser.parse(stream, handler, metadata, context);
    return handler.toString();
}
```

假设类路径中有一个包含以下内容的Microsoft Word文件：

```text
Apache Tika - a content analysis toolkit
The Apache Tika™ toolkit detects and extracts metadata and text ...
```

可以提取并验证内容：

```java
@Test
public void whenUsingParser_thenContentIsReturned() throws IOException, TikaException, SAXException {
    InputStream stream = this.getClass().getClassLoader()
        .getResourceAsStream("tika.docx");
    String content = TikaAnalysis.extractContentUsingParser(stream);

    assertThat(content, containsString("Apache Tika - a content analysis toolkit"));
    assertThat(content, containsString("detects and extracts metadata and text"));

    stream.close();
}
```

再次，使用Tika类可以更方便地编写代码：

```java
public static String extractContentUsingFacade(InputStream stream) throws IOException, TikaException {
    Tika tika = new Tika();
    String content = tika.parseToString(stream);
    return content;
}
```

### 5.3 提取元数据

除了文档内容之外，解析器API还可以提取元数据：

```java
public static Metadata extractMetadatatUsingParser(InputStream stream) throws IOException, SAXException, TikaException {
    Parser parser = new AutoDetectParser();
    ContentHandler handler = new BodyContentHandler();
    Metadata metadata = new Metadata();
    ParseContext context = new ParseContext();

    parser.parse(stream, handler, metadata, context);
    return metadata;
}
```

当类路径中存在Microsoft Excel文件时，此测试用例确认提取的元数据是正确的：

```java
@Test
public void whenUsingParser_thenMetadataIsReturned() throws IOException, TikaException, SAXException {
    InputStream stream = this.getClass().getClassLoader()
        .getResourceAsStream("tika.xlsx");
    Metadata metadata = TikaAnalysis.extractMetadatatUsingParser(stream);

    assertEquals("org.apache.tika.parser.DefaultParser", metadata.get("X-Parsed-By"));
    assertEquals("Microsoft Office User", metadata.get("Author"));

    stream.close();
}
```

最后，这是使用Tika门面类的提取方法的另一个版本：

```java
public static Metadata extractMetadatatUsingFacade(InputStream stream) throws IOException, TikaException {
    Tika tika = new Tika();
    Metadata metadata = new Metadata();

    tika.parse(stream, metadata);
    return metadata;
}
```

## 6. 总结

本教程重点介绍如何使用Apache Tika进行内容分析，使用Parser和Detector API，我们可以自动检测文档的类型，并提取其内容和元数据。

对于高级用例，我们可以创建自定义解析器和检测器类来更好地控制解析过程。