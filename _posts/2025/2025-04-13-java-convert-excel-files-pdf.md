---
layout: post
title:  使用Java将Excel文件转换为PDF
category: libraries
copyright: libraries
excerpt: iText
---

## 1. 简介

在本文中，我们将探讨如何使用[Apache POI](https://www.baeldung.com/java-microsoft-excel)和[iText](https://www.baeldung.com/java-pdf-creation)库在Java中将Excel文件转换为PDF。Apache POI负责Excel文件解析和数据提取，而iText负责PDF文档的创建和格式化，利用它们的优势，我们可以高效地转换Excel数据，同时保留其原始格式和样式。

## 2. 添加依赖

在开始实现之前，我们需要将Apache POI和iText库添加到项目中，在pom.xml文件中，添加以下依赖：

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
</dependency>
<dependency>
    <groupId>com.itextpdf</groupId>
    <artifactId>itextpdf</artifactId>
</dependency>
```

可以从Maven Central下载[Apache POI](https://mvnrepository.com/artifact/org.apache.poi/poi)和[iText](https://mvnrepository.com/artifact/com.itextpdf/itextpdf)库的最新版本。

## 3. 加载Excel文件

![](/assets/images/2025/libraries/javaconvertexcelfilespdf01.png)

有了这些库，让我们使用Apache POI加载目标Excel文件。首先，我们使用[FileInputStream](https://www.baeldung.com/reading-file-in-java)打开Excel文件，并创建一个表示已加载工作簿的XSSFWorkbook对象：

```java
FileInputStream inputStream = new FileInputStream(excelFilePath);
XSSFWorkbook workbook = new XSSFWorkbook(inputStream);
```

我们将使用这个对象来访问单个工作表及其数据。

## 4. 创建PDF文档

接下来，我们将利用iText创建一个新的PDF文档：

```java
Document document = new Document();
PdfWriter.getInstance(document, new FileOutputStream(pdfFilePath));
document.open();
```

这将创建一个新的Document对象，并将其与负责写入PDF内容的PDFWriter实例关联。最后，我们通过[FileOutputStream](https://www.baeldung.com/java-write-to-file)指定PDF的所需输出位置。

## 5. 解析Excel数据

文档准备好后，我们将遍历工作表中的每一行以提取单元格值：

```java
void addTableData(PdfPTable table) throws DocumentException, IOException {
    XSSFSheet worksheet = workbook.getSheetAt(0);
    Iterator<Row> rowIterator = worksheet.iterator();
    while (rowIterator.hasNext()) {
        Row row = rowIterator.next();
        if (row.getRowNum() == 0) {
            continue;
        }
        for (int i = 0; i < row.getPhysicalNumberOfCells(); i++) {
            Cell cell = row.getCell(i);
            String cellValue;
            switch (cell.getCellType()) {
                case STRING:
                    cellValue = cell.getStringCellValue();
                    break;
                case NUMERIC:
                    cellValue = String.valueOf(BigDecimal.valueOf(cell.getNumericCellValue()));
                    break;
                case BLANK:
                default:
                    cellValue = "";
                    break;
            }
            PdfPCell cellPdf = new PdfPCell(new Phrase(cellValue));
            table.addCell(cellPdf);
        }
    }
}
```

代码首先创建一个与工作表第一行列数匹配的PdfTable对象，然后，遍历工作表中的每一行，提取单元格值并将其织入PDF表格。**但是，目前不支持Excel公式，因此将返回空字符串**。

对于每个提取出的单元格值，我们使用包含提取数据的Phrase创建一个新的PdfPCell对象。Phrase是一个iText元素，表示格式化的文本字符串。

## 6. 保留Excel样式

**使用Apache POI和iText的主要优势之一是能够保留原始Excel文件的格式和样式**，包括字体样式、颜色和对齐方式。

通过从Apache POI访问相关的单元格样式信息，我们可以使用iText将其应用于PDF文档中的相应元素。然而，需要注意的是，这种方法虽然保留了格式和样式，但可能无法完全复制从Excel直接导出或使用打印机驱动程序打印的PDF的外观和风格。对于更复杂的格式需求，可能需要进行额外的调整。

### 6.1 字体样式

我们将创建一个专用的getCellStyle(Cell cell)方法，从与每个单元格关联的CellStyle对象中提取字体、颜色等样式信息：

```java
Font getCellStyle(Cell cell) throws DocumentException, IOException {
    Font font = new Font();
    CellStyle cellStyle = cell.getCellStyle();
    org.apache.poi.ss.usermodel.Font cellFont = cell.getSheet()
            .getWorkbook()
            .getFontAt(cellStyle.getFontIndexAsInt());

    if (cellFont.getItalic()) {
        font.setStyle(Font.ITALIC);
    }

    if (cellFont.getStrikeout()) {
        font.setStyle(Font.STRIKETHRU);
    }

    if (cellFont.getUnderline() == 1) {
        font.setStyle(Font.UNDERLINE);
    }

    short fontSize = cellFont.getFontHeightInPoints();
    font.setSize(fontSize);

    if (cellFont.getBold()) {
        font.setStyle(Font.BOLD);
    }

    String fontName = cellFont.getFontName();
    if (FontFactory.isRegistered(fontName)) {
        font.setFamily(fontName);
    } else {
        logger.warn("Unsupported font type: {}", fontName);
        font.setFamily("Helvetica");
    }

    return font;
}
```

Phrase对象可以接收单元格值和Front对象作为其构造函数的参数：

```java
PdfPCell cellPdf = new PdfPCell(new Phrase(cellValue, getCellStyle(cell));
```

这使我们能够控制PDF单元格中文本的内容和格式，请注意，**iText的内置字体仅限于Courier、Helvetica和TimesRoman**。因此，在直接应用提取的单元格之前，我们应该检查iText是否支持该单元格的字体系列，如果Excel文件使用了其他字体系列，则不会反映在PDF输出中。

### 6.2 背景颜色样式

除了保留字体样式外，我们还希望确保Excel文件中单元格的背景颜色准确反映在生成的PDF中。为此，我们将创建一个新方法setBackgroundColor()，用于从Excel单元格中提取背景颜色信息并将其应用于相应的PDF单元格。

```java
void setBackgroundColor(Cell cell, PdfPCell cellPdf) {
    short bgColorIndex = cell.getCellStyle()
            .getFillForegroundColor();
    if (bgColorIndex != IndexedColors.AUTOMATIC.getIndex()) {
        XSSFColor bgColor = (XSSFColor) cell.getCellStyle()
                .getFillForegroundColorColor();
        if (bgColor != null) {
            byte[] rgb = bgColor.getRGB();
            if (rgb != null && rgb.length == 3) {
                cellPdf.setBackgroundColor(new BaseColor(rgb[0] & 0xFF, rgb[1] & 0xFF, rgb[2] & 0xFF));
            }
        }
    }
}
```

### 6.3 对齐样式

Apache POI在CellStyle对象上提供了getAlignment()方法，该方法返回一个表示对齐方式的常量值，一旦我们获得了映射的iText对齐常量，我们就可以使用setHorizontalAlignment()方法将其设置在PdfPCell对象上。

以下是如何结合对齐提取和应用的示例：

```java
void setCellAlignment(Cell cell, PdfPCell cellPdf) {
    CellStyle cellStyle = cell.getCellStyle();

    HorizontalAlignment horizontalAlignment = cellStyle.getAlignment();

    switch (horizontalAlignment) {
        case LEFT:
            cellPdf.setHorizontalAlignment(Element.ALIGN_LEFT);
            break;
        case CENTER:
            cellPdf.setHorizontalAlignment(Element.ALIGN_CENTER);
            break;
        case JUSTIFY:
        case FILL:
            cellPdf.setVerticalAlignment(Element.ALIGN_JUSTIFIED);
            break;
        case RIGHT:
            cellPdf.setHorizontalAlignment(Element.ALIGN_RIGHT);
            break;
    }
}
```

现在，让我们更新现有代码，遍历单元格以包含字体和背景颜色样式：

```java
PdfPCell cellPdf = new PdfPCell(new Phrase(cellValue, getCellStyle(cell)));
setBackgroundColor(cell, cellPdf);
setCellAlignment(cell, cellPdf);
```

请注意，生成的Excel看起来与从Excel导出的PDF(或通过打印机驱动程序打印的PDF)不同。

## 7. 保存PDF文档

最后，我们可以将生成的PDF文档保存到所需的位置，这涉及关闭PDF文档对象并确保所有资源均已正确释放：

```java
document.add(table);
document.close();
workbook.close();
```

![](/assets/images/2025/libraries/javaconvertexcelfilespdf02.png)

## 8. 总结

我们学习了如何使用Apache POI和iText在Java中将Excel文件转换为PDF，通过结合Apache POI的Excel处理功能和iText的PDF生成功能，我们可以无缝地保留格式并将Excel的样式应用到PDF中。