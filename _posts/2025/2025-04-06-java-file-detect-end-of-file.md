---
layout: post
title:  在Java中检测EOF
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 简介

EOF(文件结束)是指我们在读取文件时到达文件末尾的情况；了解EOF检测至关重要，因为在某些应用程序中，我们可能需要[读取](https://www.baeldung.com/reading-file-in-java)配置文件、处理数据或验证文件。在Java中，我们可以通过多种方式检测EOF。

在本教程中，我们将探讨Java中几种EOF检测方法。

## 2. 示例设置

但是，在继续之前，让我们首先创建一个包含虚拟数据的示例文本文件以供测试：

```java
@Test
@Order(0)
public void prepareFileForTest() {
    File file = new File(pathToFile);

    if (!file.exists()) {
        try {
            file.createNewFile();
            FileWriter writer = new FileWriter(file);
            writer.write(LOREM_IPSUM);
            writer.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

此方法必须先于其他方法运行，因为它确保测试文件的存在。因此，我们添加了@Order(0)注解。

## 3. 使用FileInputStream检测EOF

在第一种方法中，我们将使用FileInputStream，它是[InputStream](https://www.baeldung.com/java-mocking-inputstream#basics)的子类。

有一个read()方法，**其工作方式是逐字节读取数据，以便当到达EOF时产生-1的值**。

让我们读取测试文件到文件末尾并将数据存储在[ByteArrayOutputStream](https://www.baeldung.com/java-outputstream#2-bytearrayoutputstream)对象中：

```java
String readWithFileInputStream(String pathFile) throws IOException {
    try (FileInputStream fis = new FileInputStream(pathFile);
         ByteArrayOutputStream baos = new ByteArrayOutputStream()) {
        int data;
        while ((data = fis.read()) != -1) {
            baos.write(data);
        }
        return baos.toString();
    }
}
```

现在让我们创建一个单元测试并确保测试通过：

```java
@Test
@Order(1)
public void givenDummyText_whenReadWithFileInputStream_thenReturnText() {
    try {
        String actualText = eofDetection.readWithFileInputStream(pathToFile);
        assertEquals(LOREM_IPSUM, actualText);
    } catch (IOException e) {
        fail(e.getMessage());
    }
}
```

FileInputStream的优势在于效率-它非常快；不幸的是，它没有逐行读取文本的方法，因此在读取文本文件的情况下，我们必须将字节转换为字符。

因此，**此方法适合读取二进制数据，并提供了逐字节处理的灵活性**。但是，如果我们想以结构化格式读取文本数据，则需要更多的数据转换代码。

## 4. 使用BufferedReader检测EOF

[BufferedReader](https://www.baeldung.com/java-buffered-reader)是java.io包中的一个类，用于从输入流中读取文本。BufferedReader的工作方式是将数据缓冲或临时存储在内存中。

在BufferedReader中，有一个readline()方法，它逐行读取文件，如果到达EOF则返回一个空值：

```java
String readWithBufferedReader(String pathFile) throws IOException {
    try (FileInputStream fis = new FileInputStream(pathFile);
        InputStreamReader isr = new InputStreamReader(fis);
        BufferedReader reader = new BufferedReader(isr)) {
        StringBuilder actualContent = new StringBuilder();
        String line;
        while ((line = reader.readLine()) != null) {
            actualContent.append(line);
        }
        return actualContent.toString();
    }
}
```

这里，文件的内容由readLine()方法逐行读取。然后，将结果存储在actualContent变量中，直到它产生一个表示EOF的空值。

接下来我们来做一个测试，保证结果的准确性：

```java
@Test
@Order(2)
public void givenDummyText_whenReadWithBufferedReader_thenReturnText() {
    try {
        String actualText = eofDetection.readWithBufferedReader(pathToFile);
        assertEquals(LOREM_IPSUM, actualText);
    } catch (IOException e) {
        fail(e.getMessage());
    }
}
```

由于我们有一个readLine()方法，因此**该技术非常适合读取CSV等结构化格式的文本数据**。但是，它不适合读取二进制数据。

## 5. 使用Scanner检测EOF

[Scanner](https://www.baeldung.com/java-scanner)是java.util包中的一个类，可用于读取各种类型的数据输入，例如文本、整数等。

**Scanner提供了一个hasNext()方法来读取文件的全部内容，直到产生一个false值，表示EOF**：

```java
String readWithScanner(String pathFile) throws IOException{
    StringBuilder actualContent = new StringBuilder();
    File file = new File(pathFile);
    Scanner scanner = new Scanner(file);
    while (scanner.hasNext()) {
    	String line = scanner.nextLine();
        actualContent.append(line);
    }
    return actualContent.toString();
}
```

只要hasNext()的计算结果为true，我们就可以观察Scanner如何读取文件，这意味着我们可以使用nextLine()方法从Scanner中检索字符串值，直到hasNext()的计算结果为false，这表明我们已经到达了EOF。

让我们测试一下以确保该方法正常工作：

```java
@Test
@Order(3)
public void givenDummyText_whenReadWithScanner_thenReturnText() {
    try {
        String actualText = eofDetection.readWithScanner(pathToFile);
        assertEquals(LOREM_IPSUM, actualText);
    } catch (IOException e) {
        fail(e.getMessage());
    }
}
```

**这种方式的优点是灵活，可以轻松读取各种类型的数据，但对于二进制数据来说，这种方式不太理想**，性能可能比BufferedReader稍慢，不适合读取二进制数据。

## 6. 使用FileChannel和ByteBuffer检测EOF

[FileChannel](https://www.baeldung.com/java-filechannel)和[ByteBuffer](https://www.baeldung.com/java-bytebuffer)是Java NIO(新I/O)中的类，是对传统I/O的改进。

FileChannel函数用于处理文件输入和输出操作，而ByteBuffer用于有效地处理字节数组形式的二进制数据。

对于EOF检测，我们将使用这两个类-FileChannel用于读取文件，ByteBuffer用于存储结果。我们使用的方法是**读取缓冲区，直到它返回值-1，这表示文件结束(EOF)**：

```java
String readFileWithFileChannelAndByteBuffer(String pathFile) throws IOException {
    try (FileInputStream fis = new FileInputStream(pathFile);
        FileChannel channel = fis.getChannel()) {
        ByteBuffer buffer = ByteBuffer.allocate((int) channel.size());
        while (channel.read(buffer) != -1) {
            buffer.flip();
            buffer.clear();
        }
        return StandardCharsets.UTF_8.decode(buffer).toString();
    }
}
```

这次我们不需要使用[StringBuilder](https://www.baeldung.com/java-string-builder-string-buffer)，因为我们可以从转换或解码后的ByteBuffer对象中获取读取文件的结果。

让我们再次测试以确保该方法有效：

```java
@Test
@Order(4)
public void givenDummyText_whenReadWithFileChannelAndByteBuffer_thenReturnText() {
    try {
        String actualText = eofDetection.readFileWithFileChannelAndByteBuffer(pathToFile);
        assertEquals(LOREM_IPSUM, actualText);
    } catch (IOException e) {
        fail(e.getMessage());
    }
}
```

**此方法在从文件读取或向文件写入数据时提供高性能，适合随机访问，并支持MappedByteBuffer**。然而，它的使用更为复杂，需要细致的缓冲区管理。

它特别适合读取二进制数据和需要随机文件访问的应用程序。

## 7. FileInputStream、BufferedReader、Scanner、FileChannel和ByteBuffer

下表总结了这四种方法的比较，每种方法都有优点和缺点：

|  特征  | FileInputStream | BufferedReader | Scanner | FileChannel和ByteBuffer |
|:----:|:---------------:|:--------------:|:-------:|:----------------------:|
| 数据类型 |       二进制       |     结构化文本      |  结构化文本  |          二进制           |
|  性能  |        好        |       好        |    好    |           出色           |
| 灵活性  |        高        |       中等       |    高    |           低            |
| 易于使用 |        低        |       高        |    高    |           低            |

## 8. 总结

在本文中，我们学习了Java中EOF检测的四种方法。

每种方法都有其优点和缺点，正确的选择取决于我们应用程序的具体需求，它是否涉及读取结构化文本数据或二进制数据，以及性能在我们的用例中有多重要。