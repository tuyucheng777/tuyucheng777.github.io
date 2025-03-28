---
layout: post
title:  如何使用Java创建受密码保护的Zip文件并解压
category: libraries
copyright: libraries
excerpt: Zip4j
---

## 1. 概述

在之前的教程中，我们展示了如何在java.util.zip包的帮助下[使用Java进行压缩和解压缩](https://www.baeldung.com/java-compress-and-uncompress)。但我们没有任何标准Java库来创建受密码保护的zip文件。

在本教程中，我们将学习如何创建受密码保护的zip文件并使用[Zip4j](https://github.com/srikanth-lingala/zip4j)库对其进行解压缩，它是最全面的zip文件Java库。

## 2. 依赖

让我们首先将[zip4j](https://mvnrepository.com/artifact/net.lingala.zip4j/zip4j)依赖添加到pom.xml文件中：

```xml
<dependency>
    <groupId>net.lingala.zip4j</groupId>
    <artifactId>zip4j</artifactId>
    <version>2.11.5</version>
</dependency>
```

## 3. 压缩文件

首先，**我们将使用ZipFile addFile()方法将名为aFile.txt的文件压缩到名为compressed.zip的受密码保护的档案中**：

```java
ZipParameters zipParameters = new ZipParameters();
zipParameters.setEncryptFiles(true);
zipParameters.setCompressionLevel(CompressionLevel.HIGHER);
zipParameters.setEncryptionMethod(EncryptionMethod.AES);

ZipFile zipFile = new ZipFile("compressed.zip", "password".toCharArray());
zipFile.addFile(new File("aFile.txt"), zipParameters);
```

setCompressionLevel行是可选的，我们可以从FASTEST到ULTRA级别进行选择(默认为NORMAL)。

在这个例子中，我们使用了AES加密。如果我们想使用Zip标准加密，我们只需将AES替换为ZIP_STANDARD。

请注意，**如果文件“aFile.txt”在磁盘上不存在，该方法将抛出异常：“net.lingala.zip4j.exception.ZipException: File does not exist: ...”**。

为了解决这个问题，我们必须确保该文件是手动创建并放在项目文件夹中，或者我们必须从Java创建它：

```java
File fileToAdd = new File("aFile.txt");
if (!fileToAdd.exists()) {
    fileToAdd.createNewFile();
}
```

另外，在我们完成新的ZipFile之后，**关闭资源是一个好习惯**：

```java
zipFile.close();

```

## 4. 压缩多个文件

让我们稍微修改一下代码，以便我们可以一次压缩多个文件：

```java
ZipParameters zipParameters = new ZipParameters();
zipParameters.setEncryptFiles(true);
zipParameters.setEncryptionMethod(EncryptionMethod.AES);

List<File> filesToAdd = Arrays.asList(
    new File("aFile.txt"),
    new File("bFile.txt")
);

ZipFile zipFile = new ZipFile("compressed.zip", "password".toCharArray());
zipFile.addFiles(filesToAdd, zipParameters);
```

我们不使用addFile方法，而是**使用addFiles()并传入文件列表**。

## 5. 压缩目录

我们可以通过将addFile方法替换为addFolder来压缩文件夹：

```java
ZipFile zipFile = new ZipFile("compressed.zip", "password".toCharArray());
zipFile.addFolder(new File("/users/folder_to_add"), zipParameters);
```

## 6. 创建拆分Zip文件

当大小超过特定限制时，我们**可以使用createSplitZipFile和createSplitZipFileFromFolder方法将zip文件拆分为多个文件**：

```java
ZipFile zipFile = new ZipFile("compressed.zip", "password".toCharArray());
int splitLength = 1024 * 1024 * 10; //10MB
zipFile.createSplitZipFile(Arrays.asList(new File("aFile.txt")), zipParameters, true, splitLength);
zipFile.createSplitZipFileFromFolder(new File("/users/folder_to_add"), zipParameters, true, splitLength);
```

splitLength的单位是字节。

## 7. 提取所有文件

提取文件同样简单，我们可以使用extractAll()方法从compressed.zip中提取所有文件：

```java
ZipFile zipFile = new ZipFile("compressed.zip", "password".toCharArray());
zipFile.extractAll("/destination_directory");
```

## 8. 提取单个文件

如果我们只想从compressed.zip中提取一个文件，我们可以使用extractFile()方法：

```java
ZipFile zipFile = new ZipFile("compressed.zip", "password".toCharArray());
zipFile.extractFile("aFile.txt", "/destination_directory");
```

## 9. 总结

总之，我们学习了如何使用Zip4j库在Java中创建受密码保护的zip文件并解压缩它们。