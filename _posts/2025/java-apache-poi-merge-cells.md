---
layout: post
title:  使用Apache POI合并Excel中的单元格
category: apache
copyright: apache
excerpt: Apache POI
---

## 1. 概述

在本教程中，我们将展示如何使用Apache POI合并[Excel](https://www.baeldung.com/java-microsoft-excel)中的单元格。

## 2. Apache POI

首先，我们需要将[poi](https://mvnrepository.com/artifact/org.apache.poi/poi)依赖添加到项目pom.xml文件中：

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>5.3.0</version>
</dependency>
```

Apache POI使用[Workbook](https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Workbook.html)接口来表示Excel文件，它还使用[Sheet](https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Sheet.html)、[Row](https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Row.html)和[Cell](https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Cell.html)接口来模拟Excel文件中不同级别的元素。

## 3. 合并单元格

在Excel中，我们有时希望跨两个或多个单元格显示一个字符串。例如，我们可以水平合并多个单元格，以创建跨越多列的表格标题：

![](/assets/images/2025/apache/javaapachepoimergecells01.png)

**为此，我们可以使用[addMergedRegion](https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Sheet.html#addMergedRegion-org.apache.poi.ss.util.CellRangeAddress-)合并由[CellRangeAddress](https://poi.apache.org/apidocs/dev/org/apache/poi/ss/util/CellRangeAddress.html)定义的多个单元格**。设置单元格范围有两种方法，首先，我们可以使用4个从0开始的索引来定义左上角单元格位置和右下角单元格位置：

```java
sheet = // existing Sheet setup
int firstRow = 0;
int lastRow = 0;
int firstCol = 0;
int lastCol = 2
sheet.addMergedRegion(new CellRangeAddress(firstRow, lastRow, firstCol, lastCol));
```

我们还可以使用单元格范围引用字符串来提供合并区域：

```java
sheet = // existing Sheet setup
sheet.addMergedRegion(CellRangeAddress.valueOf("A1:C1"));
```

如果单元格在合并之前已有数据，Excel将使用左上角单元格的值作为合并区域的值。对于其他单元格，Excel将丢弃其数据。

在Excel文件中添加多个合并区域时，不应创建任何重叠，否则，Apache POI将在运行时抛出异常。

## 4. 对齐合并单元格

合并单元格后，我们通常需要对齐合并单元格内的内容，以确保其看起来井然有序。为了对齐内容，我们可以使用CellStyle类设置各种对齐属性。

以下是创建单元格样式并将对齐方式应用于合并单元格的方法：

```java
CellStyle cellStyle = workbook.createCellStyle();
cellStyle.setAlignment(HorizontalAlignment.CENTER);
cellStyle.setVerticalAlignment(VerticalAlignment.CENTER);

Row row = sheet.getRow(firstRow);
Cell cell = row.getCell(firstCol);
```

在此示例中，我们将合并单元格的水平和垂直对齐方式设置为居中，这确保合并单元格中的文本在水平和垂直方向上均居中：

![](/assets/images/2025/apache/javaapachepoimergecells02.png)

## 5. 总结

在这篇快速文章中，我们了解了如何使用Apache POI合并多个单元格，我们还讨论了两种定义合并区域的方法。