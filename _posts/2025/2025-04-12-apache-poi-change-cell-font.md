---
layout: post
title:  使用Apache POI更改单元格字体样式
category: apache
copyright: apache
excerpt: Apache POI
---

## 1. 简介

[Apache POI](https://www.baeldung.com/java-microsoft-excel)是一个开源库，供软件开发人员创建和操作[Microsoft Office文档](https://www.baeldung.com/apache-poi-insert-excel-row)。除其他功能外，它还允许开发人员以编程方式更改文档格式。

在本文中，我们将讨论如何使用名为CellStyle的类来更改Microsoft Excel中单元格的样式。也就是说，使用此类，我们可以编写代码来修改Microsoft Excel文档中单元格的样式。首先，它是Apache POI库提供的一项功能，允许在工作簿中创建具有多种格式属性的样式。其次，该样式可以应用于该工作簿中的多个单元格。

除此之外，我们还将研究使用CellStyle类的常见陷阱。

## 2. Apache POI和Maven依赖

让我们将[Apache POI](https://mvnrepository.com/artifact/org.apache.poi/poi)作为依赖添加到项目pom.xml文件中：

```xml
<dependency>
    <groupId>org.apache.poi</groupId> 
    <artifactId>poi</artifactId> 
    <version>5.3.0</version> 
</dependency>
```

## 3. 创建CellStyle

让我们从实例化CellStyle开始：

```java
Workbook workbook = new XSSFWorkbook(fileLocation);
CellStyle cellStyle = wb.createCellStyle();
```

接下来，我们将设置所需的格式属性，例如，下面的代码将设置其为日期格式：

```java
cellStyle.setDataFormat(createHelper.createDataFormat().getFormat("m/d/yy h:mm"));
```

最重要的是，我们可以设置CellStyle的多个格式属性，以获得所需的样式组合。例如，我们将下面的代码应用于同一个CellStyle对象。因此，它不仅具有日期格式的样式，还具有居中对齐的文本样式：

```java
cellStyle.setAlignment(HorizontalAlignment.CENTER);
```

请注意，**CellStyle有几个我们可以修改的格式属性**：

|         属性          |                      描述|
|:-------------------:| :--------------------------------------------: |
|     DataFormat      |           单元格的数据格式，例如日期|
|      Alignment      |              单元格的水平对齐类型|
|         Hidden          |                 单元格是否隐藏|
|         Indention          |            单元格中文本缩进的空格数|
| BorderBottom、BorderLeft、BorderRight、BorderTop | 单元格底部、左侧、右侧和顶部边框使用的边框类型|
|         Font          |         此样式的字体属性，例如字体颜色|

稍后当我们使用它来更改字体样式时，我们将再次查看Font属性。

## 4. 使用CellStyle格式化字体

CellStyle的Font属性用于设置与字体相关的格式，例如，我们可以设置字体名称、颜色和大小；我们可以设置字体是粗体还是斜体，Font的两个属性都可以为true或false，我们还可以将下划线样式设置为：

|          值          |          解释           |
|:-------------------:|:---------------------:|
|       U_NONE        |       不带下划线的文本        |
|      U_SINGLE       |  单下划线文本，其中只有单词带有下划线   |
|        U_SINGLE_ACCOUNTING         | 单下划线文本，几乎整个单元格宽度都有下划线 |
|      U_DOUBLE       |  双下划线文本，其中只有单词带有下划线   |
| U_DOUBLE_ACCOUNTING | 双下划线文本，几乎整个单元格宽度都有下划线 |

我们继续前面的例子。我们将编写一个名为CellStyler的类，并添加一个创建警告文本样式的方法：

```java
public class CellStyler {
    public CellStyle createWarningColor(Workbook workbook) {
        CellStyle style = workbook.createCellStyle();
        Font font = workbook.createFont();
        font.setFontName("Courier New");
        font.setBold(true);
        font.setUnderline(Font.U_SINGLE);
        font.setColor(HSSFColorPredefined.DARK_RED.getIndex());
        style.setFont(font);

        style.setAlignment(HorizontalAlignment.CENTER);
        style.setVerticalAlignment(VerticalAlignment.CENTER);
        return style;
    }
}
```

现在，让我们创建一个Apache POI工作簿并获取第一个工作表：

```java
Workbook workbook = new XSSFWorkbook(fileLocation);
Sheet sheet = workbook.getSheetAt(0);
```

请注意，**我们设置行高是为了看到文本对齐的效果**：

```java
Row row1 = sheet.createRow(0);
row1.setHeightInPoints((short) 40);
```

让我们实例化该类并使用它来设置样式：

```java
CellStyler styler = new CellStyler();
CellStyle style = styler.createWarningColor(workbook);

Cell cell1 = row1.createCell(0);
cell1.setCellStyle(style);
cell1.setCellValue("Hello");

Cell cell2 = row1.createCell(1);
cell2.setCellStyle(style);
cell2.setCellValue("world!");
```

现在，让我们将此工作簿保存到一个文件中，并在Microsoft Excel中打开该文件以查看字体样式效果，我们应该看到：

![](/assets/images/2025/apache/apachepoichangecellfont01.png)

## 5. 常见陷阱

让我们看一下使用CellStyle时常犯的两个错误。

### 5.1 意外修改所有单元格样式

首先，从单元格获取CellStyle并开始修改它是一个常见的错误。Apache POI文档中关于getCellStyle方法的说明提到，**单元格的getCellStyle方法始终返回非空值**，这意味着该单元格有一个默认值，这也是工作簿中所有单元格最初使用的默认样式。因此，以下代码将使所有单元格都具有日期格式：

```java
cell.setCellValue(rdf.getEffectiveDate());
cell.getCellStyle().setDataFormat(HSSFDataFormat.getBuiltinFormat("d-mmm-yy"));
```

### 5.2 为每个单元格创建新样式

另一个常见的错误是工作簿中有太多相似的样式：

```java
CellStyle style1 = codeToCreateCellStyle();
Cell cell1 = row1.createCell(0);
cell1.setCellStyle(style1);

CellStyle style2 = codeToCreateCellStyle();
Cell cell2 = row1.createCell(1);
cell2.setCellStyle(style2);
```

CellStyle的作用域限定于一个工作簿，因此，类似的样式应该由多个单元格共享。在上面的示例中，该样式应该只创建一次，并在cell1和cell2之间共享：

```java
CellStyle style1 = codeToCreateCellStyle();
Cell cell1 = row1.createCell(0);
cell1.setCellStyle(style1);
cell1.setCellValue("Hello");

Cell cell2 = row1.createCell(1);
cell2.setCellStyle(style1);
cell2.setCellValue("world!");
```

## 6. 总结

在本文中，我们学习了如何使用CellStyle及其Font属性来设置Apache POI中单元格的样式。如果我们能够避免一些陷阱，那么设置单元格样式相对简单，正如本文所述。

在代码示例中，我们展示了如何根据需要以编程方式设置电子表格文档的样式，就像使用Excel应用程序本身一样。当需要生成美观的数据呈现电子表格时，这一点至关重要。