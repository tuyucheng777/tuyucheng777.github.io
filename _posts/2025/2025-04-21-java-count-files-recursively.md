---
layout: post
title:  使用Java获取目录及其子目录中的文件数量
category: java-nio
copyright: java-nio
excerpt: Java NIO
---

## 1. 概述

**计算目录及其子目录中的文件数量是编程中的一项常见任务**，无论你是构建备份实用程序、监控磁盘使用情况，还是跨系统同步文件。对于Java开发者来说，这个看似简单的问题提供了一个探索传统和现代文件处理方法的机会。

在本教程中，我们将深入探讨两种主要方法：许多人熟悉的递归java.io.File方法，以及Java 7中引入的更高效的[基于NIO](https://www.baeldung.com/java-io-vs-nio)的解决方案。

## 2. 设置

**统计目录及其子目录中文件数量的任务需要遍历一个可能很复杂的树形结构，其中每个子目录可能包含更多文件或其他子目录**。这种递归特性带来了技术挑战：如何确保所有层级的准确性，避免符号链接导致的无限循环，以及如何管理大型数据集的性能？

对于Java开发人员来说，解决方案在于选择正确的工具并设计清晰灵活的方法。

让我们为实现定义方法签名：

```java
public long numberOfFilesIn(String path) {
    // TODO: Implementation 
}
```

该方法接收一个String path输入参数作为起始目录，并以long类型返回找到的文件数。long类型确保我们能够处理超出32位int限制的计数，这对于企业级目录至关重要。

## 3. 使用java.io.File

我们的第一个实现是[用Java读取文件](https://www.baeldung.com/reading-file-in-java)并对其进行计数，将使用[java.io.File](https://www.baeldung.com/java-io-file)库。

### 3.1 使用Java 8+

让我们从使用[Java 8](https://www.baeldung.com/java-8-new-features)及更高版本的实现开始，这样我们就可以使用[Stream](https://www.baeldung.com/java-8-streams)以函数式的方式编写。

我们的任务是遍历目录、搜索文件，然后继续进入更深的嵌套目录，这个过程是重复的。有一种实现方式在这里很有意义：[递归](https://www.baeldung.com/java-recursion)，与迭代方法相比，递归可以自我调用，这有助于我们降低复杂性：

```java
File currentFile = new File(path);
File[] filesOrNull = currentFile.listFiles();
// Is this a file already?
long currentFileNumber = currentFile.isFile() ? 1 : 0;
if (filesOrNull == null) { // no sub directories found
    return currentFileNumber; // stop condition #1
}
return currentFileNumber + Arrays.stream(filesOrNull)
    .mapToLong(FindFolder::filesInside) // <-- recursion call here
    .sum();
```

我们的实现基本上分为三个部分：

1. 检查当前路径是否解析为文件或目录
2. 如果当前目录无法列出任何文件，我们将立即返回
3. 否则，我们递归计算子文件夹中的文件数并返回总数，包括当前文件数

```java
private static long filesInside(File it) {
    if (it.isFile()) {
        return 1; // stop condition #2
    } else if (it.isDirectory()) {
        return numberOfFilesIn(it.getAbsolutePath()); // <-- recursion to caller
    } else {
        return 0; // stop condition #3
    }
}
```

请注意，我们在递归实现中包含了三个停止条件，**如果未能及时终止递归循环，可能会导致内存不足异常**。 

### 3.2 Java 8之前

如果我们使用的是低于版本8的Java，我们可以简单地从上面的递归流实现重构我们的方法。

这是我们的第二种方法，重写后没有Stream的部分如下所示：

```java
for (File file : filesOrNull) {
    if (file.isDirectory()) {
        currentFileNumber += numberOfFilesIn(file.getAbsolutePath());
    } else if (file.isFile()) {
        currentFileNumber += 1; // add this file to count
    }
}
return currentFileNumber;
```

它看起来与我们之前使用的映射函数非常相似，只是这次我们通过添加中间结果来修改currentFileNumber变量。如果我们忽略可变性问题，这个解决方案看起来简洁、代码量少、可读性好。

## 4. 使用NIO

在Java中使用文件系统有另一个很好的替代库，自Java 7开始提供：[NIO](https://www.baeldung.com/java-nio-2-file-api)。

### 4.1 Files.find

我们完成该任务的第三种方法-计算目录和子目录中的文件数量，将利用[Files.find](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#find-java.nio.file.Path-int-java.util.function.BiPredicate-java.nio.file.FileVisitOption...-)功能：

```java
try (Stream<Path> stream = Files.find(
  Paths.get(path), 
  Integer.MAX_VALUE,
  (__, attr) -> attr.isRegularFile())) {
    return stream.count();
} catch (IOException e) {
    // or log here
    throw new RuntimeException(e);
}
```

我们使用[try with resources](https://www.baeldung.com/java-try-with-resources)在我们的初始路径目录中打开[Path](https://www.baeldung.com/java-path-vs-file)流。

### 4.2 遍历

NIO的第二种可能性是以迭代的方式“[遍历](https://www.baeldung.com/java-list-directory-files#walking)”文件系统。

[Files.walk](https://docs.oracle.com/javase/8/docs/api/java/nio/file/Files.html#walk-java.nio.file.Path-java.nio.file.FileVisitOption...-)的实现如下：

```java
Path dir = Path.of(path);
try (Stream<Path> stream = Files.walk(dir)) {
    return stream.parallel()
        .map(getFileOrEmpty())
        .flatMap(Optional::stream)
        .filter(it -> !it.isDirectory())
        .count();
} catch (IOException e) {
    throw new RuntimeException(e);
}
```

和之前一样，我们在初始路径目录中获取一个Path流。**由于搜索目录的顺序无关紧要，而且需要覆盖的路径数量可能很大，因此我们将其转换为并行的流**。然后，我们过滤掉无法与默认提供程序关联的Path，并将其包装到[Java Optional](https://www.baeldung.com/java-optional)中。最后，返回所有非目录(即文件)元素的数量。

```java
private static Function<Path, Optional<File>> getFileOrEmpty() {
    return it -> {
        try {
            return Optional.of(it.toFile());
        } catch (UnsupportedOperationException e) {
            // You may print or log the exception here;
            return Optional.empty();
        }
    };
}
```

提取的方法getFileOrEmpty返回一个映射函数，用于将有效的File安全地包装到Optional中。它有两个好处：保持调用者方法的简洁；以及处理UnsupportedOperationException异常，该异常不应该在Stream中[置之不理](https://www.baeldung.com/java-exceptions)。

## 5. 总体考虑

在实现文件计数解决方案时，有几个因素需要注意。性能至关重要-由于堆栈溢出风险，递归java.io.File方法在处理深层目录时可能会遇到困难，而NIO基于流的方法在处理大型数据集时则具有更好的扩展性。对于复杂的结构，可以考虑使用Files.walk的并行流，但这需要谨慎处理并发文件修改。

**安全性也很重要：如果用户输入驱动路径，请验证它以防止目录遍历攻击**。

**此外，利用NIO的Unicode支持确保与国际文件名的兼容性，避免非ASCII字符的问题**。

平衡效率、安全性和稳健性可确保这些方法有效满足现实世界的需求。

## 6. 验证

现在我们已经了解了所有不同的实现，让我们设计一个测试来运行。

由于验证必须通过真实目录，因此让我们创建一些文件和文件夹来搜索：

```text
filesToBeFound
|-- file1.txt
|-- subEmptyFolder
|-- subFolder1
    |-- file2.txt
    |-- file3.txt
|-- subFolder2 
    |-- file4.txt
    |-- subSubFolder
        |-- subSubSubFolder
            |-- file5.txt
```

唯一剩下的部分就是测试本身：

```java
private final String resourcePath = this.getClass().getResource("/filesToBeFound").getPath();

@Test
void shouldReturnNumberOfAllFilesInsidePath() {
    assertThat(FindFolder.numberOfFilesIn(resourcePath)).isEqualTo(5);
}
```

我们的每个实现都应该在安装目录中找到相同的5个文件。

## 7. 总结

在本教程中，我们探索了两种使用Java统计目录及其子目录中文件数量的强大方法。java.io.File方法具有递归的简单性，适合较小、较浅的目录结构，并为开发人员提供了清晰的入口点。然而，该方法对递归的依赖在深层层次结构中可能会失效，从而存在堆栈溢出错误的风险。相比之下，基于NIO的解决方案(使用Files.find和Files.walk)提供了效率和可扩展性，利用流来处理大型数据集。并行Files.walk选项进一步优化了大型目录的性能，尽管它需要谨慎的异常处理。

对于大多数现代应用程序来说，NIO方法因其稳健性和性能而脱颖而出，成为更佳选择。对于快速、简单的任务，请选择java.io.File；但如果对可扩展性要求较高，则更推荐使用NIO。