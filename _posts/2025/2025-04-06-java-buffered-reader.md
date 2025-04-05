---
layout: post
title:  BufferedReader指南
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

[BufferedReader](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/BufferedReader.html)是一个简化从字符输入流中读取文本的类，它缓冲字符以便能够高效地读取文本数据。

在本教程中，我们将了解如何使用BufferedReader类。

## 2. 何时使用BufferedReader

一般来说， 如果我们想从任何类型的输入源(无论是文件、套接字还是其他东西)读取文本，BufferedReader都会派上用场。

简而言之，**它使我们能够通过读取字符块并将它们存储在内部缓冲区中来最大程度地减少I/O操作的数量**。当缓冲区有数据时，读取器将从中读取数据，而不是直接从底层流中读取数据。

### 2.1 缓冲另一个读取器

与大多数Java I/O类一样，**BufferedReader实现了装饰器模式，这意味着它期望在其构造函数中有一个Reader**。通过这种方式，它使我们能够灵活地扩展具有缓冲功能的Reader实现的实例：

```java
BufferedReader reader = new BufferedReader(new FileReader("src/main/resources/input.txt"));
```

但是，如果缓冲对我们来说无关紧要，我们可以直接使用FileReader：

```java
FileReader reader = new FileReader("src/main/resources/input.txt");
```

除了缓冲之外，**BufferedReader还提供了一些很好的辅助函数，用于逐行读取文件**。因此，尽管直接使用FileReader看起来更简单，但BufferedReader可以提供很大的帮助。

### 2.2 缓冲流

通常，**我们可以配置BufferedReader以将任何类型的输入流作为底层源**，我们可以使用InputStreamReader并将其包装在构造函数中：

```java
BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
```

在上面的例子中，我们从System.in读取数据，它通常对应于键盘的输入。类似地，我们可以传递一个输入流以从套接字、文件或任何可以想象的文本输入类型中读取。唯一的先决条件是有一个合适的InputStream实现。

### 2.3 BufferedReader与Scanner

作为替代方案，我们可以使用Scanner类来实现与BufferedReader相同的功能。

但是，这两个类之间存在显著差异，这可能使它们对我们来说或多或少方便，具体取决于我们的用例：

-   BufferedReader是同步的(线程安全的)，而Scanner不是
-   Scanner可以使用正则表达式解析原始类型和字符串
-   BufferedReader允许更改缓冲区的大小，而Scanner具有固定的缓冲区大小
-   BufferedReader具有更大的默认缓冲区大小
-   Scanner隐藏了IOException，而BufferedReader强制我们处理它
-   BufferedReader通常比Scanner快，因为它只读取数据而不解析数据

考虑到这些，**如果我们要解析文件中的单个标记，那么Scanner会比BufferedReader更自然一些。但是，每次只读一行是BufferedReader的优势所在**。

