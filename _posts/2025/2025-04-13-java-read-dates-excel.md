---
layout: post
title:  使用Java从Excel读取值
category: apache
copyright: apache
excerpt: Apache POI
---

## 1. 概述

对于Microsoft Excel文件来说，从不同的单元格读取值可能会有些棘手；Excel文件是按行和单元格组织的电子表格，其中可以包含字符串、数字、日期、布尔值甚至公式值。[Apache POI](https://www.baeldung.com/java-microsoft-excel)是一个库，**提供了一整套工具来处理不同的Excel文件和值类型**。

在本教程中，我们将重点学习如何处理Excel文件、遍历行和单元格，以及如何使用正确的方式读取每个单元格的值类型。

## 2. Maven依赖

让我们首先将Apache POI依赖添加到pom.xml：

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.3.0</version>
</dependency>
```

可以在Maven Central找到最新版本的[poi-ooxml](https://mvnrepository.com/artifact/org.apache.poi/poi-ooxml)。

## 3. Apache POI概述

层次结构从工作簿开始，它代表整个Excel文件，每个文件可以包含一个或多个工作表，这些工作表是行和单元格的集合。根据Excel文件的版本，**HSSF是表示旧版Excel文件(.xls)的类的前缀，而XSSF则用于表示最新版本(.xlsx)**。因此，我们有：

- XSSFWorkbook和HSSFWorkbook类代表Excel工作簿
- Sheet接口代表Excel工作表
- Row接口表示行
- Cell接口代表单元格

### 3.1 处理Excel文件

首先，我们打开要读取的文件，并将其转换为FileInputStream进行进一步处理。FileInputStream的构造函数会抛出java.io.FileNotFoundException异常，因此我们需要将其封装在try-catch块中，并在最后关闭该流：

```java
public static void readExcel(String filePath) {
    File file = new File(filePath);
    try {
        FileInputStream inputStream = new FileInputStream(file);
        // ...
        inputStream.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

### 3.2 遍历Excel文件

成功打开InputStream后，就可以创建XSSFWorkbook并遍历每个工作表的行和单元格了。如果我们知道工作表的确切数量或特定工作表的名称，我们可以分别使用XSSFWorkbook的getSheetAt(int index)和getSheet(String sheetName)方法。

由于我们想要读取任何类型的Excel文件，我们将**使用3个嵌套的for循环遍历所有工作表，一个用于工作表，一个用于每个工作表的行，最后一个用于每个工作表的单元格**。

为了本教程的目的，我们仅将数据打印到控制台：

```java
FileInputStream inputStream = new FileInputStream(file);
Workbook baeuldungWorkBook = new XSSFWorkbook(inputStream);
for (Sheet sheet : baeuldungWorkBook) {
    // ...
}
```

然后，为了遍历工作表的行，我们需要找到从工作表对象中获取的第一行和最后一行的索引：

```java
int firstRow = sheet.getFirstRowNum();
int lastRow = sheet.getLastRowNum();
for (int index = firstRow + 1; index <= lastRow; index++) {
    Row row = sheet.getRow(index);
}
```

最后，我们对单元格执行相同的操作。此外，在访问每个单元格时，我们可以选择传递一个MissingCellPolicy值，该值基本上告诉POI当单元格值为空或null时应返回什么。MissingCellPolicy枚举包含三个枚举值：

- RETURN_NULL_AND_BLANK
- RETURN_BLANK_AS_NULL
- CREATE_NULL_AS_BLANK

单元格迭代的代码如下：

```java
for (int cellIndex = row.getFirstCellNum(); cellIndex < row.getLastCellNum(); cellIndex++) {
    Cell cell = row.getCell(cellIndex, Row.MissingCellPolicy.CREATE_NULL_AS_BLANK);
    // ...
}
```

### 3.3 读取Excel中的单元格值

正如我们之前提到的，Microsoft Excel的单元格可以包含不同类型的值，因此区分不同单元格值类型并使用适当的方法提取值非常重要。以下是所有值类型的列表：

- NONE
- NUMERIC
- STRING
- FORMULA
- BLANK
- BOOLEAN
- ERROR

我们将重点介绍4种主要的单元格值类型：NUMERIC、STRING、BOOLEAN和FORMULA，其中最后一种包含前三种类型的计算值。

让我们创建一个辅助方法，它基本上会检查每个值的类型，并根据类型使用适当的方法来访问该值。它也可以把[单元格值当作字符串](https://www.baeldung.com/java-apache-poi-cell-string-value)，并使用相应的方法进行检索。

有两点值得注意；首先，日期值存储为数字值，并且如果单元格的值类型为FORMULA，我们需要使用getCachedFormulaResultType()而不是getCellType()方法来[检查公式的计算结果](https://www.baeldung.com/apache-poi-read-cell-value-formula)：

```java
public static void printCellValue(Cell cell) {
    CellType cellType = cell.getCellType().equals(CellType.FORMULA)
            ? cell.getCachedFormulaResultType() : cell.getCellType();
    if (cellType.equals(CellType.STRING)) {
        System.out.print(cell.getStringCellValue() + " | ");
    }
    if (cellType.equals(CellType.NUMERIC)) {
        if (DateUtil.isCellDateFormatted(cell)) {
            System.out.print(cell.getDateCellValue() + " | ");
        } else {
            System.out.print(cell.getNumericCellValue() + " | ");
        }
    }
    if (cellType.equals(CellType.BOOLEAN)) {
        System.out.print(cell.getBooleanCellValue() + " | ");
    }
}
```

现在，我们需要做的就是在单元格循环中调用printCellValue方法即可。以下是完整代码片段：

```java
// ...
for (int cellIndex = row.getFirstCellNum(); cellIndex < row.getLastCellNum(); cellIndex++) {
    Cell cell = row.getCell(cellIndex, Row.MissingCellPolicy.CREATE_NULL_AS_BLANK);
    printCellValue(cell);
}
// ...
```

## 4. 总结

在本文中，我们展示了一个使用Apache POI读取Excel文件和访问不同单元格值的示例项目。