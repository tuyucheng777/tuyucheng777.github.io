---
layout: post
title:  使用POI的数字格式
category: apache
copyright: apache
excerpt: Apache POI
---

## 1. 概述

在本快速教程中，**我们将演示如何使用Apache POI设置Excel中的数字单元格的格式**。

## 2. Apache POI

[Apache POI](https://poi.apache.org/)是一个开源的纯Java项目，它提供了用于读写Microsoft Office格式文件(例如Word、PowerPoint和[Excel](https://www.baeldung.com/java-microsoft-excel))的库。

在使用较新的.xlsx文件格式时，我们将使用XSSFWorkbook类，而对于.xls格式，我们将使用HSSFWorkbook类。

## 3. 数字格式

Apache POI的setCellValue方法仅接收double类型输入或可隐式转换为double类型的输入，并返回double类型数值；setCellStyle方法用于添加所需的样式。Excel数字格式中的#字符表示如果需要，可在此处放置一个数字；而字符0则表示始终在此处放置一个数字，即使它是一个不必要的0。

### 3.1 仅显示值的格式

让我们使用0.00或#.##之类的模式来格式化double值，首先，我们将创建一个简单的实用方法来格式化单元格值：

```java
public static void applyNumericFormat(Workbook outWorkbook, Row row, Cell cell, Double value, String styleFormat) {
    CellStyle style = outWorkbook.createCellStyle();
    DataFormat format = outWorkbook.createDataFormat();
    style.setDataFormat(format.getFormat(styleFormat));
    cell.setCellValue(value);
    cell.setCellStyle(style);
}
```

让我们验证一个简单的代码来验证所提到的方法：

```java
File file = new File("number_test.xlsx");
try (Workbook outWorkbook = new XSSFWorkbook()) {
    Sheet sheet = outWorkbook.createSheet("Numeric Sheet");
    Row row = sheet.createRow(0);
    Cell cell = row.createCell(0);
    ExcelNumericFormat.applyNumericFormat(outWorkbook, row, cell, 10.251, "0.00");
    FileOutputStream fileOut = new FileOutputStream(file);
    outWorkbook.write(fileOut);
    fileOut.close();
}
```

这将在电子表格中添加数字单元格：

![](/assets/images/2025/apache/apachepoinumericformat01.png)

注意：**显示值是格式化的值，但实际值保持不变**。如果我们尝试访问同一个单元格，我们仍然会得到10.251。

让我们验证实际值：

```java
try (Workbook inWorkbook = new XSSFWorkbook("number_test.xlsx")) {
    Sheet sheet = inWorkbook.cloneSheet(0);
    Row row = sheet.getRow(0);
    Assertions.assertEquals(10.251, row.getCell(0).getNumericCellValue());
}
```

### 3.2 实际值和显示值的格式

让我们使用模式来格式化显示和实际值：

```java
File file = new File("number_test.xlsx");
try (Workbook outWorkbook = new HSSFWorkbook()) {
    Sheet sheet = outWorkbook.createSheet("Numeric Sheet");
    Row row = sheet.createRow(0);
    Cell cell = row.createCell(0);
    DecimalFormat df = new DecimalFormat("#,###.##");
    ExcelNumericFormat.applyNumericFormat(outWorkbook, row, cell, Double.valueOf(df.format(10.251)), "#,###.##");
    FileOutputStream fileOut = new FileOutputStream(file);
    outWorkbook.write(fileOut);
    fileOut.close();
}
```

这将在电子表格中添加数字单元格并显示格式化的值，同时还会改变实际值：

![](/assets/images/2025/apache/apachepoinumericformat02.png)

让我们断言上述案例中的实际值：

```java
try (Workbook inWorkbook = new XSSFWorkbook("number_test.xlsx")) {
    Sheet sheet = inWorkbook.cloneSheet(0);
    Row row = sheet.getRow(0);
    Assertions.assertEquals(10.25, row.getCell(0).getNumericCellValue());
}
```

## 4. 总结

在本文中，我们演示了如何在Excel工作表中显示格式化的值(无论是否更改数字单元格的实际值)。