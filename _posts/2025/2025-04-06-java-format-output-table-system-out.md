---
layout: post
title:  使用System.out以表格格式格式化输出
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

在Java应用程序中，经常需要以表格形式显示数据。System.out提供了多种方法来实现这一点，从简单的字符串拼接到高级格式化技术。

在本教程中，我们将探讨使用System.out以表状结构格式化输出的各种方法。

## 2. 使用字符串拼接

格式化表格输出的最简单方法是使用字符串拼接，虽然这种方法很简单，但需要手动调整空格和对齐方式：

```java
void usingStringConcatenation() {
    String[] headers = {"ID", "Name", "Age"};
    String[][] data = {
        {"1", "James", "24"},
        {"2", "Sarah", "27"},
        {"3", "Keith", "31"}
    };

    System.out.println(headers[0] + "   " + headers[1] + "   " + headers[2]);

    for (String[] row : data) {
        System.out.println(row[0] + "   " + row[1] + "   " + row[2]);
    }
}
```

预期输出为：

```text
ID   Name   Age
1   James   24
2   Sarah   27
3   Keith   31
```

**虽然这种方法有效，但在处理动态数据时会变得麻烦，因为它需要手动调整以保持一致**。

## 3. 使用printf()

[System.out.printf()](https://www.baeldung.com/java-printstream-printf)方法提供了一种更灵活的格式化字符串的方法，我们可以指定每列的宽度并确保正确对齐：

```java
void usingPrintf() {
    String[] headers = {"ID", "Name", "Age"};
    String[][] data = {
        {"1", "James", "24"},
        {"2", "Sarah", "27"},
        {"3", "Keith", "31"}
    };

    System.out.printf("%-5s %-10s %-5s%n", headers[0], headers[1], headers[2]);

    for (String[] row : data) {
        System.out.printf("%-5s %-10s %-5s%n", row[0], row[1], row[2]);
    }
}
```

预期输出如下：

```text
ID    Name       Age  
1     James      24   
2     Sarah      27   
3     Keith      31  
```

在printf方法中，%-5s指定一个左对齐的字符串，宽度为5个字符。而%-10s指定一个左对齐的字符串，宽度为10个字符。

**这种方法更加简洁，并且可以确保无论数据的长度如何，列都是对齐的**。

## 4. 使用String.format()

如果我们需要将格式化的输出存储在字符串中而不是直接打印，我们可以使用[String.format()](https://www.baeldung.com/string/format)，此方法使用与printf()相同的格式化语法。

```java
void usingStringFormat() {
    String[] headers = {"ID", "Name", "Age"};
    String[][] data = {
            {"1", "James", "24"},
            {"2", "Sarah", "27"},
            {"3", "Keith", "31"}
    };

    String header = String.format("%-5s %-10s %-5s", headers[0], headers[1], headers[2]);
    System.out.println(header);

    for (String[] row : data) {
        String formattedRow = String.format("%-5s %-10s %-5s", row[0], row[1], row[2]);
        System.out.println(formattedRow);
    }
}
```

**输出与printf()示例相同，不同之处在于String.format()返回格式化的字符串，我们可以使用它进行进一步处理或记录**：

```text
ID    Name       Age  
1     James      24   
2     Sarah      27   
3     Keith      31   
```

## 5. 使用StringBuilder

处理动态或大量数据时，使用[StringBuilder](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/StringBuilder.html)构建表会更高效，这种方法允许我们将整个输出构建为单个字符串并打印一次：

```java
void usingStringBuilder() {
    String[] headers = {"ID", "Name", "Age"};
    String[][] data = {
            {"1", "James", "24"},
            {"2", "Sarah", "27"},
            {"3", "Keith", "31"}
    };

    StringBuilder table = new StringBuilder();

    table.append(String.format("%-5s %-10s %-5s%n", headers[0], headers[1], headers[2]));

    for (String[] row : data) {
        table.append(String.format("%-5s %-10s %-5s%n", row[0], row[1], row[2]));
    }

    System.out.print(table.toString());
}
```

预期输出：

```text
ID    Name       Age  
1     James      24   
2     Sarah      27   
3     Keith      31   
```

**当我们需要创建复杂的输出或需要考虑性能时，此方法特别有用，因为StringBuilder减少了多字符串拼接的开销**。

## 6. 使用ASCII表

我们可以使用外部库，如[ASCII Table](https://github.com/vdmeer/asciitable)，它是一个第三方库，可以轻松创建美观的ASCII表。

要使用ASCII表方法，我们需要在我们的项目中包含它的[依赖](https://mvnrepository.com/artifact/ascii-table/ascii-table/1.0)：

```xml
<dependency>
    <groupId>de.vandermeer</groupId>
    <artifactId>asciitable</artifactId>
    <version>0.3.2</version>
</dependency>
```

然后我们可以使用它的方法来创建一个类似表格的结构，下面是一个使用ASCII Table的示例：

```java
void usingAsciiTable() {
    AsciiTable table = new AsciiTable();
    table.addRule();
    table.addRow("ID", "Name", "Age");
    table.addRule();
    table.addRow("1", "James", "24");
    table.addRow("2", "Sarah", "27");
    table.addRow("3", "Keith", "31");
    table.addRule();
    String renderedTable = table.render();
    System.out.println(renderedTable);
}
```

预期输出为：

```text
┌──────────────────────────┬─────────────────────────┬─────────────────────────┐
│ID                        │Name                     │Age                      │
├──────────────────────────┼─────────────────────────┼─────────────────────────┤
│1                         │James                    │24                       │
│2                         │Sarah                    │27                       │
│3                         │Keith                    │31                       │
└──────────────────────────┴─────────────────────────┴─────────────────────────┘

```

## 7. 总结

在本教程中，我们探索了使用System.out以表格形式格式化输出的各种方法。对于简单的任务，基本字符串拼接或printf效果很好。但是，对于更动态或更复杂的输出，使用StringBuilder可能更有效；ASCII表有助于生成干净且格式良好的输出。通过选择适合我们要求的正确方法，我们可以生成简洁、可读且结构良好的控制台输出。