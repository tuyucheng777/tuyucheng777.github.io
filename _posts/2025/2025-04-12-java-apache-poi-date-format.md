---
layout: post
title:  使用Apache POI设置日期格式
category: apache
copyright: apache
excerpt: Apache POI
---

## 1. 简介

当我们在Apache POI中处理日期时，我们希望确保它们的格式正确。

幸运的是，使用Apache POI设置日期格式很容易。在本教程中，我们将展示如何使用[Apache POI](https://www.baeldung.com/java-microsoft-excel)将日期的自定义DataFormat定义为CellStyle，以及如何使用现有的DataFormats。

## 2. 起点

我们的起点将是一个新的XSSFWorkbook、一个XSSFCell和一个已经创建的CellStyle：

```java
XSSFWorkbook wb = new XSSFWorkbook();
CellStyle cellStyle = wb.createCellStyle();
wb.createSheet();
XSSFSheet sheet = wb.getSheetAt(0);
XSSFCell dateCell = sheet.createRow(0).createCell(0);
dateCell.setCellValue(new Date());
```

由于我们尚未设置所需的DataFormat，因此我们的Date将被转换为数字并显示为：

>44898,9262857176

这种表示方式对于人类来说可读性很差，因此，接下来我们将探讨如何通过格式化来创建更好的可视化效果。

## 3. 创建自定义数据格式

首先，我们需要创建一个新的CreationHelper；通过CreationHelper，我们可以创建一个具有特定Format的新DataFormat。此DataFormat存储在内部，并由一个short引用。我们需要将它添加到CellStyle本身，并将CellStyle应用于Cell：

```java
CreationHelper createHelper = wb.getCreationHelper();
short format = createHelper.createDataFormat().getFormat("m.d.yy h:mm");
cellStyle.setDataFormat(format);
dateCell.setCellStyle(cellStyle);
```

设置此自定义CellStyle后，我们的日期将被格式化：

```text
02.12.2022 21:30
```

但是，如果我们创建新的自定义DataFormat，则应始终记住Excel工作簿最多支持65000种单元格样式。因此，**我们应该始终重用现有的单元格样式**，并尽可能将其应用于多个单元格。

## 4. 使用默认数据格式

正如我们所了解的，**Apache POI使用短代码链接到不同的DataFormat**。Excel已经有很多内置DataFormat，我们可以通过它们的短代码直接调用它们：

```java
cellStyle.setDataFormat((short) 14);
dateCell.setCellStyle(cellStyle);
```

之后，我们可以使用以下代码行以字符串表示形式获取DataFormat：

```java
cellStyle.getDataFormatString();
```

在我们的示例中，我们将获得以下内容：

```text
m/d/yy
```

最常见的数据格式是：

| 短值|      格式       |
| :--: |:-------------:|
|14|    m/d/yy     |
|15|   d-mmm-yy    |
|16|     d-mmm     |
|17|    mmm-yy     |
|18|  h:mm AM/PM   |
|19| h:mm:ss AM/PM |
|20|     h:mm      |
|21|    h:mm:ss    |
|22|   m/d/yy h:mm   |

如果其中一种数据格式符合我们的需求，我们应该始终使用它，因为Excel会将其显示为其格式之一，而不是自定义格式；**这也会触发Excel使用该格式的本地化可视化效果**。

## 5. 总结

可以看出，使用Apache POI设置日期格式既快捷又简单，而且对于以人性化的方式可视化日期也至关重要。