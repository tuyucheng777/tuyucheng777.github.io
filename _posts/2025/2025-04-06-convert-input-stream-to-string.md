---
layout: post
title:  InputStream转为String
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

在本教程中，我们将了解如何将InputStream转换为String。

我们将首先使用纯Java(包括Java 8/9解决方案)，然后研究使用[Guava](https://github.com/google/guava)和[Apache Commons IO](https://commons.apache.org/proper/commons-io/)库。

## 2. 使用Java转换–StringBuilder

让我们看一下使用普通Java、InputStream和StringBuilder的简单、较低级别的方法：

```java
@Test
public void givenUsingJava5_whenConvertingAnInputStreamToAString_thenCorrect() throws IOException {
    String originalString = randomAlphabetic(DEFAULT_SIZE);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());

    StringBuilder textBuilder = new StringBuilder();
    try (Reader reader = new BufferedReader(new InputStreamReader
            (inputStream, Charset.forName(StandardCharsets.UTF_8.name())))) {
        int c = 0;
        while ((c = reader.read()) != -1) {
            textBuilder.append((char) c);
        }
    }
    assertEquals(textBuilder.toString(), originalString);
}
```

## 3. 使用Java 8转换–BufferedReader

Java 8为BufferedReader带来了一个新的[lines()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/BufferedReader.html#lines())方法，让我们看看如何使用它来将InputStream转换为String：

```java
@Test
public void givenUsingJava8_whenConvertingAnInputStreamToAString_thenCorrect() {
    String originalString = randomAlphabetic(DEFAULT_SIZE);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());

    String text = new BufferedReader(
      new InputStreamReader(inputStream, StandardCharsets.UTF_8))
        .lines()
        .collect(Collectors.joining("n"));

    assertThat(text, equalTo(originalString));
}
```

值得一提的是，lines()在底层使用了readLine()方法。readLine()假定一行由换行符(“n”)、回车符(“r”)或回车符后紧跟换行符中的任何一个终止。换句话说，它支持所有常见的行尾样式：Unix、Windows，甚至是旧的Mac OS。

另一方面，**当我们使用Collectors.joining()时，我们需要明确决定要为创建的String使用哪种类型的EOL**。

我们还可以使用Collectors.joining(System.lineSeparator())，在这种情况下输出取决于系统设置。

## 4. 使用Java 9转换–InputStream.readAllBytes()

如果我们使用的是Java 9或更高版本，我们可以使用添加到InputStream的新readAllBytes方法：

```java
@Test
public void givenUsingJava9_whenConvertingAnInputStreamToAString_thenCorrect() throws IOException {
    String originalString = randomAlphabetic(DEFAULT_SIZE);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());

    String text = new String(inputStream.readAllBytes(), StandardCharsets.UTF_8);
    
    assertThat(text, equalTo(originalString));
}
```

我们需要知道，这段简单的代码适用于将所有字节读入字节数组的简单情况，我们不应该用它来读取具有大量数据的输入流。

## 5. 使用Java和Scanner进行转换

接下来，让我们看一个使用标准文本Scanner的普通Java示例：

```java
@Test
public void givenUsingJava7_whenConvertingAnInputStreamToAString_thenCorrect() throws IOException {
    String originalString = randomAlphabetic(8);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());

    String text = null;
    try (Scanner scanner = new Scanner(inputStream, StandardCharsets.UTF_8.name())) {
        text = scanner.useDelimiter("A").next();
    }

    assertThat(text, equalTo(originalString));
}
```

请注意，InputStream将随着Scanner的关闭而关闭。

还值得澄清一下useDelimiter(“\\\\A”) 的作用，这里我们传递了'\\A'，这是一个边界标记正则表达式，表示输入的开始。本质上，这意味着next()调用读取整个输入流。

## 6. 使用ByteArrayOutputStream进行转换

最后，让我们看另一个简单的Java示例，这次使用ByteArrayOutputStream类：

```java
@Test
public void givenUsingPlainJava_whenConvertingAnInputStreamToString_thenCorrect() throws IOException {
    String originalString = randomAlphabetic(8);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());

    ByteArrayOutputStream buffer = new ByteArrayOutputStream();
    int nRead;
    byte[] data = new byte[1024];
    while ((nRead = inputStream.read(data, 0, data.length)) != -1) {
        buffer.write(data, 0, nRead);
    }

    buffer.flush();
    byte[] byteArray = buffer.toByteArray();

    String text = new String(byteArray, StandardCharsets.UTF_8);
    assertThat(text, equalTo(originalString));
}
```

在此示例中，**通过读取和写入字节块将InputStream转换为ByteArrayOutputStream，然后将OutputStream转换为字节数组，用于创建String**。

## 7. 使用java.nio转换

另一种解决方案是**将InputStream的内容复制到文件中，然后将其转换为String**：

```java
@Test
public void givenUsingTempFile_whenConvertingAnInputStreamToAString_thenCorrect() throws IOException {
    String originalString = randomAlphabetic(DEFAULT_SIZE);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());

    Path tempFile = Files.createTempDirectory("").resolve(UUID.randomUUID().toString() + ".tmp");
    Files.copy(inputStream, tempFile, StandardCopyOption.REPLACE_EXISTING);
    String result = new String(Files.readAllBytes(tempFile));

    assertThat(result, equalTo(originalString));
}
```

这里我们使用java.nio.file.Files类来创建一个临时文件，并将InputStream的内容复制到该文件中，然后使用同一个类通过readAllBytes()方法将文件内容转换为String。

## 8. 使用Guava转换

让我们从一个利用ByteSource功能的Guava示例开始：

```java
@Test
public void givenUsingGuava_whenConvertingAnInputStreamToAString_thenCorrect() throws IOException {
    String originalString = randomAlphabetic(8);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());

    ByteSource byteSource = new ByteSource() {
        @Override
        public InputStream openStream() throws IOException {
            return inputStream;
        }
    };

    String text = byteSource.asCharSource(Charsets.UTF_8).read();

    assertThat(text, equalTo(originalString));
}
```

步骤分解：

-   首先，我们将InputStream包装到ByteSource中，据我们所知，这是最简单的方法。
-   然后，我们将ByteSource视为具有UTF8字符集的CharSource。
-   最后，我们使用CharSource将其作为字符串读取。

**进行转换的一种更简单的方法是使用Guava**，但需要显式关闭流；幸运的是，我们可以简单地使用try-with-resources语法来解决这个问题：

```java
@Test
public void givenUsingGuavaAndJava7_whenConvertingAnInputStreamToAString_thenCorrect() throws IOException {
    String originalString = randomAlphabetic(8);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());
 
    String text = null;
    try (Reader reader = new InputStreamReader(inputStream)) {
        text = CharStreams.toString(reader);
    }
 
    assertThat(text, equalTo(originalString));
}
```

## 9. 使用Apache Commons IO进行转换

现在让我们看看如何使用Commons IO库来实现这一点。

这里需要注意的一点是，与Guava不同，这些示例都不会关闭InputStream：

```java
@Test
public void givenUsingCommonsIo_whenConvertingAnInputStreamToAString_thenCorrect() throws IOException {
    String originalString = randomAlphabetic(8);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());

    String text = IOUtils.toString(inputStream, StandardCharsets.UTF_8.name());
    assertThat(text, equalTo(originalString));
}
```

我们还可以使用StringWriter进行转换：

```java
@Test
public void givenUsingCommonsIoWithCopy_whenConvertingAnInputStreamToAString_thenCorrect() throws IOException {
    String originalString = randomAlphabetic(8);
    InputStream inputStream = new ByteArrayInputStream(originalString.getBytes());

    StringWriter writer = new StringWriter();
    String encoding = StandardCharsets.UTF_8.name();
    IOUtils.copy(inputStream, writer, encoding);

    assertThat(writer.toString(), equalTo(originalString));
}
```

## 10. 总结

在本文中，我们学习了如何将InputStream转换为String。我们从使用纯Java开始，然后探讨了如何使用[Guava](https://github.com/google/guava)和[Apache Commons IO](https://commons.apache.org/proper/commons-io/)库。