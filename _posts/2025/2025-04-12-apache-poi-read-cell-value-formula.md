---
layout: post
title:  使用Apache POI读取Excel单元格值而不是公式
category: apache
copyright: apache
excerpt: Apache POI
---

## 1. 简介

在Java中读取Excel文件时，我们通常需要读取单元格的值来执行某些计算或生成报告。然而，我们可能会遇到一个或多个单元格包含公式而非原始数据值的情况。那么，如何获取这些单元格的实际数据值呢？

在本教程中，我们将研究使用[Apache POI](https://www.baeldung.com/java-microsoft-excel)Java库读取Excel单元格值(而不是计算单元格值的公式)的不同方法。

有两种方法可以解决这个问题：

- 获取单元格的最后一个缓存值
- 在运行时计算公式以获取单元格值

## 2. Maven依赖

我们需要在Apache POI的 pom.xml文件中添加以下依赖：

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.3.0</version>
</dependency>
```

 可以从Maven Central下载最新版本的[poi-ooxml](https://mvnrepository.com/artifact/org.apache.poi/poi-ooxml)。

## 3. 获取最后一个缓存值

当公式计算单元格的值时，Excel会为单元格存储两个对象。一个是公式本身，另一个是缓存值，**缓存值包含公式计算出的最后一个值**。

所以这里的想法是，我们可以获取最后一个缓存值并将其视为单元格值，最后一个缓存值并不总是正确的单元格值，但是，当我们处理已保存的Excel文件，并且该文件最近没有修改时，最后一个缓存值应该是单元格值。

让我们看看如何获取单元格的最后一个缓存值：

```java
FileInputStream inputStream = new FileInputStream(new File("temp.xlsx"));
Workbook workbook = new XSSFWorkbook(inputStream);
Sheet sheet = workbook.getSheetAt(0);

CellAddress cellAddress = new CellAddress("C2");
Row row = sheet.getRow(cellAddress.getRow());
Cell cell = row.getCell(cellAddress.getColumn());

if (cell.getCellType() == CellType.FORMULA) {
    switch (cell.getCachedFormulaResultType()) {
        case BOOLEAN:
            System.out.println(cell.getBooleanCellValue());
            break;
        case NUMERIC:
            System.out.println(cell.getNumericCellValue());
            break;
        case STRING:
            System.out.println(cell.getRichStringCellValue());
            break;
    }
}
```

## 4. 计算公式以获取单元格值

Apache POI提供了一个FormulaEvaluator类，使我们能够计算Excel表中公式的结果。

因此，我们可以使用FormulaEvaluator在运行时直接计算单元格值，FormulaEvaluator类提供了一个名为evaluateFormulaCell的方法，该方法计算给定Cell对象的单元格值，并返回一个CellType对象，该对象表示单元格值的数据类型。

让我们看看这种方法的实际效果：

```java
// existing Workbook setup

FormulaEvaluator evaluator = workbook.getCreationHelper().createFormulaEvaluator(); 

// existing Sheet, Row, and Cell setup

if (cell.getCellType() == CellType.FORMULA) {
    switch (evaluator.evaluateFormulaCell(cell)) {
        case BOOLEAN:
            System.out.println(cell.getBooleanCellValue());
            break;
        case NUMERIC:
            System.out.println(cell.getNumericCellValue());
            break;
        case STRING:
            System.out.println(cell.getStringCellValue());
            break;
    }
}
```

## 5. 选择哪种方法

这里两种方法的简单区别在于，第一种方法使用最后缓存的值，而第二种方法在运行时评估公式。

如果我们使用的是已保存的Excel文件，并且我们不会在运行时对该电子表格进行更改，那么缓存值方法会更好，因为我们不必评估公式。

但是，如果我们知道我们将在运行时进行频繁更改，那么最好在运行时评估公式以获取单元格值。

## 6. 总结

在这篇简短的文章中，我们看到了两种获取Excel单元格值的方法，而不是计算它的公式。