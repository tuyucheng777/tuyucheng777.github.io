---
layout: post
title:  Java FileWriter中flush()和close()的区别
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

文件处理是我们经常遇到的一个重要方面，在将数据写入文件时，通常使用[FileWriter](https://www.baeldung.com/java-filewriter)类。在这个类中，两个重要的方法flush()和close()在管理文件输出流方面发挥着不同的作用。

在本教程中，我们将介绍FileWriter的常见用法，并深入研究其flush()和close()方法之间的区别。

## 2. FileWriter和try-with-resources

Java中的[try-with-resources](https://www.baeldung.com/java-try-with-resources)语句是一种强大的资源管理机制，尤其是在处理文件处理等I/O操作时。**一旦退出代码块(无论是正常退出还是由于异常退出)，它都会自动关闭其声明中指定的资源**。

但是，在某些情况下使用FileWriter和try-with-resources并不理想或没有必要。

FileWriter在有或没有try-with-resources的情况下可能会表现不同，接下来，我们来仔细看看。

### 2.1 使用FileWriter和try-with-resources

如果我们将FileWriter与try-with-resources一起使用，则**当我们退出try-with-resources块时，FileWriter对象会自动刷新并关闭**。

接下来，让我们在单元测试中展示这一点：

```java
@Test
void whenUsingFileWriterWithTryWithResources_thenAutoClosed(@TempDir Path tmpDir) throws IOException {
    Path filePath = tmpDir.resolve("auto-close.txt");
    File file = filePath.toFile();

    try (FileWriter fw = new FileWriter(file)) {
        fw.write("Catch Me If You Can");
    }

    List<String> lines = Files.readAllLines(filePath);
    assertEquals(List.of("Catch Me If You Can"), lines);
}
```

由于我们的测试将写入和读取文件，因此我们使用了[JUnit 5的临时目录扩展](https://www.baeldung.com/junit-5-temporary-directory)(@TempDir)，使用此扩展，我们可以专注于测试核心逻辑，而无需为测试目的手动创建和管理临时目录和文件。

如测试方法所示，我们在try-with-resources块中写入一个字符串。然后，当我们使用[Files.readAllLines()](https://www.baeldung.com/reading-file-in-java#1-reading-a-small-file)检查文件内容时，我们得到了预期的结果。

### 2.2 使用不带try-with-resources的FileWriter

但是，当我们使用没有try-with-resources的FileWriter时，**FileWriter对象不会自动刷新和关闭**：

```java
@Test
void whenUsingFileWriterWithoutFlush_thenDataWontBeWritten(@TempDir Path tmpDir) throws IOException {
    Path filePath = tmpDir.resolve("noFlush.txt");
    File file = filePath.toFile();
    FileWriter fw = new FileWriter(file);
    fw.write("Catch Me If You Can");

    List<String> lines = Files.readAllLines(filePath);
    assertEquals(0, lines.size());
    fw.close(); //close the resource
}
```

如上面的测试所示，**虽然我们通过调用FileWriter.write()向文件写入了一些文本，但是文件仍然是空的**。

接下来我们来看看如何解决这个问题。

## 3. FileWriter.flush()和FileWriter.close()

本节我们先解决“文件仍为空”的问题，然后再讨论FileWriter.flush()和FileWriter.close()的区别。

### 3.1 解决“文件仍为空”问题

首先我们来快速了解一下为什么调用FileWriter.write()之后文件仍然为空。

当我们调用FileWriter.write()时，数据不会立即写入磁盘上的文件。相反，它会暂时存储在缓冲区中。因此，**要可视化文件中的数据，必须将缓冲数据刷新到文件中**。

直接的方法是调用flush()方法：

```java
@Test
void whenUsingFileWriterWithFlush_thenGetExpectedResult(@TempDir Path tmpDir) throws IOException {
    Path filePath = tmpDir.resolve("flush1.txt");
    File file = filePath.toFile();
    
    FileWriter fw = new FileWriter(file);
    fw.write("Catch Me If You Can");
    fw.flush();

    List<String> lines = Files.readAllLines(filePath);
    assertEquals(List.of("Catch Me If You Can"), lines);
    fw.close(); //close the resource
}
```

可以看到，调用flush()之后，我们可以通过读取文件获取预期的数据。

或者，我们可以调用close()方法将缓冲数据传输到文件。这是因为**close()首先执行刷新，然后关闭文件流写入器**。

接下来，让我们创建一个测试来验证这一点：

```java
@Test
void whenUsingFileWriterWithClose_thenGetExpectedResult(@TempDir Path tmpDir) throws IOException {
    Path filePath = tmpDir.resolve("close1.txt");
    File file = filePath.toFile();
    FileWriter fw = new FileWriter(file);
    fw.write("Catch Me If You Can");
    fw.close();

    List<String> lines = Files.readAllLines(filePath);
    assertEquals(List.of("Catch Me If You Can"), lines);
}
```

因此，它看起来与flush()调用非常相似。然而，这两种方法在处理文件输出流时有不同的用途。

接下来我们来仔细看看它们的区别。

### 3.2 flush()和close()方法之间的区别

**flush()方法主要用于强制立即写入任何缓冲数据而不关闭FileWriter，而close()方法既执行刷新又释放相关资源**。

换句话说，调用flush()可确保缓冲数据迅速写入磁盘，从而**允许继续对文件执行写入或追加操作**而无需关闭流。相反，调用close()时，它会将现有的缓冲数据写入文件，然后关闭文件。因此，**除非打开新流(例如通过初始化新的FileWriter对象)，否则无法将更多数据写入文件**。

接下来，我们通过一些例子来理解这一点：

```java
@Test
void whenUsingFileWriterWithFlushMultiTimes_thenGetExpectedResult(@TempDir Path tmpDir) throws IOException {
    List<String> lines = List.of("Catch Me If You Can", "A Man Called Otto", "Saving Private Ryan");
    Path filePath = tmpDir.resolve("flush2.txt");
    File file = filePath.toFile();
    FileWriter fw = new FileWriter(file);
    for (String line : lines) {
        fw.write(line + System.lineSeparator());
        fw.flush();
    }

    List<String> linesInFile = Files.readAllLines(filePath);
    assertEquals(lines, linesInFile);
    fw.close(); //close the resource
}
```

在上面的例子中，我们调用了3次write()来将3行写入文件。每次调用write()之后，我们都会调用flush()。最后，我们可以从目标文件中读取这3行。

但是，**如果我们在调用FileWriter.close()之后尝试写入数据，则会引发IOException，并显示错误消息“Stream closed”**：

```java
@Test
void whenUsingFileWriterWithCloseMultiTimes_thenGetIOExpectedException(@TempDir Path tmpDir) throws IOException {
    List<String> lines = List.of("Catch Me If You Can", "A Man Called Otto", "Saving Private Ryan");
    Path filePath = tmpDir.resolve("close2.txt");
    File file = filePath.toFile();
    FileWriter fw = new FileWriter(file);
    //write and close
    fw.write(lines.get(0) + System.lineSeparator());
    fw.close();

    //writing again throws IOException
    Throwable throwable = assertThrows(IOException.class, () -> fw.write(lines.get(1)));
    assertEquals("Stream closed", throwable.getMessage());
}
```

## 4. 总结

在本文中，我们探讨了FileWriter的常见用法。此外，我们还讨论了FileWriter的flush()和close()方法之间的区别。