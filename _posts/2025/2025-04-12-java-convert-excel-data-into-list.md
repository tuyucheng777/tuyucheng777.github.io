---
layout: post
title:  如何将Excel数据转换为Java对象列表
category: apache
copyright: apache
excerpt: Apache POI
---

## 1. 简介

了解数据映射在软件开发中至关重要，Excel是一款广泛使用的数据管理软件，因此对于Java开发人员来说，了解如何在Excel和Java对象之间映射数据至关重要。

在本教程中，我们将探讨如何将Excel数据转换为Java对象列表。

Maven仓库中提供了多个Java库来在Java中处理Excel文件，其中最常见的是Apache POI。但是，在本教程中，**我们将使用4个Java Excel库(包括Apache POI、Poiji、FastExcel和JExcelApi(Jxl))将Excel数据转换为Java对象列表**。

## 2. 模型设置

首先，我们需要创建对象的蓝图，即FoodInfo类：

```java
public class FoodInfo {

    private String category; 
    private String name; 
    private String measure;
    private double calories; 
   
    // standard constructors, toString, getters and setters
}
```

## 3. Apache POI

Apache POI是一套用于Microsoft文档的Java API，**它是一组纯Java库的集合，用于从Word、Outlook、Excel等Microsoft Office文件读取数据或向其中写入数据**。

### 3.1 Maven依赖

