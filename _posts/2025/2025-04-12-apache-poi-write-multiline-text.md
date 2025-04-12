---
layout: post
title:  使用Apache POI在Excel单元格中输入多行文本
category: apache
copyright: apache
excerpt: Apache POI
---

## 1. 简介

我们可以使用Apache POI在Microsoft Excel电子表格中以编程方式创建多行文本，但是，它不会显示为多行，这是因为使用代码向单元格添加文本不会自动调整单元格高度并应用所需的格式将其转换为多行文本。

这个简短的教程将演示正确显示此类文本所需的代码。

## 2. Apache POI和Maven依赖

Apache POI是一个开源库，允许软件开发人员创建和操作Microsoft Office文档。作为先决条件，读者可以参考我们关于[Java使用Microsoft Excel](https://www.baeldung.com/java-microsoft-excel)的文章，以及如何[使用Apache POI在Excel中插入行](https://www.baeldung.com/apache-poi-insert-excel-row)的教程。

首先，我们需要将[Apache POI](https://mvnrepository.com/artifact/org.apache.poi/poi)依赖添加到项目pom.xml文件中：

```xml
<dependency>
    <groupId>org.apache.poi</groupId> 
    <artifactId>poi</artifactId> 
    <version>5.3.0</version> 
</dependency>
```

## 3. 添加和格式化多行文本

让我们从一个包含多行文本的单元格开始：

```java
cell.setCellValue("Hello \n world!");
```

如果我们仅使用上述代码生成并保存一个Excel文件，它将如下图所示：

![](/assets/images/2025/apache/apachepoiwritemultilinetext01.png)

我们可以点击上图中的1和2来验证该文本确实是多行文本。

使用代码，格式化单元格并将其行高扩展为等于或大于两行文本的任意值：

```java
cell.getRow()
    .setHeightInPoints(cell.getSheet().getDefaultRowHeightInPoints() * 2);
```

之后，我们需要设置单元格样式来换行：

```java
CellStyle cellStyle = cell.getSheet().getWorkbook().createCellStyle();
cellStyle.setWrapText(true);
cell.setCellStyle(cellStyle);
```

保存使用上述代码生成的文件并在Microsoft Excel中查看它将在单元格中显示多行文本。

![](/assets/images/2025/apache/apachepoiwritemultilinetext02.png)

## 4. 总结

在本教程中，我们学习了如何使用Apache POI向单元格添加多行文本。然后，我们通过对单元格应用一些格式来确保它显示为两行文本；否则，单元格将显示为一行。