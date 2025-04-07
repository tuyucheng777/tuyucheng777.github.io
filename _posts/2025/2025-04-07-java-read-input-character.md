---
layout: post
title:  在Java中逐个字符地读取输入
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 简介

在许多Java应用程序中，我们需要逐个字符地读取输入数据，因为这是一项常见任务，尤其是在处理来自流源的大量数据时。

**在本教程中，我们将研究在Java中一次读取一个字符的各种方法**。

## 2. 使用BufferedReader进行控制台输入

我们可以利用[BufferedReader](https://www.baeldung.com/java-buffered-reader)从控制台逐个字符地读取，**请注意，如果我们想要以交互方式读取字符，这种方法很有用**。

让我们举个例子：

```java
@Test
public void givenInputFromConsole_whenUsingBufferedStream_thenReadCharByChar() throws IOException {
    ByteArrayInputStream inputStream = new ByteArrayInputStream("TestInput".getBytes());
    System.setIn(inputStream);

    try (BufferedReader buffer = new BufferedReader(new InputStreamReader(System.in))) {
        char[] result = new char[9];
        int index = 0;
        int c;
        while ((c = buffer.read()) != -1) {
            result[index++] = (char) c;
        }

        assertArrayEquals("TestInput".toCharArray(), result);
    }
}
```

在这里，我们简单地通过使用“TestInput”内容实例化ByteArrayInputStream来模拟控制台输入。然后，我们使用BufferedReader从System.in读取字符。之后，我们使用read()方法将一个字符读取为整数码并将其转换为char。最后，我们使用assertArrayEquals()方法来验证读取的字符是否与预期输入匹配。

## 3. 使用FileReader读取文件

当处理文件时，[FileReader](https://www.baeldung.com/java-filereader)是逐个字符读取的合适选择：

```java
@Test
public void givenInputFromFile_whenUsingFileReader_thenReadCharByChar() throws IOException {
    File tempFile = File.createTempFile("tempTestFile", ".txt");
    FileWriter fileWriter = new FileWriter(tempFile);
    fileWriter.write("TestFileContent");
    fileWriter.close();

    try (FileReader fileReader = new FileReader(tempFile.getAbsolutePath())) {
        char[] result = new char[15];
        int index = 0;
        int charCode;
        while ((charCode = fileReader.read()) != -1) {
            result[index++] = (char) charCode;
        }

        assertArrayEquals("TestFileContent".toCharArray(), result);
    }
}
```

在上面的代码中，我们创建了一个临时测试文件，内容为“tempTestFile”，用于模拟。然后，我们使用FileReader通过tempFile.getAbsolutePath()方法与路径指定的文件建立连接，在[try-with-resources](https://www.baeldung.com/java-try-with-resources)块中，我们逐个字符地读取文件。

## 4. 使用Scanner进行标记化输入

对于允许标记化输入的更动态的方法，我们可以使用[Scanner](https://www.baeldung.com/java-scanner)：

```java
@Test
public void givenInputFromConsole_whenUsingScanner_thenReadCharByChar() {
    ByteArrayInputStream inputStream = new ByteArrayInputStream("TestInput".getBytes());
    System.setIn(inputStream);

    try (Scanner scanner = new Scanner(System.in)) {
        if (scanner.hasNext()) {
            char[] result = scanner.next().toCharArray();
            assertArrayEquals("TestInput".toCharArray(), result);
        }
    }
}
```

我们通过使用“TestInput”内容实例化ByteArrayInputStream来模拟上述测试方法中的控制台输入，**然后，我们使用hasNext()方法来验证是否还有另一个标记。之后，我们使用next()方法将当前标记作为String获取**。

## 5. 总结

总之，我们探索了Java中读取字符的各种方法，包括使用BufferedReader进行交互式控制台输入、使用FileReader进行基于文件的字符读取以及通过Scanner进行标记化输入处理，为开发人员提供了在各种场景中有效处理字符数据的多种方法。