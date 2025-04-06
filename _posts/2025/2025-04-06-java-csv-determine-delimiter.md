---
layout: post
title:  如何确定CSV文件中的分隔符
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 简介

在[CSV](https://www.baeldung.com/java-csv)(逗号分隔值)文件中，分隔符用于分隔数据字段，并且分隔符在不同的文件中可能有所不同。[常见的分隔符](https://www.baeldung.com/java-string-split-multiple-delimiters)包括逗号、分号或制表符。在处理CSV文件时，识别正确的分隔符至关重要，因为它可以确保准确解析并防止数据损坏。

在本教程中，我们将探讨如何确定CSV文件中的分隔符。

## 2. 了解CSV文件中的分隔符

CSV文件中的分隔符用于分隔记录中的各个字段,最常见的分隔符是：

- **逗号(,)**：大多数CSV文件的标准
- **分号(;)**：通常用于以逗号作为小数分隔符的区域
- **制表符(\\t)**：通常出现在制表符分隔值文件中
- **管道符(|)**：偶尔用于避免与更传统的分隔符发生冲突

处理CSV文件时，我们必须使用正确的分隔符才能正确解析数据。

例如，假设我们有一个包含以下内容的CSV文件：

```java
Location,Latitude,Longitude,Elevation(m)
New York,40.7128,-74.0060,10
Los Angeles,34.0522,-118.2437,71
```

我们可以看到，字段之间用逗号(,)分隔。

## 3. 简单行采样

确定分隔符的一种方法是从文件中抽取几行并计算常见分隔符的出现次数。

首先，让我们定义将在多行中测试的可能的分隔符：

```java
private static final char[] POSSIBLE_DELIMITERS = {',', ';', '\t', '|'};
```

**我们可以假设跨行出现最频繁的字符可能是分隔符**：

```java
@Test
public void givenCSVLines_whenDetectingDelimiterUsingFrequencyCount_thenCorrectDelimiterFound() {
    char[] POSSIBLE_DELIMITERS = {',', ';', '\t', '|'};
    Map<Character, Integer> delimiterCounts = new HashMap<>();

    for (char delimiter : POSSIBLE_DELIMITERS) {
        int count = 0;
        for (String line : lines) {
            count += line.length() - line.replace(String.valueOf(delimiter), "").length();
        }
        delimiterCounts.put(delimiter, count);
    }

    char detectedDelimiter = delimiterCounts.entrySet().stream()
            .max(Map.Entry.comparingByValue())
            .map(Map.Entry::getKey)
            .orElse(',');

    assertEquals(',', detectedDelimiter);
}
```

在这个方法中，我们使用replace()从行中删除分隔符，length()中的差异计算每个分隔符出现的次数，然后我们将计数存储在HashMap中。最后，我们使用[stream().max()](https://www.baeldung.com/java-collection-min-max)找到计数最高的分隔符并返回它。**如果该方法未找到分隔符，则使用orElse()方法默认为逗号**。

## 4. 使用采样的动态分隔符检测

**检测分隔符的更强大的方法是取第一行中的所有字符的集合，然后抽取其他行来测试哪个字符始终导致相同数量的列**：

```java
@Test
public void givenCSVLines_whenDetectingDelimiter_thenCorrectDelimiterFound() {
    String[] lines = {
            "Location,Latitude,Longitude,Elevation(m)",
            "New York,40.7128,-74.0060,10",
            "Los Angeles,34.0522,-118.2437,71",
            "Chicago,41.8781,-87.6298,181"
    };

    char detectedDelimiter = ',';
    for (char delimiter : lines[0].toCharArray()) {
        boolean allRowsHaveEqualColumnCounts = Arrays.stream(lines)
                .map(line -> line.split(Pattern.quote(String.valueOf(delimiter))))
                .map(columns -> columns.length)
                .distinct()
                .count() == 1;

        if (allRowsHaveEqualColumnCounts) {
            detectedDelimiter = delimiter;
            break;
        }
    }

    assertEquals(',', detectedDelimiter);
}
```

在这种方法中，我们迭代第一行中的每个字符，将每个字符视为潜在的分隔符。然后我们检查此字符是否在所有行中产生一致的列数，该方法使用split()拆分每一行，并使用[Pattern.quote()](https://www.baeldung.com/regular-expressions-java)处理特殊字符，例如|或\\t。

对于每个潜在的分隔符，我们使用它来拆分所有行并计算每行的列数(字段数)。此外，该算法的关键部分是通过使用[distinct()](https://www.baeldung.com/java-streams-distinct-by)检查列数是否均匀来验证所有行的列数是否保持一致。

最后，如果所考虑的分隔符为每一行产生一致的列数，我们假设它是正确的分隔符。**如果在行间未找到一致的分隔符，我们也会默认使用逗号**。

## 5. 总结

在本文中，我们探讨了两种检测CSV文件中分隔符的方法。第一种方法使用简单的行采样并计算潜在分隔符的出现次数。第二种方法更为可靠，它检查多行中的列数是否一致，以确定正确的分隔符。可以根据CSV文件的复杂性应用任一方法。