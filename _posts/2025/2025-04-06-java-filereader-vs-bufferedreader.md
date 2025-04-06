---
layout: post
title:  Java中FileReader和BufferedReader的区别
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

FileReader和BufferedReader是两个可以从输入流读取字符的类。

在本教程中，我们将了解它们之间的区别。

## 2. FileReader

**FileReader类可以从文件中读取字符流**，此外，它只能逐个字符地读取文件，每次我们调用它的read()方法时，它都会直接访问硬盘上的文件以从中读取一个字符。因此，[FileReader](https://www.baeldung.com/java-filereader)本身在从文件中读取字符时非常慢且效率低下。此外，FileReader只能从文件中读取字符，而不能读取其他类型的输入流。

### 2.1 构造函数

FileReader有三个构造函数：

- FileReader(File file)：接收File实例作为参数
- FileReader(FileDescriptor fd)：接收FileDescriptor作为参数
- FileReader(String fileName)：接收文件名(包括其路径)作为参数

### 2.2 返回值

每次我们调用read()方法时，它都会返回一个整数值，表示从文件中读取的字符的Unicode值，如果到达字符流的末尾，则返回-1。

### 2.3 示例

我们来看一个使用FileReader从包含“qwerty”作为内容的文本文件中读取字符的例子：

```java
@Test
public void whenReadingAFile_thenReadsCharByChar() {
    StringBuilder result = new StringBuilder();

    try (FileReader fr = new FileReader("src/test/resources/sampleText2.txt")) {
        int i = fr.read();

        while(i != -1) {
            result.append((char)i);

            i = fr.read();
        }
    } catch (IOException e) {
        e.printStackTrace();
    }

    assertEquals("qwerty", result.toString());
}
```

在上面的代码中，我们将read()方法的返回值转换为char，然后将其追加到结果字符串。

## 3. BufferedReader

**[BufferedReader](https://www.baeldung.com/java-buffered-reader)类创建一个缓冲区来保存来自字符输入流的数据，此外，输入流可以是文件、控制台、字符串或任何其他类型的字符流**。

它的构造函数接收一个Reader作为字符输入流，因此，我们可以将任何实现Reader抽象类的类赋予BufferedReader作为从中读取字符的输入流。

当我们开始从BufferedReader读取时，它会从输入流读取整个数据块并将其存储在缓冲区中。之后，如果我们继续从BufferedReader读取，它会从缓冲区而不是底层字符流返回字符，直到缓冲区为空。然后，它会从输入流读取另一个数据块并将其存储在缓冲区中以供进一步读取调用。

BufferedReader类减少了对输入流调用的读取操作，并且从缓冲区读取通常比访问底层输入流要快得多。因此，BufferedReader提供了一种更快、更高效的从字符流读取字符的方法。

### 3.1 构造函数

BufferedReader有两个构造函数：

- BufferedReader(Reader in)：接收字符输入流(必须实现Reader抽象类)作为参数
- BufferedReader(Reader in, int sz)：接收字符输入流和缓冲区大小作为参数

### 3.2 返回值

如果我们调用read()方法，它将返回一个int值，即从输入流读取的字符的Unicode值。此外，如果我们调用readLine()方法，它将从缓冲区读取整行并将其作为字符串值返回。

### 3.3 示例

让我们使用BufferedReader从包含3行内容的文本文件中读取字符，使用更高效的InputStreamReader实现：

```java
@Test
public void whenReadingAFile_thenReadsLineByLine() {
    StringBuilder result = new StringBuilder();

    final Path filePath = new File("src/test/resources/sampleText1.txt").toPath();
    try (BufferedReader br = new BufferedReader(new InputStreamReader(Files.newInputStream(filePath), StandardCharsets.UTF_8))) {
        String line;

        while((line = br.readLine()) != null) {
            result.append(line);
            result.append('\n');
        }
    } catch (IOException e) {
        e.printStackTrace();
    }

    assertEquals("first line\nsecond line\nthird line\n", result.toString());
}
```

上述测试代码通过，这意味着BufferedReader成功从文件中读取全部3行文本。

## 4. 有什么区别？

**BufferedReader比FileReader更快、更高效**，因为它从输入流读取整个数据块并将其保存在缓冲区中以供进一步读取调用，而FileReader需要访问每个字符的文件。此外，FileReader只能逐个字符地读取文件，而BufferedReader还有其他方法，如readLine()，它可以从缓冲区读取整行。最后，FileReader只能从文件中读取，而BufferedReader可以从任何类型的字符输入流(文件、控制台、字符串等)读取：

|    FileReader    | BufferedReader |
| :----------------: |:--------------:|
| 速度较慢且效率较低 |    更快速、更高效     |
| 只能逐个字符地读取 |    能够读取字符和行    |
|   只能从文件读取   |  可以读取任何类型的字符流  |

 

如果我们读取的是小文件，并且对文件数据的读取调用很少，那么FileReader就足够了。但是，对于大文件或对数据的读取操作很多时，BufferedReader是更好的选择。

## 5. 总结

在本教程中，我们学习了如何使用FileReader和BufferedReader以及它们之间的区别。