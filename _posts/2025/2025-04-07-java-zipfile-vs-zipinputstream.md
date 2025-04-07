---
layout: post
title:  Java中ZipFile和ZipInputStream之间的区别
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

有时我们需要压缩文件，将多个文件打包成一个存档，以方便传输和节省空间。对于这种用例，**Zip是一种广泛使用的压缩存档文件格式**。

Java提供了一组标准类，例如ZipFile和ZipInputStream，用于访问zip文件。在本教程中，我们将学习如何使用它们读取zip文件。此外，我们将探索它们的功能差异并评估它们的性能。

## 2. 创建一个Zip文件

在我们深入研究读取zip文件的代码之前，让我们先回顾一下[创建zip文件](https://www.baeldung.com/java-compress-and-uncompress#zip_file)的过程。

在下面的代码片段中，我们将有两个变量，data存储要压缩的内容，file代表我们的目标文件：

```java
String data = "..."; // a very long String

try (BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream(file)); ZipOutputStream zos = new ZipOutputStream(bos)) {
    ZipEntry zipEntry = new ZipEntry("zip-entry.txt");
    zos.putNextEntry(zipEntry);
    zos.write(data);
    zos.closeEntry();
}
```

**此代码片段将数据存档到名为zip-entry.txt的zip条目中，然后将该条目写入目标文件**。

## 3. 通过ZipFile读取

首先，让我们看看如何通过[ZipFile](https://www.baeldung.com/java-read-zip-files)类从zip文件读取所有条目：

```java
try (ZipFile zipFile = new ZipFile(compressedFile)) {
    Enumeration<? extends ZipEntry> zipEntries = zipFile.entries();
    while (zipEntries.hasMoreElements())  {
        ZipEntry zipEntry = zipEntries.nextElement();
        try (InputStream inputStream = new BufferedInputStream(zipFile.getInputStream(zipEntry))) {
            // Read data from InputStream
        }
    }
}
```

我们创建一个ZipFile实例来读取压缩文件，ZipFile.entries()返回zip文件中的所有zip条目，然后我们可以从ZipEntry获取InputStream来读取其内容。

**除了entries()之外，ZipFile还有一个方法getEntry(...)，它允许我们根据条目名称随机访问特定的ZipEntry**：

```java
ZipEntry zipEntry = zipFile.getEntry("str-data-10.txt");
try (InputStream inputStream = new BufferedInputStream(zipFile.getInputStream(zipEntry))) {
    // Read data from InputStream
}
```

## 4. 通过ZipInputStream读取

接下来，我们将介绍一个通过ZipInputStream从zip文件读取所有条目的典型示例：

```java
try (BufferedInputStream bis = new BufferedInputStream(new FileInputStream(compressedFile)); ZipInputStream zipInputStream = new ZipInputStream(bis)) {
    ZipEntry zipEntry;
    while ((zipEntry = zipInputStream.getNextEntry()) != null) {
        // Read data from ZipInputStream
    }
}
```

我们创建一个ZipInputStream来包装数据源，在我们的例子中是compressedFile。之后，我们通过getNextEntry()迭代ZipInputStream。

在循环中，我们通过从ZipInputStream读取数据来读取每个ZipEntry的数据。一旦我们完成对条目的读取，我们再次调用getNextEntry()来表示我们将要读取下一个条目。

## 5. 功能差异

虽然这两个类都可以用于从zip文件中读取条目，但它们有两个明显的功能差异。

### 5.1 访问类型

它们之间的主要区别是ZipFile支持随机访问，而ZipInputStream仅支持顺序访问。

在ZipFile中，我们可以通过调用ZipFile.getEntry(...)来提取特定条目，当我们只需要ZipFile中的特定条目时，此功能特别有用。如果我们想在ZipInputStream中实现相同的功能，我们必须循环遍历每个ZipEntry，直到在迭代过程中找到匹配项。

### 5.2 数据源

ZipFile要求数据源是物理文件，而ZipInputStream只需要InputStream。可能存在我们的数据不是文件的情况；例如，我们的数据来自网络流。在这种情况下，我们必须将整个InputStream转换为文件，然后才能使用ZipFile处理它。

## 6. 性能比较

我们已经了解了ZipFile和ZipInputStream之间的功能差异，现在，让我们进一步探讨性能方面的差异。

我们将使用[JMH](https://www.baeldung.com/java-microbenchmark-harness)来捕捉这两者之间的处理速度，JMH是一个用于测量代码片段性能的框架。

在进行基准测试之前，我们必须在pom.xml中包含以下Maven依赖：

```xml
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-core</artifactId>
    <version>1.37</version>
</dependency>
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-generator-annprocess</artifactId>
    <version>1.37</version>
</dependency>
```

可以在Maven Central找到最新版本的JMH [Core](https://mvnrepository.com/artifact/org.openjdk.jmh/jmh-core)和[Annotation](https://mvnrepository.com/artifact/org.openjdk.jmh/jmh-generator-annprocess)。

### 6.1 读取所有条目

在本实验中，我们旨在评估从zip文件读取所有条目的性能。在我们的设置中，我们有一个包含10个条目的zip文件，每个条目包含200KB的数据，我们将分别通过ZipFile和ZipInputStream读取它们：

|   类   | 运行时间(毫秒)|
|:-----:| :--------------: |
| ZipFile  |      11.072      |
| ZipInputStream |      11.642      |

**从结果来看，我们看不出这两个类之间有任何显著的性能差异，运行时间方面的差异在10%以内。在读取zip文件中的所有条目时，它们都表现出相当的效率**。

### 6.2 读取最后一条记录

接下来，我们将专门读取同一个zip文件中的最后一项：

|       类        | 运行时间(毫秒)|
|:--------------:| :--------------: |
|    ZipFile     |      1.016       |
| ZipInputStream |      12.830      |

这次它们之间有很大的区别，ZipFile读取10个条目中的单个条目所需的时间仅为读取所有条目所需的时间的1/10，而ZipInputStream所花的时间几乎相同。

我们可以观察到ZipInputStream从结果中按顺序读取条目，输入流必须从zip文件的开头读取，直到找到目标条目，而ZipFile允许跳转到目标条目而无需读取整个文件。

**结果表明选择ZipFile而不是ZipInputStream非常重要，特别是在处理大量条目中的少量条目时**。

## 7. 总结

在软件开发中，通常使用zip来处理压缩文件，Java提供了两个不同的类ZipFile和ZipIputStream来读取zip文件。

在本文中，我们探讨了它们的用法和功能差异，并评估了它们之间的性能。

它们之间的选择取决于我们的要求，当我们处理大型zip存档中的有限数量条目时，我们将选择ZipFile以确保最佳性能。相反，如果我们的数据源不是文件，我们将选择ZipInputStream。