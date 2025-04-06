---
layout: post
title:  将CSV标头读入列表
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

在这个简短的教程中，我们将探索在Java中将[CSV](https://www.baeldung.com/java-csv-file-array)标头读入列表的不同方法。

首先，我们将学习如何使用JDK类来实现这一点。然后，我们将了解如何使用外部库(例如[OpenCSV](https://www.baeldung.com/opencsv)和[Apache Commons CSV)](https://www.baeldung.com/apache-commons-csv)实现相同的目标。

## 2. 使用BufferedReader

[BufferedReader](https://www.baeldung.com/java-buffered-reader)类提供了解决我们挑战的最简单的解决方案，**它提供了一种快速有效的方法来读取CSV文件，因为它通过逐块读取内容来减少IO操作的数量**。

那么，让我们看看它的实际效果：

```java
class CsvHeadersAsListUnitTest {

    private static final String CSV_FILE = "src/test/resources/employees.csv";
    private static final String COMMA_DELIMITER = ",";
    private static final List<String> EXPECTED_HEADERS = List.of("ID", "First name", "Last name", "Salary");

    @Test
    void givenCsvFile_whenUsingBufferedReader_thenGetHeadersAsList() throws IOException {
        try (BufferedReader reader = new BufferedReader(new FileReader(CSV_FILE))) {
            String csvHeadersLine = reader.readLine();
            List<String> headers = Arrays.asList(csvHeadersLine.split(COMMA_DELIMITER));
            assertThat(headers).containsExactlyElementsOf(EXPECTED_HEADERS);
        }
    }
}
```

如我们所见，我们使用[try-with-resources](https://www.baeldung.com/java-try-with-resources)创建了一个BufferedReader实例，这样，我们就可以确保文件随后被关闭。**此外，我们调用一次[readLine()](https://www.baeldung.com/java-buffered-reader#2-reading-line-by-line)方法来提取表示标头的第一行**。最后，我们使用[split()](https://www.baeldung.com/string/split)方法和[Arrays#asList](https://www.baeldung.com/java-arrays-aslist-vs-new-arraylist#arraysaslist)来获取标头作为列表。

## 3. 使用Scanner

[Scanner](https://www.baeldung.com/java-scanner)类提供了另一种解决方案来实现相同的结果，顾名思义，**它扫描并读取给定文件的内容**。因此，让我们添加另一个测试用例来查看如何使用Scanner读取CSV文件头：

```java
@Test
void givenCsvFile_whenUsingScanner_thenGetHeadersAsList() throws IOException {
    try(Scanner scanner = new Scanner(new File(CSV_FILE))) {
        String csvHeadersLine = scanner.nextLine();
        List<String> headers = Arrays.asList(csvHeadersLine.split(COMMA_DELIMITER));
        assertThat(headers).containsExactlyElementsOf(EXPECTED_HEADERS);
    }
}
```

类似地，Scanner类具有[nextLine()](https://www.baeldung.com/java-scanner-nextline)方法，我们可以使用它来获取输入文件的第一行。这里，第一行代表我们的CSV文件的标头。

## 4. 使用OpenCSV

或者，我们可以使用OpenCSV库来读取特定CSV文件的标头。在深入了解细节之前，让我们将其[Maven依赖](https://mvnrepository.com/artifact/com.opencsv/opencsv)添加到pom.xml文件中：

```xml
<dependency>
    <groupId>com.opencsv</groupId>
    <artifactId>opencsv</artifactId>
    <version>5.9</version>
</dependency>
```

通常，**OpenCSV带有一组用于读取和解析CSV文件的现成类和方法**。因此，让我们通过一个实际的例子来说明这个库的使用：

```java
@Test
void givenCsvFile_whenUsingOpenCSV_thenGetHeadersAsList() throws CsvValidationException, IOException {
    try (CSVReader csvReader = new CSVReader(new FileReader(CSV_FILE))) {
        List<String> headers = Arrays.asList(csvReader.readNext());
        assertThat(headers).containsExactlyElementsOf(EXPECTED_HEADERS);
    }
}
```

如上所示，OpenCSV提供了CSVReader类来读取给定文件的内容，**CSVReader类提供了readNext()方法，可以直接以字符串数组的形式检索下一行**。

## 5. 使用Apache Commons CSV

另一个解决方案是使用Apache Commons CSV库，顾名思义，它提供了几个用于创建和读取CSV文件的便捷功能。

首先，我们需要将其[依赖](https://mvnrepository.com/artifact/org.apache.commons/commons-csv)的最新版本添加到pom.xml中：

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-csv</artifactId>
    <version>1.11.0</version>
</dependency>
```

**简而言之，Apache Commons CSV的CSVParser类提供了getHeaderNames()方法来返回标头名称的只读列表**：

```java
@Test
void givenCsvFile_whenUsingApacheCommonsCsv_thenGetHeadersAsList() throws IOException {
    CSVFormat csvFormat = CSVFormat.DEFAULT.builder()
        .setDelimiter(COMMA_DELIMITER)
        .setHeader()
        .build();

    try (BufferedReader reader = new BufferedReader(new FileReader(CSV_FILE));
        CSVParser parser = CSVParser.parse(reader, csvFormat)) {
        List<String> headers = parser.getHeaderNames();
        assertThat(headers).containsExactlyElementsOf(EXPECTED_HEADERS);
    }
}
```

这里，我们使用CSVParser类根据指定的格式解析输入文件，**在[setHeader()](https://commons.apache.org/proper/commons-csv/apidocs/org/apache/commons/csv/CSVFormat.Builder.html#setHeader(java.lang.String...))方法的帮助下，会自动从输入文件中解析标头**。

## 6. 总结

在这篇短文中，我们探讨了将CSV文件标头读取为列表的不同解决方案。

首先，我们学习了如何使用JDK来实现这一点。然后，我们了解了如何使用外部库来实现相同的目标。