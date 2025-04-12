---
layout: post
title:  使用Apache POI获取Excel单元格的字符串值
category: apache
copyright: apache
excerpt: Apache POI
---

## 1. 概述

[Microsoft Excel](https://www.baeldung.com/java-microsoft-excel)单元格可以具有不同类型，如字符串、数字、布尔值和公式。

在本快速教程中，我们将展示如何使用Apache POI将单元格值读取为字符串(无论单元格类型如何)。

## 2. Apache POI

首先，我们需要将[poi](https://mvnrepository.com/artifact/org.apache.poi/poi)依赖添加到项目pom.xml文件中：

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>5.3.0</version>
</dependency>
```

Apache POI使用[Workbook](https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Workbook.html)接口来表示Excel文件，它还使用[Sheet](https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Sheet.html)、[Row](https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Row.html)和[Cell](https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Cell.html)接口来建模Excel文件中不同级别的元素。在Cell级别，我们可以使用它的[getCellType()](https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Cell.html#getCellType--)方法来获取[单元格类型](https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/CellType.html)。Apache POI支持以下单元格类型：

- BLANK
- BOOLEAN
- ERROR
- FORMULA
- NUMERIC
- STRING

如果我们想在屏幕上显示Excel文件的内容，我们希望获取单元格的字符串表示形式，而不是其原始值。因此，**对于非STRING类型的单元格，我们需要将其数据转换为字符串值**。

## 3. 获取单元格字符串值

**我们可以使用[DataFormatter](https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/DataFormatter.html)来获取Excel单元格的字符串值**，它可以获取单元格中存储值的格式化字符串表示形式。例如，如果某个单元格的数值是1.234，并且该单元格的格式规则是保留两位小数，那么我们将获得字符串表示形式“1.23”：

```java
Cell cell = // a numeric cell with value of 1.234 and format rule "0.00"

DataFormatter formatter = new DataFormatter();
String strValue = formatter.formatCellValue(cell);

assertEquals("1.23", strValue);
```

因此，DataFormatter.formatCellValue()的结果就是与Excel中显示的字符串完全一样的显示字符串。

## 4. 获取公式单元格的字符串值

如果单元格的类型为FORMULA，则上述方法将返回原始公式字符串，而不是计算出的公式值。因此，**为了获取公式值的字符串表示形式，我们需要使用[FormulaEvaluator](https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/FormulaEvaluator.html)来计算公式**：

```java
Workbook workbook = // existing Workbook setup
FormulaEvaluator evaluator = workbook.getCreationHelper().createFormulaEvaluator();

Cell cell = // a formula cell with value of "SUM(1,2)"

DataFormatter formatter = new DataFormatter();
String strValue = formatter.formatCellValue(cell, evaluator);

assertEquals("3", strValue);
```

此方法适用于所有单元格类型，如果单元格类型为FORMULA，我们将使用给定的FormulaEvaluator对其进行求值；否则，我们将返回不进行任何求值的字符串表示形式。