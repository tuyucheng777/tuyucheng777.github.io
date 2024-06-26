---
layout: post
title:  在Java中获取文件的Mime类型
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

在本教程中，我们将介绍获取文件的IME类型的各种策略。我们将探讨在适用的情况下扩展策略可用的MIME类型的方法。

## 2. 使用Java 7

让我们从Java 7开始-它提供了用于解析MIME类型的方法[Files.probeContentType(path)](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Files.html#probeContentType(java.nio.file.Path))：

```java
@Test
public void whenUsingJava7_thenSuccess() {
    Path path = new File("product.png").toPath();
    String mimeType = Files.probeContentType(path);
 
    assertEquals(mimeType, "image/png");
}
```

此方法使用已安装的FileTypeDetector实现来探测MIME类型。它调用每个实现的probeContentType来解析类型。

现在，如果文件被任何实现识别，则返回内容类型。但是，如果没有发生这种情况，则会调用系统默认文件类型检测器。

**但是，默认实现是特定于操作系统的，并且可能会失败，具体取决于我们使用的操作系统**。

除此之外，还需要注意的是，如果文件不存在于文件系统中，该策略将失败。此外，如果文件没有扩展名，则会导致失败。

##  3. 使用URLConnection

URLConnection提供了多个用于检测文件的MIME类型的API。让我们简要探讨一下它们中的每一个。

### 3.1 使用getContentType()

我们可以使用URLConnection的getContentType()方法来检索文件的MIME类型：

```java
@Test
public void whenUsingGetContentType_thenSuccess(){
    File file = new File("product.png");
    URLConnection connection = file.toURL().openConnection();
    String mimeType = connection.getContentType();
 
    assertEquals(mimeType, "image/png");
}
```

但是，这种方法的一个主要缺点是**速度非常慢**。

### 3.2 使用guessContentTypeFromName()

接下来，让我们看看如何使用guessContentTypeFromName()来实现目的：

```java
@Test
public void whenUsingGuessContentTypeFromName_thenSuccess(){
    File file = new File("product.png");
    String mimeType = URLConnection.guessContentTypeFromName(file.getName());
 
    assertEquals(mimeType, "image/png");
}
```

此方法**使用内部FileNameMap从扩展中解析MIME类型**。

我们还可以选择使用guessContentTypeFromStream()来代替，它使用输入流的前几个字符来确定类型。

### 3.3 使用getFileNameMap()

使用URLConnection获取MIME类型的更快方法是使用getFileNameMap()方法：

```java
@Test
public void whenUsingGetFileNameMap_thenSuccess(){
    File file = new File("product.png");
    FileNameMap fileNameMap = URLConnection.getFileNameMap();
    String mimeType = fileNameMap.getContentTypeFor(file.getName());
 
    assertEquals(mimeType, "image/png");
}
```

该方法返回URLConnection的所有实例使用的MIME类型表。然后使用该表来解析输入文件类型。

当涉及到URLConnection时，内置的MIME类型表非常有限。

默认情况下，**该类使用JRE_HOME/lib中的content-types.properties文件。但是，我们可以通过使用content.types.user.table属性指定特定于用户的表来扩展它**：

```java
System.setProperty("content.types.user.table","<path-to-file>");
```

## 4. 使用MimeTypesFileTypeMap

MimeTypesFileTypeMap使用文件扩展名解析MIME类型。此类随Java 6一起提供，因此在我们使用JDK 1.6时非常方便。

现在让我们看看如何使用它：

```java
@Test
public void whenUsingMimeTypesFileTypeMap_thenSuccess() {
    File file = new File("product.png");
    MimetypesFileTypeMap fileTypeMap = new MimetypesFileTypeMap();
    String mimeType = fileTypeMap.getContentType(file.getName());
 
    assertEquals(mimeType, "image/png");
}
```

在这里，我们可以将文件名或File实例本身作为参数传递给函数。但是，以File实例为参数的函数在内部调用了接收文件名作为参数的重载方法。

**在内部，此方法查找名为mime.types的文件以进行类型解析。请务必注意，该方法按特定顺序搜索文件**：

1.  以编程方式将条目添加到MimetypesFileTypeMap实例
2.  用户主目录中的.mime.types
3.  <java.home\>/lib/mime.types
4.  名为META-INF/mime.types的资源
5.  名为META-INF/mimetypes.default的资源(通常只能在activation.jar文件中找到)

但是，如果没有找到文件，它将返回application/octet-stream作为响应。

## 5. 使用jMimeMagic

[jMimeMagic](https://github.com/arimus/jmimemagic)是一个受限制许可的库，我们可以使用它来获取文件的MIME类型。

让我们从配置Maven依赖项开始：

```xml
<dependency>
    <groupId>net.sf.jmimemagic</groupId>
    <artifactId>jmimemagic</artifactId>
    <version>0.1.5</version>
</dependency>
```

我们可以在[Maven Central](https://mvnrepository.com/artifact/net.sf.jmimemagic/jmimemagic)上找到这个库的最新版本。

接下来，我们将探讨如何使用该库：

```java
@Test    
public void whenUsingJmimeMagic_thenSuccess() {
    File file = new File("product.png");
    Magic magic = new Magic();
    MagicMatch match = magic.getMagicMatch(file, false);
 
    assertEquals(match.getMimeType(), "image/png");
}
```

该库可以处理数据流，因此不需要文件存在于文件系统中。

## 6. 使用Apache Tika

[Apache Tika](https://tika.apache.org/)是一个工具集，可以从各种文件中检测和提取元数据和文本。它具有丰富而强大的API，并附带我们可以使用的[tika-core](https://mvnrepository.com/artifact/org.apache.tika/tika-core)，用于检测文件的MIME类型。

让我们从配置Maven依赖项开始：

```xml
<dependency>
    <groupId>org.apache.tika</groupId>
    <artifactId>tika-core</artifactId>
    <version>1.18</version>
</dependency>
```

接下来，我们将使用detect()方法来解析类型：

```java
@Test
public void whenUsingTika_thenSuccess() {
    File file = new File("product.png");
    Tika tika = new Tika();
    String mimeType = tika.detect(file);
 
    assertEquals(mimeType, "image/png");
}
```

该库依赖流前缀中的魔术标记来进行类型解析。

## 7. 总结

在本文中，我们研究了获取文件MIME类型的各种策略。此外，我们还分析了这些方法的权衡。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-io-1)上获得。
