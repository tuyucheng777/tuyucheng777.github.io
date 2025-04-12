---
layout: post
title:  使用Apache POI为整行应用粗体文本样式
category: apache
copyright: apache
excerpt: Apache POI
---

## 1. 简介

在本快速教程中，我们将探索使用[Apache POI库](https://www.baeldung.com/java-microsoft-excel)将粗体字体样式应用于Excel工作表整行的有效方法，通过简单易懂的示例和宝贵的见解，我们将深入了解每种方法的细微差别。

## 2. 依赖

让我们从写入和加载Excel文件所需的依赖[poi](https://mvnrepository.com/artifact/org.apache.poi/poi)开始：

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>5.2.5</version>
</dependency>
```

## 3. 场景和辅助方法

我们的场景是创建一个包含标题行和几行数据的[工作表](https://www.baeldung.com/java-microsoft-excel)，然后，我们将为标题行使用的字体定义粗体样式。最后，我们将创建一些方法来设置这种粗体样式。最重要的是，我们将了解为什么需要多个方法来执行此操作，因为显而易见的选择(setRowStyle())无法按预期工作。

**为了方便创建工作表，我们将从一个工具类开始**，让我们编写几个方法来创建单元格以及包含单元格的行：

```java
public class PoiUtils {

    private static void newCell(Row row, String value) {
        short cellNum = row.getLastCellNum();
        if (cellNum == -1)
            cellNum = 0;

        Cell cell = row.createCell(cellNum);
        cell.setCellValue(value);
    }

    public static Row newRow(Sheet sheet, String... rowValues) {
        Row row = sheet.createRow(sheet.getLastRowNum() + 1);

        for (String value : rowValues) {
            newCell(row, value);
        }

        return row;
    }

    // ...
}
```

**然后，要创建粗体字体样式，我们首先从[Workbook](https://www.baeldung.com/java-read-dates-excel#apache-poi-overview)中创建一个字体，然后调用setBold(true)**。其次，我们将创建一个使用粗体字体的CellStyle：

```java
public static CellStyle boldFontStyle(Workbook workbook) {
    Font boldFont = workbook.createFont();
    boldFont.setBold(true);

    CellStyle boldStyle = workbook.createCellStyle();
    boldStyle.setFont(boldFont);

    return boldStyle;
}
```

最后，为了将工作表写入文件，我们需要在工作簿上调用write()：

```java
public static void write(Workbook workbook, Path path) throws IOException {
    try (FileOutputStream fileOut = new FileOutputStream(path.toFile())) {
        workbook.write(fileOut);
    }
}
```

## 4. 使用setRowStyle()的注意事项

查看POI API时，我们任务最明显的选择是[Row.setRowStyle()](https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Row.html#setRowStyle-org.apache.poi.ss.usermodel.CellStyle-)，**不幸的是，此方法工作不稳定，并且目前存在一个[bug](https://bz.apache.org/bugzilla/show_bug.cgi?id=48344)。问题似乎是Microsoft Office渲染器忽略了行样式，而只关心单元格样式**。

**另一方面，它可以与OpenOffice兼容，但前提是我们使用SXSSFWorkbook实现，该实现适用于大文件**。为了测试这一点，我们将从一个示例工作表方法开始：

```java
private void writeSampleSheet(Path destination, Workbook workbook) throws IOException {
    Sheet sheet = workbook.createSheet();
    CellStyle boldStyle = PoiUtils.boldFontStyle(workbook);

    Row header = PoiUtils.newRow(sheet, "Name", "Value", "Details");
    header.setRowStyle(boldStyle);

    PoiUtils.newRow(sheet, "Albert", "A", "First");
    PoiUtils.newRow(sheet, "Jane", "B", "Second");

    PoiUtils.write(workbook, destination);
}
```

然后，使用断言方法检查第一行和第二行的样式。**首先，我们断言第一行具有粗体字体样式；然后，对于其中的每个单元格，我们断言其默认样式与我们为第一行设置的样式不同，这断言了我们的行样式具有优先级**；最后，我们断言第二行未应用任何样式：

```java
private void assertRowStyleAppliedAndDefaultCellStylesDontMatch(Path sheetFile) throws IOException, InvalidFormatException {
    try (Workbook workbook = new XSSFWorkbook(sheetFile.toFile())) {
        Sheet sheet = workbook.getSheetAt(0);
        Row row0 = sheet.getRow(0);

        XSSFCellStyle rowStyle = (XSSFCellStyle) row0.getRowStyle();
        assertTrue(rowStyle.getFont().getBold());

        row0.forEach(cell -> {
            XSSFCellStyle style = (XSSFCellStyle) cell.getCellStyle();
            assertNotEquals(rowStyle, style);
        });

        Row row1 = sheet.getRow(1);
        XSSFCellStyle row1Style = (XSSFCellStyle) row1.getRowStyle();
        assertNull(row1Style);

        Files.delete(sheetFile);
    }
}
```

最终，我们的测试包括将工作表写入临时文件，然后将其读回。我们确保样式已应用，测试第一行和第二行的样式，然后删除该文件：

```java
@Test
void givenXssfWorkbook_whenSetRowStyle1stRow_thenOnly1stRowStyled() throws IOException, InvalidFormatException {
    Path sheetFile = Files.createTempFile("xssf-row-style", ".xlsx");

    try (Workbook workbook = new XSSFWorkbook()) {
        writeSampleSheet(sheetFile, workbook);
    }

    assertRowStyleAppliedAndDefaultCellStylesDontMatch(sheetFile);
}
```

**运行此测试时，我们现在可以检查是否只为第一行获得了粗体样式，这正是我们想要的**。

## 5. 使用setCellStyle()设置行中的单元格

鉴于setRowStyle()的问题，我们只剩下setCellStyle()了，我们需要它来设置行中每个需要应用粗体样式的单元格的样式。所以，**我们来修改一下原来的方法，遍历标题的每一行，并使用粗体样式调用setCellStyle()**：

```java
@Test
void givenXssfWorkbook_whenSetCellStyleForEachRow_thenAllCellsContainStyle() throws IOException, InvalidFormatException {
    Path sheetFile = Files.createTempFile("xssf-cell-style", ".xlsx");

    try (Workbook workbook = new XSSFWorkbook()) {
        Sheet sheet = workbook.createSheet();
        CellStyle boldStyle = PoiUtils.boldFontStyle(workbook);

        Row header = PoiUtils.newRow(sheet, "Name", "Value", "Details");
        header.forEach(cell -> cell.setCellStyle(boldStyle));

        PoiUtils.newRow(sheet, "Albert", "A", "First");
        PoiUtils.write(workbook, sheetFile);
    }

    // ...
}
```

**这样，我们就能保证样式在不同格式和平台上保持一致**。最后，我们来断言一下，行样式未设置，并且第一行的每个单元格都包含粗体字体样式：

```java
try (Workbook workbook = new XSSFWorkbook(sheetFile.toFile())) {
    Sheet sheet = workbook.getSheetAt(0);
    Row row0 = sheet.getRow(0);

    XSSFCellStyle rowStyle = (XSSFCellStyle) row0.getRowStyle();
    assertNull(rowStyle);

    row0.forEach(cell -> {
        XSSFCellStyle style = (XSSFCellStyle) cell.getCellStyle();
        assertTrue(style.getFont().getBold());
    });

    Row row1 = sheet.getRow(1);
    rowStyle = (XSSFCellStyle) row1.getRowStyle();
    assertNull(rowStyle);

    Files.delete(sheetFile);
}
```

请注意，我们在这里使用XSSFWorkbook只是为了方便，此方法在所有Workbook实现中均一致有效。

## 6. 总结

在本文中，我们了解到虽然setRowStyle()可能无法可靠地实现我们的目标，但我们发现了一个使用setCellStyle()的强大替代方案。现在，我们可以自信地设置Excel工作表中行的格式，确保在不同平台上获得一致且具有视觉冲击力的结果。