让我们将[Maven依赖](https://mvnrepository.com/artifact/org.apache.poi)添加到pom.xml文件中：

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>5.2.5</version>
</dependency>
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.2.5</version>
</dependency>
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml-schemas</artifactId>
    <version>4.1.2</version>
</dependency>
```

### 3.2 将Excel数据转换为对象列表

通过使用Workbook接口，我们可以访问各种功能来读取Excel文件的工作表和单元格。**该接口有两种实现，分别对应每种Excel格式：HSSFWorkbook(用于.xls)和XSSFWorkbook(用于.xlsx)**。

此代码片段使用Apache POI库从.xlsx文件读取Excel数据并将其转换为FoodInfo对象列表：

```java
public static List<FoodInfo> excelDataToListOfObjets_withApachePOI(String fileLocation) throws IOException {
    FileInputStream file = new FileInputStream(new File(fileLocation));
    Workbook workbook = new XSSFWorkbook(file);
    Sheet sheet = workbook.getSheetAt(0);
    List<FoodInfo> foodData = new ArrayList<FoodInfo>();
    DataFormatter dataFormatter = new DataFormatter();
    for (int n = 1; n < sheet.getPhysicalNumberOfRows(); n++) {
        Row row = sheet.getRow(n);
        FoodInfo foodInfo = new FoodInfo();
        int i = row.getFirstCellNum();

        foodInfo.setCategory(dataFormatter.formatCellValue(row.getCell(i)));
        foodInfo.setName(dataFormatter.formatCellValue(row.getCell(++i)));
        foodInfo.setMeasure(dataFormatter.formatCellValue(row.getCell(++i)));
        foodInfo.setCalories(row.getCell(++i).getNumericCellValue());

        foodData.add(foodInfo);
    }
    return foodData;
}
```

**为了确定Sheet对象中非空行的数量，我们使用getPhysicalNumberOfRows()方法**。然后，我们循环遍历所有行，但不包括标题行(i = 1)。

根据我们需要填充的食物对象的字段，**我们使用dataFormatter对象或getNumericValue()方法将单元格值转换并分配给适当的数据类型**。

让我们通过编写单元测试来验证我们的代码，以确保它使用名为food_info.xlsx的Excel文件按预期工作：

```java
@Test
public void whenParsingExcelFileWithApachePOI_thenConvertsToList() throws IOException {
    List<FoodInfo> foodInfoList = ExcelDataToListApachePOI
        .excelDataToListOfObjets_withApachePOI("src\\main\\resources/food_info.xlsx");

    assertEquals("Beverages", foodInfoList.get(0).getCategory());
    assertEquals("Dairy", foodInfoList.get(3).getCategory());
}
```

**Apache POI库为旧版本和新版本的Excel文件提供支持，即.xls和.xlsx**。

## 4. Poiji

**Poiji是一个线程安全的Java库，它提供了从Excel工作表到Java类的单向数据映射API**。它基于Apache POI库构建，但与Apache POI不同的是，Poiji使用起来更加简单，可以直接将Excel的每一行数据转换为Java对象。

### 4.1 设置Maven依赖

这是我们需要添加到pom.xml文件的[Poiji Maven依赖](https://mvnrepository.com/artifact/com.github.ozlerhakan/poiji)：

```xml
<dependency>
    <groupId>com.github.ozlerhakan</groupId>
    <artifactId>poiji</artifactId>
    <version>4.1.1</version>
</dependency>
```

### 4.2 使用注解设置类

**Poiji库通过要求使用@ExcelCellName(String cellName)或@ExcelCell(int cellIndex)标注类字段来简化Excel数据检索**。

下面，我们通过添加注解为Poiji库设置FoodInfo类：

```java
public class FoodInfo {

    @ExcelCellName("Category")
    private String category;

    @ExcelCellName("Name")
    private String name;

    @ExcelCellName("Measure")
    private String measure;

    @ExcelCellName("Calories")
    private double calories;

    // standard constructors, getters and setters 
}
```

该API支持映射包含多个工作表的Excel工作簿，当我们的文件包含多个工作表时，**我们可以在类上使用@ExcelSheet(String sheetName)注解来指示要使用哪个工作表，其他工作表将被忽略**。

但是，如果我们不使用此注解，则只会考虑工作簿中的第一个Excel表。

在某些情况下，我们可能不需要从目标Excel工作表的每一行中提取数据。为了解决这个问题，**我们可以在类中添加一个带有@ExcelRow注解的私有int rowIndex属性，这将允许我们指定要访问的行项的索引**。

### 4.3 将Excel数据转换为对象列表

**与本文中提到的库不同，Poiji库默认忽略Excel工作表的标题行**。

以下代码片段从Excel文件中提取数据并将其数据转换为FoodInfo列表：

```java
public class ExcelDataToListOfObjectsPOIJI {
    public static List<FoodInfo> excelDataToListOfObjets_withPOIJI(String fileLocation){
        return Poiji.fromExcel(new File(fileLocation), FoodInfo.class);
    }
}
```

该程序将fileLocation文件的第一个Excel表转换为FoodInfo类，**每一行都成为FoodInfo类的一个实例，单元格值代表对象的属性。输出是一个FoodInfo对象列表，其大小等于原始Excel表中的行数(不包括标题行)**。

在某些情况下，密码可能会保护我们的Excel工作表，我们可以通过PoijiOptionsBuilder定义密码：

```java
PoijiOptions options = PoijiOptionsBuilder.settings()
    .password("<excel_sheet_password>").build();
List<FoodInfo> foodData = Poiji.fromExcel(new File(fileLocation), FoodInfo.class, options);
```

为了确保我们的代码按预期工作，我们编写了一个单元测试：

```java
@Test
public void whenParsingExcelFileWithPOIJI_thenConvertsToList() throws IOException {
    List<FoodInfo> foodInfoList = ExcelDataToListOfObjectsPOIJI
            .excelDataToListOfObjets_withPOIJI("src\\main\\resources/food_info.xlsx");

    assertEquals("Beverages", foodInfoList.get(0).getCategory());
    assertEquals("Dairy", foodInfoList.get(3).getCategory());
}
```

## 5. FastExcel

**FastExcel是一个高效的库，它占用极少的内存，并提供高性能，可用于在Java中创建和读取基本的Excel工作簿**。它仅支持较新版本的Excel文件(.xlsx)，并且与Apache POI相比功能有限。

它仅读取单元格内容，不包括图、样式或其他单元格格式。

### 5.1 设置Maven依赖

以下是添加到pom.xml的[FastExcel](https://mvnrepository.com/artifact/org.dhatim/fastexcel)和[FastExcel Reader](https://mvnrepository.com/artifact/org.dhatim/fastexcel-reader) Maven依赖：

```xml
<dependency>
      <groupId>org.dhatim</groupId>
      <artifactId>fastexcel</artifactId>
      <version>0.15.7</version>
</dependency>
<dependency>
      <groupId>org.dhatim</groupId>
      <artifactId>fastexcel-reader</artifactId>
      <version>0.15.7</version>
</dependency>
```

### 5.2 将Excel数据转换为对象列表

**处理大文件时，尽管功能有限，FastExcel读取器仍然是一个不错的选择。它易于使用，我们可以使用ReadableWorkbook类访问整个Excel工作簿**。

这使我们能够按名称或索引单独检索工作表。

在下面的方法中，我们从Excel表中读取数据并将其转换为FoodInfo对象列表：

```java
public static List<FoodInfo> excelDataToListOfObjets_withFastExcel(String fileLocation) throws IOException, NumberFormatException {
    List<FoodInfo> foodData = new ArrayList<FoodInfo>();

    try (FileInputStream file = new FileInputStream(fileLocation);
         ReadableWorkbook wb = new ReadableWorkbook(file)) {
        Sheet sheet = wb.getFirstSheet();
        for (Row row: sheet.read()) {
            if (row.getRowNum() == 1) {
                continue;
            }
            FoodInfo food = new FoodInfo();
            food.setCategory(row.getCellText(0));
            food.setName(row.getCellText(1));
            food.setMeasure(row.getCellText(2));
            food.setCalories(Double.parseDouble(row.getCellText(3)));

            foodData.add(food);
        }
    }
    return foodData;
}
```

因为API读取了工作表中的所有行(包括标题行)，所以我们需要在循环遍历行时跳过第一行(非0基索引)。

访问单元格可以通过实例化Cell类来实现：Cell cell= row.getCell()，它有两种实现，一种接收int cellIndex参数，另一种接收String cellAddress参数(例如“C4”)；或者直接获取单元格中的数据：例如row.getCellText()。

无论哪种方式，在提取每个单元格的内容后，我们需要确保在必要时将其转换为适当的food对象字段类型。

让我们编写一个单元测试来确保转换有效：

```java
@Test
public void whenParsingExcelFileWithFastExcel_thenConvertsToList() throws IOException {
    List<FoodInfo> foodInfoList = ExcelDataToListOfObjectsFastExcel
        .excelDataToListOfObjets_withFastExcel("src\\main\\resources/food_info.xlsx");

    assertEquals("Beverages", foodInfoList.get(0).getCategory());
    assertEquals("Dairy", foodInfoList.get(3).getCategory());
}
```

## 6. JExcelApi(Jxl)

**JExcelApi(或Jxl)是一个用于读取、写入和修改Excel电子表格的轻量级Java库**。

### 6.1 设置Maven依赖

让我们将[JExcelApi](https://mvnrepository.com/artifact/net.sourceforge.jexcelapi/jxl)的Maven依赖添加到pom.xml文件中：

```xml
<dependency>
    <groupId>net.sourceforge.jexcelapi</groupId>
    <artifactId>jxl</artifactId>
    <version>2.6.12</version>
</dependency>
```

### 6.2 将Excel数据转换为对象列表

虽然**JExcel库仅支持较旧的Excel格式(.xls)文件**，但它提供了一系列用于操作Excel文件的类，Workbook类用于访问文件中的Excel工作表列表。

下面的代码使用该库将.xls文件中的数据转换为FoodInfo对象列表foodData：

```java
public static List<FoodInfo> excelDataToListOfObjets_withJxl(String fileLocation) throws IOException, BiffException {
    List<FoodInfo> foodData = new ArrayList<FoodInfo>();

    Workbook workbook = Workbook.getWorkbook(new File(fileLocation));
    Sheet sheet = workbook.getSheet(0);

    int rows = sheet.getRows();

    for (int i = 1; i < rows; i++) {
        FoodInfo foodInfo = new FoodInfo();

        foodInfo.setCategory(sheet.getCell(0, i).getContents());
        foodInfo.setName(sheet.getCell(1, i).getContents());
        foodInfo.setMeasure(sheet.getCell(2, i).getContents());
        foodInfo.setCalories(Double.parseDouble(sheet.getCell(3, i).getContents()));

        foodData.add(foodInfo);
    }
    return foodData;
}
```

**由于库不会忽略标题行，因此我们必须从i = 1开始循环**，Sheet对象是从0开始的行索引列表。

**使用JExcel库检索单元格数据与FastExcel库非常相似，这两个库都使用getCell()方法，但都有两种实现**。

然而，在JExcel中，此方法是直接从Sheet对象而不是Row对象访问的。此外，JExcel中 getCell()方法的实现之一接收两个参数，colNum和rowNum，它们都是整数：sheet.getCell(colNum, rowNum)。

为了确保转换正常工作，让我们为我们的方法编写一个单元测试：

```java
@Test
public void whenParsingExcelFileWithJxl_thenConvertsToList() throws IOException, BiffException {
    List<FoodInfo> foodInfoList = ExcelDataToListOfObjectsJxl
        .excelDataToListOfObjets_withJxl("src\\main\\resources/food_info.xls");

    assertEquals("Beverages", foodInfoList.get(0).getCategory());
    assertEquals("Dairy", foodInfoList.get(3).getCategory());
}
```

## 7. 总结

在本文中，我们探讨了如何使用多个库(例如Apache POI、Poiji、FastExcel和JExcelApi)读取Excel文件中的数据并将其转换为Java对象，选择使用哪个库取决于具体需求，并考虑每个库的优缺点。

例如，如果我们优先考虑最简单的方法，即从Excel文件中读取数据并直接将其转换为Java对象列表，我们可能会选择使用Poiji库。

就Java中双向Excel数据映射的性能和简便性而言，FastExcel和JExcelApi库是不错的选择。然而，与Apache POI相比，它们提供的功能较少，而Apache POI是一个功能丰富的库，支持样式和图形。