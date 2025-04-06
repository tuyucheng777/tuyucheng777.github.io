---
layout: post
title:  在Java中将文件转换为字节数组
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

在本快速教程中，我们将了解如何在Java中将文件转换为字节数组。

首先，我们将学习如何使用内置JDK解决方案来实现这一点。然后，我们将讨论如何使用Apache Commons IO和Guava实现相同的结果。

## 2. 使用Java

JDK提供了几种方便的方法将文件转换为字节数组。例如，我们可以使用[java.io](https://www.baeldung.com/java-io)或[java.nio](https://www.baeldung.com/tag/java-nio)包来解决我们的核心问题。那么，让我们仔细看看每个选项。

### 2.1 FileInputStream

让我们从最简单的解决方案开始，使用IO包中的FileInputStream类。通常，**此类附带以字节形式读取文件内容的方法**。

例如，假设我们有一个名为sample.txt的文件，内容为“Hello World”：

```java
class FileToByteArrayUnitTest {

    private static final String FILE_NAME = "src" + File.separator + "test" + File.separator + "resources" + File.separator + "sample.txt";
    private final byte[] expectedByteArray = { 72, 101, 108, 108, 111, 32, 87, 111, 114, 108, 100 };

    @Test
    void givenFile_whenUsingFileInputStreamClass_thenConvert() throws IOException {
        File myFile = new File(FILE_NAME);
        byte[] byteArray = new byte[(int) myFile.length()];
        try (FileInputStream inputStream = new FileInputStream(myFile)) {
            inputStream.read(byteArray);
        }

        assertArrayEquals(expectedByteArray, byteArray);
    }
}
```

这里，我们使用给定的sample.txt文件创建了FileInputStream类的一个实例。此外，我们调用read(byte[] b)方法将FileInputStream实例中的数据读入定义的字节数组中。

**值得注意的是，我们使用了[try-with-resources](https://www.baeldung.com/java-try-with-resources)方法来有效地处理资源的关闭**。

### 2.2 Files#readAllBytes

或者，我们可以使用NIO API中的Files类。**顾名思义，此实用程序类提供了多种可立即使用的静态方法来处理文件和目录**。

那么，让我们看看它的实际效果：

```java
@Test
void givenFile_whenUsingNioApiFilesClass_thenConvert() throws IOException {
    byte[] byteArray = Files.readAllBytes(Paths.get(FILE_NAME));

    assertArrayEquals(expectedByteArray, byteArray);
}
```

我们可以看到，Files类带有readAllBytes()方法，该方法返回指定路径文件中的所有字节。有趣的是，**当读取完字节后，此方法会自动关闭文件**。

另一个重要的警告是，**此方法不适用于读取大文件**。因此，我们只能将其用于简单的情况。

## 3. 使用Apache Commons IO

另一个解决方案是使用[Apache Commons IO库](https://www.baeldung.com/apache-commons-io)，它提供了许多方便的实用程序类，我们可以用它们来执行常见的IO任务。

首先，让我们在pom.xml文件中添加[commons-io依赖](https://mvnrepository.com/artifact/commons-io/commons-io)：

```xml
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.15.1</version>
</dependency>
```

### 3.1 FileUtils#readFileToByteArray

FileUtils类，顾名思义，提供了一组用于文件操作的方法。在这些方法中，我们发现了readFileToByteArray()方法：

```java
@Test
void givenFile_whenUsingApacheCommonsFileUtilsClass_thenConvert() throws IOException {
    byte[] byteArray = FileUtils.readFileToByteArray(new File(FILE_NAME));

    assertArrayEquals(expectedByteArray, byteArray);
}
```

**如上所示，readFileToByteArray()以直接的方式将指定文件的内容读入字节数组，此方法的优点是文件始终处于关闭状态**。

此外，此方法没有Files#readAllBytes的限制，如果提供的文件为空，则会抛出NullPointerException。

### 3.2 IOUtils#toByteArray

Apache Commons IO提供了另一种替代方案，我们可以使用它来实现相同的结果，它提供了IOUtils类来处理常规IO流操作。

因此，让我们用一个实际的例子来举例说明IOUtils的使用：

```java
@Test
void givenFile_whenUsingApacheCommonsIOUtilsClass_thenConvert() throws IOException {
    File myFile = new File(FILE_NAME);
    byte[] byteArray = new byte[(int) myFile.length()];
    try (FileInputStream inputStream = new FileInputStream(myFile)) {
        byteArray = IOUtils.toByteArray(inputStream);
    }

    assertArrayEquals(expectedByteArray, byteArray);
}
```

简而言之，该类带有toByteArray()方法，以byte[]形式返回InputStream的数据。我们不需要在这里使用BufferedInputStream，因为此方法在内部缓冲内容。

## 4. 使用Guava

[Guava库](https://www.baeldung.com/guava-guide)是将文件转换为字节数组时要考虑的另一个选项。像往常一样，在开始使用这个库之前，我们需要将其[依赖](https://mvnrepository.com/artifact/com.google.guava/guava)添加到pom.xml中：

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>32.1.3-jre</version>
</dependency>
```

### 4.1 Files#toByteArray

Guava库提供了自己的Files类版本，那么，让我们在实践中看看它：

```java
@Test
void givenFile_whenUsingGuavaFilesClass_thenConvert() throws IOException {
    byte[] byteArray = com.google.common.io.Files.toByteArray(new File(FILE_NAME));

    assertArrayEquals(expectedByteArray, byteArray);
}
```

**简而言之，我们使用toByteArray()方法来获取包含给定文件的所有字节的字节数组**。

## 5. 总结

在这篇短文中，我们探讨了使用JDK方法、Guava和Apache Commons IO库将文件转换为字节数组的各种方法。