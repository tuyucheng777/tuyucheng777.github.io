---
layout: post
title:  使用Apache POI为Excel单元格添加边框
category: apache
copyright: apache
excerpt: Apache POI
---

## 1. 概述

在本教程中，我们将学习如何使用[Apache POI](https://poi.apache.org/) Java库向Excel表添加边框。

要了解有关Excel处理的更多基础知识，我们可以从[Java使用Microsoft Excel](https://www.baeldung.com/java-microsoft-excel)开始。

## 2. Excel边框

我们可以为Excel单元格或单元格区域创建边框，**这些边框可以采用多种样式**。例如，粗线、细线、中等粗线和虚线。为了增加更多变化，**我们可以添加彩色边框**。

下图展示的是其中一些种类的边框：

![](/assets/images/2025/apache/apachepoiaddborders01.png)

- 单元格B2带有粗线边框
- 单元格D2带有紫色边框
- F2单元带有虚线边框，边框的每一侧都有不同的样式和颜色
- 范围B4:F6具有中等大小的边框
- 区域B8:F9带有中等大小的橙色边框

## 3. Excel边框的编码

Apache POI库提供了多种处理边框的方法，一种简单的方法是引用单元格范围并应用边框。

### 3.1 单元格范围或区域

要引用单元格范围，我们可以使用CellRangeAddress类：

```java
CellRangeAddress region = new CellRangeAddress(7, 8, 1, 5);
```

CellRangeAddress构造函数接收4个参数：第一行、最后一行、第一列和最后一列，每行和每列的索引都从0开始。在上面的代码中，它指的是单元格区域B8:F9。

我们还可以使用CellRangeAddress类引用一个单元格：

```java
CellRangeAddress region = new CellRangeAddress(1, 1, 5, 5);
```

上述代码指的是F2单元。

### 3.2 单元格边框

每个边框有4条边：上边框、下边框、左边框和右边框，**我们必须分别设置每条边框的样式**，BorderStyle类提供了多种样式。

我们可以使用RangeUtil类设置边框：

```java
RegionUtil.setBorderTop(BorderStyle.DASH_DOT, region, sheet);
RegionUtil.setBorderBottom(BorderStyle.DOUBLE, region, sheet);
RegionUtil.setBorderLeft(BorderStyle.DOTTED, region, sheet);
RegionUtil.setBorderRight(BorderStyle.SLANTED_DASH_DOT, region, sheet);
```

### 3.3 边框颜色

边框颜色也需要分别设置，IndexedColors类提供了一系列可用的颜色。

我们可以使用RangeUtil类设置边框颜色：

```java
RegionUtil.setTopBorderColor(IndexedColors.RED.index, region, sheet);
RegionUtil.setBottomBorderColor(IndexedColors.GREEN.index, region, sheet);
RegionUtil.setLeftBorderColor(IndexedColors.BLUE.index, region, sheet);
RegionUtil.setRightBorderColor(IndexedColors.VIOLET.index, region, sheet);
```

## 4. 总结

在这篇短文中，我们了解了如何使用CellRangeAddress、RegionUtil、BorderStyles和IndexedColors类生成各种单元格边框，边框的每一侧都必须单独设置。