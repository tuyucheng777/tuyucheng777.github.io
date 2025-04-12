---
layout: post
title:  使用Java操作Microsoft Excel
category: apache
copyright: apache
excerpt: Apache POI
---

## 1. 概述

在本教程中，我们将演示如何使用**Apache POI、JExcel和Fastexcel API处理Excel电子表格**。

这些库可用于动态读取、写入和修改Excel电子表格的内容，并提供将Microsoft Excel集成到Java应用程序的有效方法。

## 2. Maven依赖

首先，我们需要在pom.xml文件中添加以下依赖：

```xml
<dependency> 
  <groupId>org.apache.poi</groupId>
  <artifactId>poi</artifactId> 
  <version>5.3.0</version> 
</dependency> 
<dependency> 
  <groupId>org.apache.poi</groupId> 
  <artifactId>poi-ooxml</artifactId> 
  <version>5.3.0</version> 
</dependency>
<dependency>
  <groupId>org.jxls</groupId>
  <artifactId>jxls-jexcel</artifactId>
  <version>1.0.9</version>
</dependency>
<dependency>
    <groupId>org.dhatim</groupId>
    <artifactId>fastexcel-reader</artifactId>
    <version>0.18.1</version>
</dependency>
<dependency>
    <groupId>org.dhatim</groupId>
    <artifactId>fastexcel</artifactId>
    <version>0.18.1</version>
</dependency>
```

