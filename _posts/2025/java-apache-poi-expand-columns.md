---
layout: post
title:  使用Apache POI扩展列
category: apache
copyright: apache
excerpt: Apache POI
---

## 1. 概述

Apache POI是一种流行的Java API，用于以编程方式操作不同类型的Microsoft Office文档，例如[Word](https://www.baeldung.com/java-microsoft-word-with-apache-poi)、[Excel](https://www.baeldung.com/java-microsoft-excel)和[PowerPoint](https://www.baeldung.com/apache-poi-slideshow)。

**我们经常需要在Excel电子表格中扩展列，这是我们制作电子表格供人们阅读时的一个常见需求**，这有助于读者更好地直观地查看列中的内容，而默认的列大小无法做到这一点。

在本教程中，我们将学习如何使用API在Excel电子表格中手动和自动调整列宽。

## 2. 依赖

首先，我们需要在Maven pom.xml中添加以下[Apache POI](https://mvnrepository.com/artifact/org.apache.poi/poi)依赖：

```xml
<dependency> 
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId> 
    <version>5.2.5</version> 
</dependency> 
<dependency> 
    <groupId>org.apache.poi</groupId> 
    <artifactId>poi-ooxml</artifactId> 
    <version>5.2.5</version> 
</dependency>
```

## 3. 电子表格准备

首先，我们来快速回顾一下如何创建Excel电子表格；为了演示目的，我们将准备一个Excel电子表格，并在其中填充一些数据：

```java
Workbook workbook = new XSSFWorkbook();
Sheet sheet = workbook.createSheet("NewSheet");

Row headerRow = sheet.createRow(0);
Cell headerCell1 = headerRow.createCell(0);
headerCell1.setCellValue("Full Name");
Cell headerCell2 = headerRow.createCell(1);
headerCell2.setCellValue("Abbreviation");

Row dataRow = sheet.createRow(1); 
Cell dataCell1 = dataRow.createCell(0);
dataCell1.setCellValue("Java Virtual Machine"); 
Cell dataCell2 = dataRow.createCell(1);
dataCell2.setCellValue("JVM");

// More data rows created here...
```

现在，如果我们在Excel中打开生成的电子表格，我们会看到每一列都有相同的默认宽度：显然，由于列宽有限，第一列中的内容太长而被截断。

![](/assets/images/2025/apache/javaapachepoiexpandcolumns01.png)

## 4. 宽度调整

Apache POI提供了两种调整列宽的方法，我们可以根据自己的需求选择其中一种，现在让我们来探讨一下这两种方法。

### 4.1 固定宽度调整

**我们可以通过在目标Sheet实例上调用setColumnWidth()将特定列扩展为固定宽度**，此方法有两个参数，分别是columnIndex和width。

**手动获取显示所有内容的列宽非常复杂，因为它取决于字体类型和字体大小等多种因素**，根据[API文档](https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Sheet.html#setColumnWidth-int-int-)中setColumnWidth()的定义，width参数的单位是字符宽度的1 / 256。

鉴于Excel中默认字体Calibri的字体大小为11，我们可以使用单元格中的字符数 * 256作为列宽作为粗略近似值：

```java
String cellValue = row.getCell(0).getStringCellValue();
sheet.setColumnWidth(0, cellValue.length() * 256);
```

调整完成后，我们将看到第一列的全部内容：

![](/assets/images/2025/apache/javaapachepoiexpandcolumns02.png)

自己计算列宽有点麻烦，尤其是在处理包含大量数据行的电子表格时；我们必须逐行检查以确定最大字符数，如果列中包含不同的字体和字体大小，则进一步增加了宽度计算的复杂性。

### 4.2 自动宽度调整

幸运的是，**Apache POI提供了一个便捷的方法autoSizeColumn()来自动调整列宽**，这确保了列的内容能够被读者完整地看到。

autoSizeColumn()只需要一个列参数，即从0开始的列索引，我们可以使用以下代码自动调整第一列的列宽：

```java
sheet.autoSizeColumn(0);
```

如果我们将autoSizeColumn()应用于每一列，我们将看到以下内容；现在，所有列的内容对读者来说都是完全可见的，没有任何截断：

![](/assets/images/2025/apache/javaapachepoiexpandcolumns03.png)

## 5. 总结

在本文中，我们探讨了Apache POI中两种调整Excel电子表格列宽的方法：固定宽度调整和自动宽度调整，调整列宽对于提高可读性和创建易于阅读的Excel电子表格至关重要。