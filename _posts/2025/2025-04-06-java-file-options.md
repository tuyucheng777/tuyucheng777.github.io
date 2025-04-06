---
layout: post
title:  Java文件打开选项
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

在本教程中，我们将重点介绍Java中可用的文件标准打开选项。

我们将探讨实现OpenOption接口并定义这些标准打开选项的StandardOpenOption枚举。

## 2. OpenOption参数

在Java中，我们可以使用NIO2 API处理文件，它包含多种实用方法，其中一些方法使用可选的OpenOption参数来配置如何打开或创建文件。此外，如果未设置此参数，则该参数将具有默认值，每个方法的默认值可能不同。

StandardOpenOption枚举类型定义标准选项并实现OpenOption接口。

**以下是StandardOpenOptions枚举支持的选项列表**：

-   WRITE：打开文件进行写访问
-   APPEND：向文件追加一些数据
-   TRUNCATE_EXISTING：截断文件
-   CREATE_NEW：创建一个新文件，如果文件已经存在则抛出异常
-   CREATE：如果文件存在则打开文件，如果不存在则创建新文件
-   DELETE_ON_CLOSE：关闭流后删除文件
-   SPARSE：新创建的文件是稀疏的
-   SYNC：保留同步文件的内容和元数据
-   DSYNC：只保留同步文件的内容

**在接下来的部分中，我们将看到如何使用每个选项的示例**。

为了避免文件路径上的任何混淆，让我们获取用户主目录的句柄，这在所有操作系统中都有效：

```java
private static String HOME = System.getProperty("user.home");
```

## 3. 打开文件进行读写

首先，**如果我们想创建一个不存在的新文件，我们可以使用选项CREATE**：

```java
@Test
public void givenExistingPath_whenCreateNewFile_thenCorrect() throws IOException {
    assertFalse(Files.exists(Paths.get(HOME, "newfile.txt")));
    Files.write(path, DUMMY_TEXT.getBytes(), StandardOpenOption.CREATE);
    assertTrue(Files.exists(path));
}
```

**我们还可以使用选项CREATE_NEW，如果文件不存在，它将创建一个新文件。但是，如果文件已经存在，它将抛出异常**。

其次，**如果我们想打开文件进行读取，我们可以使用newInputStream(Path, OpenOption...)方法**，此方法打开文件进行读取并返回输入流：

```java
@Test
public void givenExistingPath_whenReadExistingFile_thenCorrect() throws IOException {
    Path path = Paths.get(HOME, DUMMY_FILE_NAME);

    try (InputStream in = Files.newInputStream(path); BufferedReader reader = new BufferedReader(new InputStreamReader(in))) {
        String line;
        while ((line = reader.readLine()) != null) {
            assertThat(line, CoreMatchers.containsString(DUMMY_TEXT));
        }
    }
}
```

请注意，**我们没有使用选项READ，因为方法newInputStream默认使用该选项**。

第三，**我们可以使用newOutputStream(Path, OpenOption...)方法创建文件、向文件追加内容或向文件写入内容**，此方法打开或创建一个用于写入的文件并返回一个OutputStream。

如果我们未指定打开选项，并且文件不存在，则API将创建一个新文件。但是，如果文件存在，它将被截断。此选项类似于使用CREATE和TRUNCATE_EXISTING选项调用方法。

让我们打开一个现有文件并追加一些数据：

```java
@Test
public void givenExistingPath_whenWriteToExistingFile_thenCorrect() throws IOException {
    Path path = Paths.get(HOME, DUMMY_FILE_NAME);

    try (OutputStream out = Files.newOutputStream(path, StandardOpenOption.APPEND, StandardOpenOption.WRITE)) {
        out.write(ANOTHER_DUMMY_TEXT.getBytes());
    }
}
```

## 4. 创建SPARSE文件

我们可以告诉文件系统新创建的文件应该是稀疏的(文件包含不会写入磁盘的空白空间)。

为此，我们应该使用选项SPARSE和CREATE_NEW选项。但是，**如果文件系统不支持稀疏文件，这个选项将被忽略**。

让我们创建一个稀疏文件：

```java
@Test
public void givenExistingPath_whenCreateSparseFile_thenCorrect() throws IOException {
    Path path = Paths.get(HOME, "sparse.txt");
    Files.write(path, DUMMY_TEXT.getBytes(), StandardOpenOption.CREATE_NEW, StandardOpenOption.SPARSE);
}
```

## 5. 保持文件同步

StandardOpenOptions枚举有SYNC和DSYNC选项，这些选项要求将数据同步写入存储中的文件。换句话说，**这些选项将保证在系统崩溃时数据不会丢失**。

让我们将一些数据追加到我们的文件并使用选项SYNC：

```java
@Test
public void givenExistingPath_whenWriteAndSync_thenCorrect() throws IOException {
    Path path = Paths.get(HOME, DUMMY_FILE_NAME);
    Files.write(path, ANOTHER_DUMMY_TEXT.getBytes(), StandardOpenOption.APPEND, StandardOpenOption.WRITE, StandardOpenOption.SYNC);
}
```

**SYNC和DSYNC的区别在于SYNC在存储中同步存储文件的内容和元数据，而DSYNC在存储中只同步存储文件的内容**。

## 6. 关闭流后删除文件

StandardOpenOptions枚举还提供了一个有用的选项，使我们能够在关闭流后销毁文件。**如果我们想创建一个临时文件，这很有用**。

让我们将一些数据追加到我们的文件，并使用选项DELETE_ON_CLOSE：

```java
@Test
public void givenExistingPath_whenDeleteOnClose_thenCorrect() throws IOException {
    Path path = Paths.get(HOME, EXISTING_FILE_NAME);
    assertTrue(Files.exists(path)); // file was already created and exists

    try (OutputStream out = Files.newOutputStream(path, StandardOpenOption.APPEND, StandardOpenOption.WRITE, StandardOpenOption.DELETE_ON_CLOSE)) {
        out.write(ANOTHER_DUMMY_TEXT.getBytes());
    }

    assertFalse(Files.exists(path)); // file is deleted
}
```

## 7. 总结

在本教程中，我们介绍了使用作为Java 7的一部分提供的新文件系统API(NIO2)在Java中打开文件的可用选项。