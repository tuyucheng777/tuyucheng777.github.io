---
layout: post
title:  使用Java将文件读入二维数组
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 简介

从文件中[读取数据](https://www.baeldung.com/reading-file-in-java)是许多应用程序中的常见任务，处理结构化数据(如[CSV文件](https://www.baeldung.com/java-csv))时，我们可以将内容存储在[二维数组](https://www.baeldung.com/java-jagged-arrays)中，以便于访问和操作。

在本教程中，我们将介绍使用Java将文件读入二维数组的三种方法：BufferedReader、Java 8非阻塞IO(NIO) API和Apache Commons CSV库。

## 2. 问题陈述

表示外部文件(例如CSV文件)数据的常用方法是通过二维数组，其中每行对应文件中的一行，每列包含单独的数据值。

例如，如果文件包含以下内容：

```text
value1,value2,value3
value4,value5,value6
value7,value8,value9
```

我们可以将此数据加载到二维数组中，例如：

```java
String[][] expectedData = {
    {"value1", "value2", "value3"},
    {"value4", "value5", "value6"},
    {"value7", "value8", "value9"}
};
```

挑战在于选择正确的方法来有效地读取和处理文件，同时确保更大或更复杂的数据集的灵活性。

## 3. 使用BufferedReader

将文件读入二维数组的最直接的方法是使用[BufferedReader](https://www.baeldung.com/java-buffered-reader)：

```java
try (BufferedReader br = new BufferedReader(new FileReader(filePath))) {
    List<String[]> dataList = new ArrayList<>();
    String line;
    while ((line = br.readLine()) != null) {
        dataList.add(line.split(","));
    }
    return dataList.toArray(new String[0][]);
}
```

首先，我们创建一个带有FileReader的BufferedReader，这样我们就可以高效地从文件中读取每一行。我们将使用ArrayList来动态存储数据。

此外，[try-with-resources](https://www.baeldung.com/java-try-with-resources)语句可确保在读取文件后自动关闭BufferedReader，从而防止资源泄漏。动态大小的ArrayList(称为dataList)将每个文件行存储为字符串数组。

在循环中，br.readLine()逐行读取文件中的每一行，直到读取完所有行，之后readLine()返回null，然后我们将每行split()成一个字符串数组。最后，我们使用toArray()方法将ArrayList转换为二维数组。

此外，让我们通过将从文件读取的实际数据与预期的二维数组进行比较来测试这个实现：

```java
@Test
public void givenFile_whenUsingBufferedReader_thenArrayIsCorrect() throws IOException {
    String[][] actualData = readFileTo2DArrayUsingBufferedReader("src/test/resources/test_file.txt");

    assertArrayEquals(expectedData, actualData);
}
```

## 4. 使用java.nio.file.Files

在Java 8中，我们可以利用[java.nio.file.Files](https://docs.oracle.com/javase/23/docs/api/java/nio/file/Files.html)类以更简洁的方式将文件读入二维数组，我们可以使用readAllLines()方法一次从文件读取所有行，而不必使用BufferedReader手动读取每一行：

```java
List<String> lines = Files.readAllLines(Paths.get(filePath));
return lines.stream()
    .map(line -> line.split(","))
    .toArray(String[][]::new);
```

在这里，我们从文件中读取所有行并将它们存储在List中。接下来，我们使用lines.stream()方法将行列表转换为Stream。随后，我们使用map()转换流中的每一行，其中line.split()创建一个字符串数组。最后，我们使用toArray()将结果收集到二维数组中。

让我们测试这种方法，以确保它也能将文件读入预期的二维数组：

```java
@Test
public void givenFile_whenUsingStreamAPI_thenArrayIsCorrect() throws IOException {
    String[][] actualData = readFileTo2DArrayUsingStreamAPI("src/test/resources/test_file.txt");

    assertArrayEquals(expectedData, actualData);
}
```

**请注意，这种方法提高了可读性并减少了样板代码的数量，使其成为现代Java应用程序的绝佳选择**。

## 5. 使用Apache Commons CSV

对于更复杂的CSV文件格式，[Apache Commons CSV](https://www.baeldung.com/apache-commons-csv)库提供了强大而灵活的解决方案。要读取CSV文件，我们首先通过使用CSVFormat类读取文件来创建CSVParser实例：

```java
Reader reader = new FileReader(filePath);
CSVParser csvParser = new CSVParser(reader, CSVFormat.DEFAULT);
```

在此代码片段中，我们还使用FileReader读取指定文件的内容，CSVParser使用读取器初始化并配置为使用默认的CSV格式。

接下来，我们可以遍历CSV文件中的记录并将它们存储在列表中：

```java
List<String[]> dataList = new ArrayList<>();
for (CSVRecord record : csvParser) {
    String[] data = new String[record.size()];
    for (int i = 0; i < record.size(); i++) {
        data[i] = record.get(i);
    }
    dataList.add(data);
}

return dataList.toArray(new String[dataList.size()][]);
```

**每个CSVRecord为一行，每个记录中的值是列**。最后，我们将得到一个完全填充文件数据的二维数组。

让我们使用Apache Commons CSV测试此方法，以确认它是否按预期工作：

```java
@Test
public void givenFile_whenUsingApacheCommonsCSV_thenArrayIsCorrect() throws IOException {
    String[][] actualData = readFileTo2DArrayUsingApacheCommonsCSV("src/test/resources/test_file.csv");

    assertArrayEquals(expectedData, actualData);
}
```

**Apache Commons CSV库处理多种类型的字符串清理，例如带引号的字段、不同的分隔符和转义字符，使其成为大型复杂CSV文件的绝佳解决方案**。

## 6. 总结

本教程中讨论的每种方法都提供了一种在Java中将文件读入二维数组的独特方法，BufferedReader方法让我们可以完全控制读取过程，而Java NIO API则提供了更简洁、更实用的解决方案。Apache Commons CSV库是更复杂的CSV数据的绝佳选择，可提供更大的灵活性和稳健性。