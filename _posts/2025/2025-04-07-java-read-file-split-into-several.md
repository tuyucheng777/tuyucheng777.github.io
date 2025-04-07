---
layout: post
title:  使用Java读取文件并将其拆分为多个文件
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

在本教程中，我们将学习如何使用Java拆分大文件。首先，我们将比较在内存中读取文件和使用流读取文件。稍后，我们将学习根据文件的大小和数量拆分文件。

## 2. 读取内存中的文件与流

每当我们在内存中读取文件时，[JVM](https://www.baeldung.com/jvm-vs-jre-vs-jdk#jvm)都会将所有行保留在内存中。对于小文件来说，这是一个不错的选择。但是，对于大文件，它经常会导致OutOfMemoryException。

通过流式传输文件是读取文件的另一种方法，并且有[许多方法可以流式传输和读取大型文件](https://www.baeldung.com/java-read-lines-large-file)。**由于整个文件不在内存中，因此它占用的内存较少，并且非常适合处理大型文件而不会引发异常**。

为了举个例子，我们将使用流来读取大文件。

## 3. 按文件大小拆分文件

虽然到目前为止我们已经学会了如何读取大文件，但有时我们需要将它们拆分成较小的文件或以较小的大小通过网络发送。

首先，我们将从将大文件拆分成较小的文件开始，每个文件都有特定的大小。

为了举例说明，我们将在项目src/main/resource文件夹中取一个4.3MB的文件largeFile.txt，并将其拆分成每个1MB的文件，并将它们存储在/target/split目录下。**让我们首先获取大文件并在其上打开一个[输入流](https://www.baeldung.com/convert-file-to-input-stream)**：

```java
File largeFile = new File("LARGE_FILE_PATH");
InputStream inputstream = Files.newInputStream(largeFile.toPath());
```

这里，我们只是加载文件元数据，大文件内容尚未加载到内存中。

在我们的示例中，我们有一个固定大小。在实际使用情况下，可以根据应用程序需要动态读取和更改此maxSizeOfSplitFiles值。

现在，**让我们编写一个方法，它接收largeFile对象并为分割文件定义maxSizeOfSplitFiles**：

```java
public List<File> splitByFileSize(File largeFile, int maxSizeOfSplitFiles, String splitFileDirPath) throws IOException {
    // ...
}
```

现在，让我们创建一个SplitLargeFile类和splitByFileSize()方法：

```java
class SplitLargeFile {

    public List<File> splitByFileSize(File largeFile, int maxSizeOfSplitFiles, String splitFileDirPath) throws IOException {
        List<File> listOfSplitFiles = new ArrayList<>();
        try (InputStream in = Files.newInputStream(largeFile.toPath())) {
            final byte[] buffer = new byte[maxSizeOfSplitFiles];
            int dataRead = in.read(buffer);
            while (dataRead > -1) {
                File splitFile = getSplitFile(FilenameUtils.removeExtension(largeFile.getName()),
                        buffer, dataRead, splitFileDirPath);
                listOfSplitFiles.add(splitFile);
                dataRead = in.read(buffer);
            }
        }
        return listOfSplitFiles;
    }

    private File getSplitFile(String largeFileName, byte[] buffer, int length, String splitFileDirPath) throws IOException {
        File splitFile = File.createTempFile(largeFileName + "-", "-split", new File(splitFileDirPath));
        try (FileOutputStream fos = new FileOutputStream(splitFile)) {
            fos.write(buffer, 0, length);
        }
        return splitFile;
    }
}
```

使用maxSizeOfSplitFiles，我们可以指定每个较小的分块文件可以有多少字节。**maxSizeOfSplitFiles量的数据将被加载到内存中，进行处理，并制成一个小文件**。然后我们将其删除。我们读取下一组maxSizeOfSplitFiles数据，这确保不会抛出OutOfMemoryException。

最后一步，该方法返回存储在splitFileDirPath下的分割文件列表，我们可以[将分割文件存储在任何临时目录或任何自定义目录中](https://www.baeldung.com/java-temp-directories)。

现在，让我们测试一下：

```java
public class SplitLargeFileUnitTest {

    @BeforeClass
    static void prepareData() throws IOException {
        Files.createDirectories(Paths.get("target/split"));
    }

    private String splitFileDirPath() throws Exception {
        return Paths.get("target").toString() + "/split";
    }

    private Path largeFilePath() throws Exception {
        return Paths.get(this.getClass().getClassLoader().getResource("largeFile.txt").toURI());
    }

    @Test
    void givenLargeFile_whenSplitLargeFile_thenSplitBySize() throws Exception {
        File input = largeFilePath().toFile();
        SplitLargeFile slf = new SplitLargeFile();
        slf.splitByFileSize(input, 1024_000, splitFileDirPath());
    }
}
```

最后，我们进行测试，可以看到程序将大文件分割成4个1MB的文件和一个240KB的文件，并将它们放在项目的target/split目录下。

## 4. 按文件数量拆分文件

现在，让我们将给定的大文件拆分为指定数量的小文件。为此，首先，我们将**根据计算出的文件数量检查小文件的大小是否合适**。

此外，我们将在内部使用与之前相同的方法splitByFileSize()进行实际的拆分。

让我们创建一个方法splitByNumberOfFiles()：

```java
class SplitLargeFile {

    public List<File> splitByNumberOfFiles(File largeFile, int noOfFiles, String splitFileDirPath)
            throws IOException {
        return splitByFileSize(largeFile, getSizeInBytes(largeFile.length(), noOfFiles), splitFileDirPath);
    }

    private int getSizeInBytes(long largefileSizeInBytes, int numberOfFilesforSplit) {
        if (largefileSizeInBytes % numberOfFilesforSplit != 0) {
            largefileSizeInBytes = ((largefileSizeInBytes / numberOfFilesforSplit) + 1) * numberOfFilesforSplit;
        }
        long x = largefileSizeInBytes / numberOfFilesforSplit;
        if (x > Integer.MAX_VALUE) {
            throw new NumberFormatException("size too large");
        }
        return (int) x;
    }
}
```

现在，让我们测试一下：

```java
@Test
void givenLargeFile_whenSplitLargeFile_thenSplitByNumberOfFiles() throws Exception { 
    File input = largeFilePath().toFile(); 
    SplitLargeFile slf = new SplitLargeFile(); 
    slf.splitByNumberOfFiles(input, 3, splitFileDirPath()); 
}
```

最后，我们测试一下，可以看到程序将大文件分割成3个1.4MB的文件，并放在项目target/split目录下。

## 5. 总结

在本文中，我们了解了在内存中读取文件和通过流读取文件之间的区别，这有助于我们根据用例选择合适的方法。之后，我们讨论了如何将大文件拆分为小文件。然后，我们了解了按大小拆分和按文件数量拆分。