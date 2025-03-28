---
layout: post
title:  如何使用自定义列标题和位置从POJO创建CSV文件
category: libraries
copyright: libraries
excerpt: OpenCSV
---

## 1. 概述

CSV是系统和应用程序之间常见的数据交换格式之一，一个常见的用例是构建[处理这些CSV文件的Java应用程序](https://www.baeldung.com/java-csv)。在将数据写入CSV文件时，我们需要将普通旧式Java对象(POJO)映射到CSV格式。

在本教程中，我们将学习如何使用自定义位置和标头名称将POJO映射到CSV格式。

## 2. OpenCSV库

[OpenCSV](https://mvnrepository.com/artifact/com.opencsv/opencsv)是一个非常流行的处理CSV文件的库，我们首先需要向项目添加Maven依赖：

```xml
<dependency>
    <groupId>com.opencsv</groupId>
    <artifactId>opencsv</artifactId>
    <version>5.9</version>
</dependency>
```

## 3. 生成输入记录

对于本文，我们将生成映射到CSV记录的示例输入记录。

### 3.1 Application记录

Application[记录](https://www.baeldung.com/java-record-keyword)是一个简单的POJO，包含id、name、age和created_at字段：

```java
public record Application(String id, String name, Integer age, String created_at) {}
```

### 3.2 Application列表

我们将生成一个Application列表，稍后将其转换为CSV格式：

```java
List<Application> applications = List.of(
        new Application("123", "Sam", 34, "2023-08-11"),
        new Application("456", "Tam", 44, "2023-02-11"),
        new Application("890", "Jam", 54, "2023-03-11")
);
```

## 4. POJO转CSV

POJO到CSV的默认映射很简单，我们可以使用StatefulBeanToCsvBuilder并定义分隔符和其他配置，然后使用FilerWriter进行写入：

```java
public static void beanToCSVWithDefault(List<Application> applications) throws Exception {
    try (FileWriter writer = new FileWriter("application.csv")) {
        var builder = new StatefulBeanToCsvBuilder<Application>(writer)
                .withQuotechar(CSVWriter.NO_QUOTE_CHARACTER)
                .withSeparator(',')
                .build();

        builder.write(applications);
    }
}
```

**默认映射将POJO转换为具有升序字段名称的CSV，但如果我们想要自定义格式的标头，这没有帮助**。

### 4.1 自定义标头策略

如果我们希望输出的CSV文件带有自定义标头，我们可以在写入CSV文件时定义并使用自定义标头策略。

例如，如果我们希望标头具有小写名称，我们可以覆盖generateHeader：

```java
public class CustomCSVWriterStrategy<T> extends HeaderColumnNameMappingStrategy<T> {
    @Override
    public String[] generateHeader(T bean) throws CsvRequiredFieldEmptyException {
        String[] header = super.generateHeader(bean);
        return Arrays.stream(header)
                .map(String::toLowerCase)
                .toArray(String[]::new);
    }
}
```

一旦我们有了自定义的头映射策略，我们就可以构建StatefulBeanToCsvBuilder，然后以CSV格式将POJO写入CSV文件：

```java
public static void beanToCSVWithCustomHeaderStrategy(List<Application> applications)
        throws IOException, CsvRequiredFieldEmptyException, CsvDataTypeMismatchException {
    try (FileWriter writer = new FileWriter("application.csv")){
        var mappingStrategy = new CustomCSVWriterStrategy<Application>();
        mappingStrategy.setType(Application.class);

        var builder = new StatefulBeanToCsvBuilder<Application>(writer)
                .withQuotechar(CSVWriter.NO_QUOTE_CHARACTER)
                .withMappingStrategy(mappingStrategy)
                .build();
        builder.write(applications);
    }
}
```

### 4.2 自定义位置策略

**也可以编写一个具有特定定义位置的CSV字段的CSV文件**，我们要做的就是用所需的字段位置标注Application记录属性：

```java
public record Application(
        @CsvBindByPosition(position = 0)
        String id,
        @CsvBindByPosition(position = 1)
        String name,
        @CsvBindByPosition(position = 2)
        Integer age,
        @CsvBindByPosition(position = 3)
        String created_at) {}
```

现在，我们可以使用带注解的Application记录将POJO写入CSV格式：

```java
public static void beanToCSVWithCustomPositionStrategy(List<Application> applications) throws Exception {
    try (FileWriter writer = new FileWriter("application3.csv")) {
        var builder = new StatefulBeanToCsvBuilder<Application1>(writer)
                .withQuotechar(CSVWriter.NO_QUOTE_CHARACTER)
                .build();

        builder.write(applications);
    }
}
```

### 4.3 自定义位置策略和标头名称

**如果我们希望CSV字段与标头一起位于某个位置，我们可以使用自定义列位置策略来实现**。

首先，我们需要用位置和标题名称注释应用程序记录类属性：

```java
public record Application1(
        @CsvBindByName(column = "id", required = true)
        @CsvBindByPosition(position = 1)
        String id,
        @CsvBindByName(column = "name", required = true)
        @CsvBindByPosition(position = 0)
        String name,
        @CsvBindByName(column = "age", required = true)
        @CsvBindByPosition(position = 2)
        Integer age,
        @CsvBindByName(column = "position", required = true)
        @CsvBindByPosition(position = 3)
        String created_at) {}
```

一旦记录类准备好了，我们需要编写自定义列位置策略。

该策略将包括生成标头和字段位置的逻辑：

```java
public class CustomColumnPositionStrategy<T> extends ColumnPositionMappingStrategy<T> {
    @Override
    public String[] generateHeader(T bean) throws CsvRequiredFieldEmptyException {
        super.generateHeader(bean);
        return super.getColumnMapping();
    }
}
```

现在我们已经完成了记录和列位置策略，我们准备在客户端逻辑中使用它们并生成CSV文件：

```java
public static void beanToCSVWithCustomHeaderAndPositionStrategy(List<Application1> applications)
        throws IOException, CsvRequiredFieldEmptyException, CsvDataTypeMismatchException {
    try (FileWriter writer = new FileWriter("application4.csv")){
        var mappingStrategy = new CustomColumnPositionStrategy<Application1>();
        mappingStrategy.setType(Application1.class);

        var builder = new StatefulBeanToCsvBuilder<Application1>(writer)
                .withQuotechar(CSVWriter.NO_QUOTE_CHARACTER)
                .withMappingStrategy(mappingStrategy)
                .build();
        builder.write(applications);
    }
}
```

## 5. 总结

在本教程中，我们学习了如何将POJO转换为CSV格式并写入CSV文件。我们讨论并编写了包含标头和CSV字段的示例。此外，我们还编写了自定义标头和位置策略。

OpenCSV没有提供用于自定义CSV文件位置和标头的现成解决方案，但可以轻松编写自定义策略以在所需位置包含字段和标头。