可以从Maven Central下载[poi](https://mvnrepository.com/artifact/org.apache.poi/poi)、[poi-ooxml](https://mvnrepository.com/artifact/org.apache.poi/poi-ooxml)、[jxls-jexcel](https://mvnrepository.com/artifact/org.jxls/jxls-jexcel)、[fastexcel-reader](https://mvnrepository.com/artifact/org.dhatim/fastexcel-reader)和[fastexcel](https://mvnrepository.com/artifact/org.dhatim/fastexcel)的最新版本。

## 3. Apache POI 

**Apache POI库支持.xls和.xlsx文件**，并且比其他用于处理Excel文件的Java库更复杂。

它提供了用于对Excel文件进行建模的Workbook接口，以及用于对Excel文件元素进行建模的Sheet、Row和Cell接口，以及针对两种文件格式的每个接口的实现。

当使用较新的.xlsx文件格式时，我们将使用XSSFWorkbook、XSSFSheet、XSSFRow和XSSFCell类。

为了使用较旧的.xls格式，我们使用HSSFWorkbook、HSSFSheet、HSSFRow和HSSFCell类。

### 3.1 从Excel读取

让我们创建一个方法，打开一个.xlsx文件，然后从该文件的第一张表中读取内容。

读取单元格内容的方法取决于单元格中数据类型，可以使用Cell接口的getCellType()方法确定单元格内容的类型。

首先，让我们从给定位置打开文件：

```java
FileInputStream file = new FileInputStream(new File(fileLocation));
Workbook workbook = new XSSFWorkbook(file);
```

接下来，让我们检索文件的第一张表并遍历每一行：

```java
Sheet sheet = workbook.getSheetAt(0);

Map<Integer, List<String>> data = new HashMap<>();
int i = 0;
for (Row row : sheet) {
    data.put(i, new ArrayList<String>());
    for (Cell cell : row) {
        switch (cell.getCellType()) {
            case STRING: ... break;
            case NUMERIC: ... break;
            case BOOLEAN: ... break;
            case FORMULA: ... break;
            default: data.get(new Integer(i)).add(" ");
        }
    }
    i++;
}
```

**Apache POI针对每种类型的数据都有不同的读取方法**，让我们进一步阐述一下上面每个switch case的内容。

当单元格类型枚举值为STRING时，将使用Cell接口的getRichStringCellValue()方法读取内容：

```java
data.get(new Integer(i)).add(cell.getRichStringCellValue().getString());
```

具有NUMERIC内容类型的单元格可以包含日期或数字，并以以下方式读取：

```java
if (DateUtil.isCellDateFormatted(cell)) {
    data.get(i).add(cell.getDateCellValue() + "");
} else {
    data.get(i).add(cell.getNumericCellValue() + "");
}
```

对于BOOLEAN值，我们有getBooleanCellValue()方法：

```java
data.get(i).add(cell.getBooleanCellValue() + "");
```

当单元格类型为FORMULA时，我们可以使用getCellFormula()方法：

```java
data.get(i).add(cell.getCellFormula() + "");
```

### 3.2 写入Excel 

Apache POI使用上一节中介绍的相同接口来写入Excel文件，并且比JExcel对样式有更好的支持。

让我们创建一种方法，将人员列表写入标题为“Persons”的工作表。

首先，我们将创建并设置包含“Name”和“Age”单元格的标题行：

```java
Workbook workbook = new XSSFWorkbook();

Sheet sheet = workbook.createSheet("Persons");
sheet.setColumnWidth(0, 6000);
sheet.setColumnWidth(1, 4000);

Row header = sheet.createRow(0);

CellStyle headerStyle = workbook.createCellStyle();
headerStyle.setFillForegroundColor(IndexedColors.LIGHT_BLUE.getIndex());
headerStyle.setFillPattern(FillPatternType.SOLID_FOREGROUND);

XSSFFont font = ((XSSFWorkbook) workbook).createFont();
font.setFontName("Arial");
font.setFontHeightInPoints((short) 16);
font.setBold(true);
headerStyle.setFont(font);

Cell headerCell = header.createCell(0);
headerCell.setCellValue("Name");
headerCell.setCellStyle(headerStyle);

headerCell = header.createCell(1);
headerCell.setCellValue("Age");
headerCell.setCellStyle(headerStyle);
```

接下来，我们用不同的风格来写一下表格的内容：

```java
CellStyle style = workbook.createCellStyle();
style.setWrapText(true);

Row row = sheet.createRow(2);
Cell cell = row.createCell(0);
cell.setCellValue("John Smith");
cell.setCellStyle(style);

cell = row.createCell(1);
cell.setCellValue(20);
cell.setCellStyle(style);
```

最后，我们将内容写入当前目录中的“temp.xlsx”文件并关闭工作簿：

```java
File currDir = new File(".");
String path = currDir.getAbsolutePath();
String fileLocation = path.substring(0, path.length() - 1) + "temp.xlsx";

FileOutputStream outputStream = new FileOutputStream(fileLocation);
workbook.write(outputStream);
workbook.close();
```

让我们在JUnit测试中测试上述方法，该测试将内容写入temp.xlsx文件，然后读取同一文件以验证它是否包含我们所写的文本：

```java
public class ExcelIntegrationTest {

    private ExcelPOIHelper excelPOIHelper;
    private static String FILE_NAME = "temp.xlsx";
    private String fileLocation;

    @Before
    public void generateExcelFile() throws IOException {
        File currDir = new File(".");
        String path = currDir.getAbsolutePath();
        fileLocation = path.substring(0, path.length() - 1) + FILE_NAME;

        excelPOIHelper = new ExcelPOIHelper();
        excelPOIHelper.writeExcel();
    }

    @Test
    public void whenParsingPOIExcelFile_thenCorrect() throws IOException {
        Map<Integer, List<String>> data
                = excelPOIHelper.readExcel(fileLocation);

        assertEquals("Name", data.get(0).get(0));
        assertEquals("Age", data.get(0).get(1));

        assertEquals("John Smith", data.get(1).get(0));
        assertEquals("20", data.get(1).get(1));
    }
}
```

## 4. JExcel 

JExcel库是一个轻量级库，其优点是比Apache POI更易于使用，但缺点是它仅提供对处理.xls(1997-2003)格式的Excel文件的支持。

**目前，不支持.xlsx文件**。

### 4.1 从Excel读取

为了处理Excel文件，此库提供了一系列类来表示Excel文件的不同部分。**Workbook类表示整个工作表集合**，Sheet类表示单个工作表，而Cell类表示电子表格的单个单元格。

让我们编写一个方法，从指定的Excel文件创建一个工作簿，获取文件的第一个工作表，然后遍历其内容并将每一行添加到HashMap中：

```java
public class JExcelHelper {

    public Map<Integer, List<String>> readJExcel(String fileLocation) throws IOException, BiffException {
        Map<Integer, List<String>> data = new HashMap<>();

        Workbook workbook = Workbook.getWorkbook(new File(fileLocation));
        Sheet sheet = workbook.getSheet(0);
        int rows = sheet.getRows();
        int columns = sheet.getColumns();

        for (int i = 0; i < rows; i++) {
            data.put(i, new ArrayList<String>());
            for (int j = 0; j < columns; j++) {
                data.get(i)
                        .add(sheet.getCell(j, i)
                                .getContents());
            }
        }
        return data;
    }
}
```

### 4.2 写入Excel 

为了写入Excel文件，JExcel库提供了与上面类似的类，它们模拟电子表格文件：WritableWorkbook、WritableSheet和WritableCell。

**WritableCell类具有与可写入的不同类型内容相对应的子类**：Label、DateTime、Number、Boolean、Blank和Formula。

该库还支持基本格式，例如控制字体、颜色和单元格宽度。

让我们编写一个方法，在当前目录中创建一个名为“temp.xls”的工作簿，然后写入我们在Apache POI部分中写入的相同内容。

首先，让我们创建工作簿：

```java
File currDir = new File(".");
String path = currDir.getAbsolutePath();
String fileLocation = path.substring(0, path.length() - 1) + "temp.xls";

WritableWorkbook workbook = Workbook.createWorkbook(new File(fileLocation));
```

接下来，让我们创建第一个工作表并编写Excel文件的标题，其中包含“Name”和“Age”单元格：

```java
WritableSheet sheet = workbook.createSheet("Sheet 1", 0);

WritableCellFormat headerFormat = new WritableCellFormat();
WritableFont font = new WritableFont(WritableFont.ARIAL, 16, WritableFont.BOLD);
headerFormat.setFont(font);
headerFormat.setBackground(Colour.LIGHT_BLUE);
headerFormat.setWrap(true);

Label headerLabel = new Label(0, 0, "Name", headerFormat);
sheet.setColumnView(0, 60);
sheet.addCell(headerLabel);

headerLabel = new Label(1, 0, "Age", headerFormat);
sheet.setColumnView(0, 40);
sheet.addCell(headerLabel);
```

使用新样式，让我们写入我们创建的表格的内容：

```java
WritableCellFormat cellFormat = new WritableCellFormat();
cellFormat.setWrap(true);

Label cellLabel = new Label(0, 2, "John Smith", cellFormat);
sheet.addCell(cellLabel);
Number cellNumber = new Number(1, 2, 20, cellFormat);
sheet.addCell(cellNumber);
```

记住写入文件并在最后关闭它非常重要，以便其他进程可以使用它，使用Workbook类的write()和close()方法：

```java
workbook.write();
workbook.close();
```

## 5. Fastexcel

**Fastexcel是一个易于使用的库，与Apache POI相比，其功能有限且内存占用较低**。

它使用CompletableFuture提供多线程支持，使其成为处理功能不丰富的大型文件时的绝佳选择。目前，该库仅提供基本样式支持，不支持图表。

### 5.1 从Excel读取

让我们编写一个方法来访问Excel文件并从其第一张工作表中读取数据，我们将数据添加到一个Map中，该Map以行索引为键，以该行内容的列表为值：

```java
public class FastexcelHelper {

    public Map<Integer, List<String>> readExcel(String fileLocation) throws IOException {
        Map<Integer, List<String>> data = new HashMap<>();

        try (FileInputStream file = new FileInputStream(fileLocation); ReadableWorkbook wb = new ReadableWorkbook(file)) {
            Sheet sheet = wb.getFirstSheet();
            try (Stream<Row> rows = sheet.openStream()) {
                rows.forEach(r -> {
                    data.put(r.getRowNum(), new ArrayList<>());

                    for (Cell cell : r) {
                        data.get(r.getRowNum()).add(cell.getRawValue());
                    }
                });
            }
        }

        return data;
    }
}
```

这里我们使用cell.getRawValue()来返回该单元格的字符串值，或者，**根据CellType，我们可以使用Row类中的getCellAsBoolean(int cellIndex)、getCellAsDate(int cellIndex)、getCellAsString(int cellIndex)或getCellAsNumber(int cellIndex)方法来读取单元格的内容**。

### 5.2 写入Excel

与上面提到的库不同，**Fastexcel使用不同的接口来写入Excel文件和读取Excel文件**。

我们首先从OutputStream创建一个Workbook，然后，我们将在默认的第一张工作表中写入“Name”和“Age”标题，并为单元格添加样式细节。接下来，让我们添加另一行，其中包含人员的姓名和年龄：

```java
public void writeExcel() throws IOException {
    File currDir = new File(".");
    String path = currDir.getAbsolutePath();
    String fileLocation = path.substring(0, path.length() - 1) + "fastexcel.xlsx";

    try (OutputStream os = Files.newOutputStream(Paths.get(fileLocation)); Workbook wb = new Workbook(os, "MyApplication", "1.0")) {
        Worksheet ws = wb.newWorksheet("Sheet 1");
        ws.width(0, 25);
        ws.width(1, 15);

        ws.range(0, 0, 0, 1).style().fontName("Arial").fontSize(16).bold().fillColor("3366FF").set();
        ws.value(0, 0, "Name");
        ws.value(0, 1, "Age");

        ws.range(2, 0, 2, 1).style().wrapText(true).set();
        ws.value(2, 0, "John Smith");
        ws.value(2, 1, 20L);
    }
}
```

让我们编写一个小测试来验证我们的代码：

```java
@Test
public void whenParsingExcelFile_thenCorrect() throws IOException {
    Map<Integer, List<String>> data = fastexcelHelper.readExcel(fileLocation);

    assertEquals("Name", data.get(1).get(0));
    assertEquals("Age", data.get(1).get(1));

    assertEquals("John Smith", data.get(3).get(0));
    assertEquals("20", data.get(3).get(1));
}
```

在这里，我们还看到fastexcel-reader库中的Row类使用偏移量为1的变量rowNum，而fastexcel库中的Worksheet类中的value(int r, int c, Object value)方法需要偏移量为0的行索引。

## 6. 总结

在本文中，我们了解了如何使用Apache POI API、JExcel API和Fastexcel API从Java程序读取和写入Excel文件。

在决定使用哪个库时，我们应该考虑每个库的优缺点。例如，Apache POI功能丰富，支持图形，但内存占用较高。相比之下，Fastexcel功能有限，但内存占用比Apache POI更少。