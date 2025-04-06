---
layout: post
title:  将CSV文件读入数组
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

简而言之，CSV(逗号分隔值)文件包含由逗号分隔符分隔的组织信息。

在本教程中，我们将研究将CSV文件读入数组的不同方法。

## 2. java.io中的BufferedReader

首先，让我们使用BufferedReader中的readLine()逐行读取记录，然后根据逗号分隔符将每行拆分为标记：

```java
List<List<String>> records = new ArrayList<>();
try (BufferedReader br = new BufferedReader(new FileReader("book.csv"))) {
    String line;
    while ((line = br.readLine()) != null) {
        String[] values = line.split(COMMA_DELIMITER);
        records.add(Arrays.asList(values));
    }
}
```

**请注意，更复杂的CSV(例如，引用或包括逗号作为值)将不会按预期使用此方法进行解析**。

## 3. java.util中的Scanner

接下来，让我们使用java.util.Scanner遍历文件内容并逐行检索：

```java
List<List<String>> records = new ArrayList<>();
try (Scanner scanner = new Scanner(new File("book.csv"));) {
    while (scanner.hasNextLine()) {
        records.add(getRecordFromLine(scanner.nextLine()));
    }
}
```

然后我们将解析这些行并将其存储到一个数组中：

```java
private List<String> getRecordFromLine(String line) {
    List<String> values = new ArrayList<String>();
    try (Scanner rowScanner = new Scanner(line)) {
        rowScanner.useDelimiter(COMMA_DELIMITER);
        while (rowScanner.hasNext()) {
            values.add(rowScanner.next());
        }
    }
    return values;
}
```

**与以前一样，使用此方法无法按预期解析更复杂的CSV**。

## 4. OpenCSV

我们可以使用OpenCSV处理更复杂的CSV文件。

**OpenCSV是一个第三方库，它提供了一个API来处理CSV文件**。

我们将使用CSVReader中的readNext()方法来读取文件中的记录：

```java
List<List<String>> records = new ArrayList<List<String>>();
try (CSVReader csvReader = new CSVReader(new FileReader("book.csv"));) {
    String[] values = null;
    while ((values = csvReader.readNext()) != null) {
        records.add(Arrays.asList(values));
    }
}
```

要深入了解有关OpenCSV的更多信息，请查看我们的[OpenCSV教程](https://www.baeldung.com/opencsv)。

## 5. 使用Files实用程序类

或者，我们可以使用Files类来实现相同的目的。此实用程序类由几个对文件和目录进行操作的静态方法组成。那么，让我们看看如何在实践中使用它。

### 5.1 使用Files#lines

lines()方法是Java 8中引入的增强功能之一，它允许我们将给定文件的所有行读取为[Stream](https://www.baeldung.com/java-8-streams)。那么，让我们看看它的实际效果：

```java
try (Stream<String> lines = Files.lines(Paths.get(CSV_FILE))) {
    List<List<String>> records = lines.map(line -> Arrays.asList(line.split(COMMA_DELIMITER)))
        .collect(Collectors.toList());
}
```

这里，Paths.get(CSV_FILE)方法返回一个[Path](https://www.baeldung.com/java-path-vs-file)实例，该实例表示CSV文件的路径。此外，我们使用map()方法将CSV文件的每一行转换为字符串列表。请注意，我们使用[try-with-resources](https://www.baeldung.com/java-try-with-resources)来确保文件在最后自动关闭。

### 5.2 使用Files#readAllLines

类似地，Files提供了readAllLines()方法作为实现相同结果的另一种选择。此方法与lines()一样，接收Path对象作为参数，并直接返回包含指定CSV文件每一行的列表：

```java
List<List<String>> records = Files.readAllLines(Paths.get(CSV_FILE))
    .stream()
    .map(line -> Arrays.asList(line.split(COMMA_DELIMITER)))
    .collect(Collectors.toList());
```


值得注意的是，我们使用Stream API将CSV文件读入List<List<String\>\>。**这里需要注意的一点是readAllLines()会将所有内容一次性放入内存中，因此不要使用它来读取大文件**。

### 5.3 使用Files#newBufferedReader

另一个选择是使用newBufferedReader()方法，**它返回[BufferedReader](https://www.baeldung.com/java-buffered-reader)的一个实例，从而提供一种更高效地读取文件的方法**。

接下来我们通过一个例子来学习如何使用该方法：

```java
try (BufferedReader reader = Files.newBufferedReader(Paths.get(CSV_FILE))) {
    List<List<String>> records = reader.lines()
        .map(line -> Arrays.asList(line.split(COMMA_DELIMITER)))
        .collect(Collectors.toList());
}
```

如上所示，我们使用与之前相同的逻辑来读取CSV文件。**请注意，与其他方法相比，newBufferedReader()是处理大文件的最佳方法**。

## 6. 总结

在这篇简短的文章中，我们探讨了将CSV文件读入数组的不同方法。