如果需要，我们还提供有关[Scanner的指南](https://www.baeldung.com/java-scanner)。

## 3. 使用BufferedReader读取文本

让我们来看看正确构建、使用和销毁BufferReader以读取文本文件的整个过程。

### 3.1 初始化BufferedReader

**首先，让我们使用BufferedReader(Reader)构造函数创建一个BufferedReader**：

```java
BufferedReader reader = new BufferedReader(new FileReader("src/main/resources/input.txt"));
```

像这样包装FileReader是向其他读取器添加缓冲功能的一个好方法。

默认情况下，这将使用8KB的缓冲区。但是，如果我们想要缓冲更小或更大的块，我们可以使用BufferedReader(Reader, int)构造函数：

```java
BufferedReader reader = new BufferedReader(new FileReader("src/main/resources/input.txt")), 16384);
```

这会将缓冲区大小设置为16384字节(16KB)。

最佳缓冲区大小取决于输入流的类型和运行代码的硬件等因素。因此，要达到理想的缓冲区大小，我们必须自己通过试验找到它。

最好使用2的幂作为缓冲区大小，因为大多数硬件设备的块大小都是2的幂。

最后，**还有一种使用java.nio API中的[Files](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/nio/file/Files.html)辅助类创建BufferedReader的便捷方法**：

```java
BufferedReader reader = Files.newBufferedReader(Paths.get("src/main/resources/input.txt"))
```

如果我们想读取文件，像这样创建它是一种很好的缓冲方法，因为我们不必先手动创建FileReader然后包装它。

### 3.2 逐行读取

接下来，我们使用readLine方法读取文件的内容：

```java
public String readAllLines(BufferedReader reader) throws IOException {
    StringBuilder content = new StringBuilder();
    String line;

    while ((line = reader.readLine()) != null) {
        content.append(line);
        content.append(System.lineSeparator());
    }

    return content.toString();
}
```

**我们可以使用Java 8中引入的lines方法更简单地完成与上述相同的操作**：

```java
public String readAllLinesWithStream(BufferedReader reader) {
    return reader.lines().collect(Collectors.joining(System.lineSeparator()));
}
```

### 3.3 关闭流

使用BufferedReader之后，我们必须调用它的close()方法来释放与其关联的任何系统资源。如果我们使用try-with-resources块，这是自动完成的：

```java
try (BufferedReader reader = new BufferedReader(new FileReader("src/main/resources/input.txt"))) {
    return readAllLines(reader);
}
```

## 4. 其他有用的方法

现在让我们关注BufferedReader中可用的各种有用方法。

### 4.1 读取单个字符

我们可以使用read()方法来读取单个字符，让我们逐个字符地读取整个内容，直到流结束：

```java
public String readAllCharsOneByOne(BufferedReader reader) throws IOException {
    StringBuilder content = new StringBuilder();

    int value;
    while ((value = reader.read()) != -1) {
        content.append((char) value);
    }

    return content.toString();
}
```

这将读取字符(作为ASCII值返回)，将它们转换为char并追加到结果中。我们重复此操作直到流的末尾，这由read()方法的响应值-1指示。

### 4.2 读取多个字符

如果我们想一次读取多个字符，我们可以使用方法read(char[] cbuf, int off, int len)：

```java
public String readMultipleChars(BufferedReader reader) throws IOException {
    int length;
    char[] chars = new char[length];
    int charsRead = reader.read(chars, 0, length);

    String result;
    if (charsRead != -1) {
        result = new String(chars, 0, charsRead);
    } else {
        result = "";
    }

    return result;
}
```

在上面的代码示例中，我们将最多5个字符读入char数组并从中构造一个字符串。如果在读取尝试中未读取任何字符(即我们已经到达流的末尾)，我们将简单地返回一个空字符串。

### 4.3 跳过字符

我们还可以通过调用skip(long n)方法跳过给定数量的字符：

```java
@Test
public void givenBufferedReader_whensSkipChars_thenOk() throws IOException {
    StringBuilder result = new StringBuilder();

    try (BufferedReader reader =
                 new BufferedReader(new StringReader("1__2__3__4__5"))) {
        int value;
        while ((value = reader.read()) != -1) {
            result.append((char) value);
            reader.skip(2L);
        }
    }

    assertEquals("12345", result);
}
```

在上面的示例中，我们读取了一个输入字符串，其中包含由两个下划线分隔的数字。为了构建仅包含数字的字符串，我们通过调用skip方法跳过下划线。

### 4.4 mark和reset

我们可以使用mark(int readAheadLimit)和reset()方法标记流中的某个位置，稍后再返回。作为一个有点牵强的例子，让我们使用mark()和reset()来忽略流开头的所有空格：

```java
@Test
public void givenBufferedReader_whenSkipsWhitespacesAtBeginning_thenOk() throws IOException {
    String result;

    try (BufferedReader reader =
                 new BufferedReader(new StringReader("    Lorem ipsum dolor sit amet."))) {
        do {
            reader.mark(1);
        } while(Character.isWhitespace(reader.read()));

        reader.reset();
        result = reader.readLine();
    }

    assertEquals("Lorem ipsum dolor sit amet.", result);
}
```

在上面的例子中，我们使用mark()方法来标记我们刚刚读取的位置。给它一个值1意味着只有代码会记住向前一个字符的标记，**它在这里很方便，因为一旦我们读到第一个非空白字符，我们就可以返回并重新读取该字符，而无需重新处理整个流。如果没有标记，我们将在最终字符串中丢失L**。

请注意，由于mark()可能抛出UnsupportedOperationException，因此将markSupported()与调用mark()的代码相关联是很常见的。不过，我们实际上并不需要它，**这是因为markSupported()总是为BufferedReader返回true**。

当然，我们也许可以用其他方式更优雅地完成上述操作，而mark和reset确实不是很典型的方法。不过，**当需要向前读时，它们肯定会派上用场**。

## 5. 总结

在本快速教程中，我们学习了如何使用BufferedReader在实际示例中读取字符输入流。