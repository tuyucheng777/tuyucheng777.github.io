---
layout: post
title:  使用Apache POI从Excel中提取列名
category: apache
copyright: apache
excerpt: Apache POI
---

## 1. 简介

无论是读取数据进行处理还是生成报告，高效处理Excel文件都至关重要。[Apache POI](https://www.baeldung.com/java-microsoft-excel)是一个功能强大的Java库，允许开发人员以编程方式操作Excel文件并与之交互。

在本教程中，我们将探索使用Apache POI从Excel表中读取列名。

我们将首先简要概述POI API，然后，我们将设置所需的依赖并介绍一个简单的数据示例。之后，我们将了解如何从新旧格式的Excel文件中的Excel工作表中提取列名。最后，我们将编写单元测试来验证所有操作是否按预期运行。

## 2. 依赖和示例设置

让我们首先在pom.xml中添加所需的依赖，包括[poi-ooxml](https://mvnrepository.com/artifact/org.apache.poi/poi-ooxml)和[commons-collections4](https://mvnrepository.com/artifact/org.apache.commons/commons-collections4)：

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>4.1.2</version>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.4</version>
</dependency>
```

让我们从存储在两个Excel文件中的一些示例数据开始，第一个是food_info.xlsx文件，包含以下列和示例数据：

![](/assets/images/2025/apache/apachepoiextractcolumnnamesexcel01.png)

下一个由consumer_info.xls文件中的消费者数据组成，该文件具有较旧的.xls扩展名，列名由以下内容组成：

![](/assets/images/2025/apache/apachepoiextractcolumnnamesexcel02.png)

接下来我们将按照步骤使用POI提供的API从这些文件中提取列名。

## 3. 从Excel中提取列名

要使用[Apache POI](https://www.baeldung.com/java-microsoft-word-with-apache-poi)从Excel表中读取列名，我们将创建一个执行以下步骤的方法：

- 打开Excel文件
- 访问所需工作表
- 读取标题行(第一行)以获取列名

### 3.1 打开Excel文件

首先，**我们需要打开Excel文件并创建一个WorkBook实例**，POI使用两种不同的抽象(即[XSSFWorkbook](https://www.baeldung.com/java-read-dates-excel)和HSSFWorkbook)支持.xls和.xlsx文件：

```java
public static Workbook openWorkbook(String filePath) throws IOException {
    try (InputStream fileInputStream = new FileInputStream(filePath)) {
        if (filePath.toLowerCase()
                .endsWith("xlsx")) {
            return new XSSFWorkbook(fileInputStream);
        } else if (filePath.toLowerCase()
                .endsWith("xls")) {
            return new HSSFWorkbook(fileInputStream);
        } else {
            throw new IllegalArgumentException("The specified file is not an Excel file");
        }
    } catch (OLE2NotOfficeXmlFileException | NotOLE2FileException e) {
        throw new IllegalArgumentException(
                "The file format is not supported. Ensure the file is a valid Excel file.", e);
    }
}
```

本质上，我们使用代表Excel工作簿的WorkBook接口，它是Apache POI中处理Excel文件的顶级对象，**XSSFWorkbook是一个实现.xlsx文件WorkBook接口的类。另一方面，HSSFWorkbook类实现了.xls文件WorkBook接口**。

### 3.2 访问工作表

现在我们有了一个Workbook，让我们通过工作表名称访问工作簿中的所需工作表：

```java
public static Sheet getSheet(Workbook workbook, String sheetName) {
    return workbook.getSheet(sheetName);
}
```

**POI API中的Sheet接口表示Excel工作簿中的工作表**。

### 3.3 读取标题行

**使用Sheet对象，我们可以根据需要访问其数据**。

让我们使用API读取包含工作表所有列名称的标题行，Row接口表示工作表中的一行。简而言之，我们将访问工作表的第一行，并将索引0传递给sheet.get()方法。然后，我们将使用Cell接口提取标题行中的每一列名称。

**Cell接口表示一行中的一个单元格**：

```java
public static List<String> getColumnNames(Sheet sheet) {
    Row headerRow = sheet.getRow(0);
    if (headerRow == null) {
        return Collections.EMPTY_LIST;
    }
    return StreamSupport.stream(headerRow.spliterator(), false)
            .filter(cell -> cell.getCellType() != CellType.BLANK)
            .map(Cell::getStringCellValue)
            .filter(cellValue -> cellValue != null && !cellValue.trim()
                    .isEmpty())
            .map(String::trim)
            .collect(Collectors.toList());
}
```

这里，我们使用Java [Stream](https://www.baeldung.com/java-streams)迭代每个Cell，我们过滤掉空白单元格以及仅包含空格或空值的单元格。然后，我们使用Cell的getStringCellValue()方法提取剩余每个单元格的字符串值。在本例中，API返回单元格中数据的字符串值。此外，我们还从这些字符串值中去除了空格。最后，我们将这些清理后的字符串值收集到一个列表中并返回。

此时，值得一提的是一个名为getRichStringTextValue()的相关方法，该方法将单元格值检索为RichTextString。这在处理格式化文本(例如，同一单元格中具有不同字体、颜色或样式的文本)时非常有用。**如果我们的用例不仅需要提取列名，还需要保留这些列名的格式，那么我们将使用Cell::getRichStringTextValue()进行映射，并将结果存储为List<RichTextString\>**。

## 4. 单元测试

现在让我们设置单元测试来查看POI API对.xls和.xlsx文件的运行情况：

```java
@Test
public void givenExcelFileWithXLSXFormat_whenGetColumnNames_thenReturnsColumnNames() throws IOException {
    Workbook workbook = ExcelUtils.openWorkbook(XLSX_TEST_FILE_PATH);
    Sheet sheet = ExcelUtils.getSheet(workbook, SHEET_NAME);
    List<String> columnNames = ExcelUtils.getColumnNames(sheet);

    assertEquals(4, columnNames.size());
    assertTrue(columnNames.contains("Category"));
    assertTrue(columnNames.contains("Name"));
    assertTrue(columnNames.contains("Measure"));
    assertTrue(columnNames.contains("Calories"));
    workbook.close();
}

@Test
public void givenExcelFileWithXLSFormat_whenGetColumnNames_thenReturnsColumnNames() throws IOException {
    Workbook workbook = ExcelUtils.openWorkbook(XLS_TEST_FILE_PATH);
    Sheet sheet = ExcelUtils.getSheet(workbook, SHEET_NAME);
    List<String> columnNames = ExcelUtils.getColumnNames(sheet);

    assertEquals(3, columnNames.size());
    assertTrue(columnNames.contains("Name"));
    assertTrue(columnNames.contains("Age"));
    assertTrue(columnNames.contains("City"));

    workbook.close();
}
```

**测试验证了API是否支持从两种类型的Excel文件中读取列名**。

## 5. 总结

在本文中，我们探讨了如何使用Apache POI从Excel表格中读取列名。我们首先概述了Apache POI，然后设置了必要的依赖。之后，我们提供了包含代码片段的分步指南，用于实现该解决方案，并包含单元测试以确保正确性。

Apache POI是一个强大的库，它简化了使用Java处理Excel文件的过程，使其成为处理应用程序和Excel之间数据交换的开发人员的宝贵工具。