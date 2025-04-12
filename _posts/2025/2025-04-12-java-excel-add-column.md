---
layout: post
title:  使用Apache POI向Excel工作表添加列
category: apache
copyright: apache
excerpt: Apache
---

## 1. 概述

在本教程中，我们将展示如何使用Apache POI向[Excel](https://www.baeldung.com/java-microsoft-excel)文件的工作表添加列。

## 2. Apache POI

首先，我们需要将[poi-ooxml](https://mvnrepository.com/artifact/org.apache.poi/poi)依赖添加到我们项目的pom.xml文件中：

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.2.5</version>
</dependency>
```

Apache POI使用[Workbook](https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Workbook.html)接口来表示Excel文件，它还使用[Sheet](https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Sheet.html)、[Row](https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Row.html)和[Cell](https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Cell.html)接口来模拟Excel文件中的不同元素。

## 3. 添加新列

在Excel中，我们有时想在现有行上添加新列。**为此，我们可以遍历每一行并在行尾创建一个新单元格**：

```java
void addColumn(Sheet sheet, CellType cellType) {
    for (Row currentRow : sheet) {
        currentRow.createCell(currentRow.getLastCellNum(), cellType);
    }
}
```

在此方法中，我们使用循环遍历输入Excel表的所有行，对于每一行，我们首先找到其最后一个单元格编号，然后在最后一个单元格后创建一个新单元格。

## 4. 总结

在这篇简短的文章中，我们展示了如何使用Apache POI添加新列。