---
layout: post
title:  在Java中根据MIME类型获取文件扩展名
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

MIME类型是指定互联网上数据的类型和格式的标签，单个MIME类型可以与多个文件扩展名相关联。**例如，“image/jpeg”MIME类型包含“.jpg”、“.jpeg”或“.jpe”等扩展名**。

在本教程中，我们将探索在Java中确定特定[MIME类型](https://www.baeldung.com/java-file-mime-type)的文件扩展名的不同方法，我们将重点介绍解决问题的4种主要方法。

我们的一些实现会在扩展名中包含一个可选的最后一个点，例如，如果我们的MIME类型名称是“image/jpeg”，则字符串“jpg”或“.jpg”将作为文件扩展名返回。

## 2. 使用Apache Tika

[Apache Tika](http://tika.apache.org/)是一个工具包，可检测和提取各种文件中的元数据和文本。它包含一个丰富而强大的API，可用于检测MIME类型的文件扩展名。

让我们首先配置Maven[依赖](https://mvnrepository.com/artifact/org.apache.tika/tika-core)：

```xml
<dependency>
    <groupId>org.apache.tika</groupId>
    <artifactId>tika-core</artifactId>
    <version>2.9.0</version>
</dependency>
```

如前所述，一个MIME类型可以有多个扩展名。为了处理这个问题，MimeType类提供了两个不同的方法：getExtension()和getExtensions()。

**getExtension()方法返回首选的文件扩展名，而getExtensions()返回该MIME类型的所有已知文件扩展名的列表**。

接下来，我们将使用MimeType类中的两种方法来检索扩展：

```java
@Test
public void whenUsingTika_thenGetFileExtension() {
    List<String> expectedExtensions = Arrays.asList(".jpg", ".jpeg", ".jpe", ".jif", ".jfif", ".jfi");
    MimeTypes allTypes = MimeTypes.getDefaultMimeTypes();
    MimeType type = allTypes.forName("image/jpeg");
    String primaryExtension = type.getExtension();
    assertEquals(".jpg", primaryExtension);
    List<String> detectedExtensions = type.getExtensions();
    assertThat(detectedExtensions).containsExactlyElementsOf(expectedExtensions);
}
```

## 3. 使用Jodd Util

我们也可以选用[Jodd Util](https://util.jodd.org/)库，它包含一个用于查找MIME类型的文件扩展名的实用程序。

让我们首先添加Maven[依赖](https://mvnrepository.com/artifact/org.jodd/jodd-util)：

```xml
<dependency>
    <groupId>org.jodd</groupId>
     <artifactId>jodd-util</artifactId>
    <version>6.2.1</version>
</dependency>
```

接下来，**我们将使用findExtensionsByMimeTypes()方法获取所有支持的文件扩展名**：

```java
@Test
public void whenUsingJodd_thenGetFileExtension() {
    List<String> expectedExtensions = Arrays.asList("jpeg", "jpg", "jpe");
    String[] detectedExtensions = MimeTypes.findExtensionsByMimeTypes("image/jpeg", false);
    assertThat(detectedExtensions).containsExactlyElementsOf(expectedExtensions);
}
```

Jodd Util提供一组有限的可识别文件类型和扩展名，它优先考虑简单性而不是全面覆盖。

在findExtensionsByMimeTypes()方法中，我们可以通过将第二个boolean参数设置为true来激活通配符模式。当提供通配符模式作为MIME类型时，我们将获取与指定通配符模式匹配的所有MIME类型的扩展名。

例如，当我们将MIME类型设置为image/\*并启用通配符模式时，我们将获得图像类别内所有MIME类型的扩展。

## 4. 使用SimpleMagic

[SimpleMagic](https://256stuff.com/sources/simplemagic/)是一个实用程序包，其主要用途是检测文件的MIME类型，它还包含一种将MIME类型转换为文件扩展名的方法。

让我们首先添加Maven[依赖](https://mvnrepository.com/artifact/com.j256.simplemagic/simplemagic)：

```xml
<dependency>
    <groupId>com.j256.simplemagic</groupId>
    <artifactId>simplemagic</artifactId>
    <version>1.17</version>
</dependency>
```

现在，我们将**使用ContentInfo类的getFileExtensions()方法来获取所有支持的文件扩展名**：

```java
@Test
public void whenUsingSimpleMagic_thenGetFileExtension() {
    List<String> expectedExtensions = Arrays.asList("jpeg", "jpg", "jpe");
    String[] detectedExtensions = ContentType.fromMimeType("image/jpeg").getFileExtensions();
    assertThat(detectedExtensions).containsExactlyElementsOf(expectedExtensions);
}
```

在SimpleMagic库中，我们有一个枚举ContentType，其中包括MIME类型的映射以及它们对应的文件扩展名和简单名称。getFileExtensions()使用这个枚举，使我们能够根据提供的MIME类型检索文件扩展名。

## 5. 使用自定义的MIME类型到扩展名的Map

我们还可以从MIME类型获取文件扩展名，而无需依赖外部库，我们将创建MIME类型到文件扩展名的自定义Map来实现此目的。

让我们创建一个名为mimeToExtensionMap的HashMap，将MIME类型与其对应的文件扩展名关联起来，**get()方法允许我们在Map中查找所提供MIME类型的预配置文件扩展名并返回它们**：

```java
@Test
public void whenUsingCustomMap_thenGetFileExtension() {
    Map<String, Set<String>> mimeToExtensionMap = new HashMap<>();
    List<String> expectedExtensions = Arrays.asList(".jpg", ".jpe", ".jpeg");
    addMimeExtensions(mimeToExtensionMap, "image/jpeg", ".jpg");
    addMimeExtensions(mimeToExtensionMap, "image/jpeg", ".jpe");
    addMimeExtensions(mimeToExtensionMap, "image/jpeg", ".jpeg");
    Set<String> detectedExtensions = mimeToExtensionMap.get("image/jpeg");
    assertThat(detectedExtensions).containsExactlyElementsOf(expectedExtensions);
}

void addMimeExtensions(Map<String, Set> map, String mimeType, String extension) {
    map.computeIfAbsent(mimeType, k-> new HashSet<>()).add(extension);
}
```

示例Map包含一些示例，但可以根据需要通过添加其他映射轻松进行定制。

## 6. 总结

在本文中，我们探讨了从MIME类型中提取文件扩展名的不同方法。**我们研究了两种不同的方法：利用现有库和根据我们的需求定制自定义逻辑**。

当处理一组有限的MIME类型时，自定义逻辑是一种选择，尽管它可能存在维护挑战。相反，**Apache Tika或Jodd Util等库提供了广泛的MIME类型覆盖范围和易用性**，使它们成为处理各种MIME类型的可靠选择。