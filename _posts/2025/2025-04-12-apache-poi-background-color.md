---
layout: post
title:  使用Apache POI设置单元格的背景颜色
category: apache
copyright: apache
excerpt: Apache POI
---

## 1. 概述

在Excel工作表上，通过更改表头的背景颜色来突出显示表头总是能带来美观的效果，本文介绍如何使用[Apache POI](https://poi.apache.org/)更改单元格的背景颜色。

## 2. Maven依赖

首先，我们需要在pom.xml中添加poi-ooxml作为依赖：

```xml
<dependency>
     <groupId>org.apache.poi</groupId>
     <artifactId>poi-ooxml</artifactId>
     <version>5.3.0</version>
 </dependency>
```

## 3. 更改单元格背景颜色

### 3.1 关于单元格背景

在Excel工作表上，我们可以通过填充颜色或图案来更改单元格背景。在下图中，单元格A1填充了浅蓝色背景，而单元格B1填充了图案。此图案的背景为黑色，顶部有浅蓝色的斑点：

![](/assets/images/2025/apache/apachepoibackgroundcolor01.png)

### 3.2 更改背景颜色的代码

Apache POI提供了三种更改背景颜色的方法；在CellStyle类中，我们可以使用setFillForegroundColor、setFillPattern和setFillBackgroundColor方法来实现此目的。IndexedColors类中定义了颜色列表；同样，FillPatternType中定义了图案列表。

**有时，setFillBackgroundColor这个名称可能会误导我们**，但是，该方法本身不足以更改单元格背景。要通过填充纯色来更改单元格背景，我们使用setFillForegroundColor和setFillPattern方法；第一个方法指定要填充的颜色，而第二个方法指定要使用的纯色填充图案。

以下代码片段是更改单元格背景的示例方法，如单元格A1所示：

```java
public void changeCellBackgroundColor(Cell cell) {
    CellStyle cellStyle = cell.getCellStyle();
    if(cellStyle == null) {
        cellStyle = cell.getSheet().getWorkbook().createCellStyle();
    }
    cellStyle.setFillForegroundColor(IndexedColors.LIGHT_BLUE.getIndex());
    cellStyle.setFillPattern(FillPatternType.SOLID_FOREGROUND);
    cell.setCellStyle(cellStyle);
}
```

**要使用图案更改单元格背景，我们需要使用两种颜色**：一种颜色填充整个背景，另一种颜色在前一种颜色上填充图案。在这里，我们需要同时使用这三种方法。

这里使用setFillBackgroundColor方法指定背景颜色，仅使用此方法不会产生任何效果，我们需要使用setFillForegroundColor来选择第二种颜色，并使用setFillPattern来指定图案类型。

以下代码片段是更改单元格背景的示例方法，如单元格B1所示：

```java
public void changeCellBackgroundColorWithPattern(Cell cell) {
    CellStyle cellStyle = cell.getCellStyle();
    if(cellStyle == null) {
        cellStyle = cell.getSheet().getWorkbook().createCellStyle();
    }
    cellStyle.setFillBackgroundColor(IndexedColors.BLACK.index);
    cellStyle.setFillPattern(FillPatternType.BIG_SPOTS);
    cellStyle.setFillForegroundColor(IndexedColors.LIGHT_BLUE.getIndex());
    cell.setCellStyle(cellStyle);
}
```

## 4. 总结

在本快速教程中，我们学习了如何使用Apache POI更改Excel表中单元格的单元格背景。

仅使用CellStyle类中的三种方法：setFillForegroundColor、setFillPattern和setFillBackgroundColor，我们可以轻松更改单元格的背景颜色和填充图案。