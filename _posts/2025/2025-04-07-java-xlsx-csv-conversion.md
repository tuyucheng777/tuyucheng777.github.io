---
layout: post
title:  如何使用Java将XLSX文件转换为CSV
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

XLSX是Microsoft Excel创建的一种流行的电子表格格式，以能够存储公式和图表等复杂数据结构而闻名。相比之下，CSV(逗号分隔值)是一种更简单的格式，通常用于应用程序之间的数据交换。

**将XLSX文件转换为CSV格式可使数据更易于访问，从而简化数据处理、集成和分析**。

在本教程中，我们将学习如何使用Java将XLSX文件转换为CSV。我们将使用Apache POI读取XLSX文件，并使用Apache Commons CSV和OpenCSV将数据写入CSV文件。

## 2. 读取XLSX文件

为了处理XLSX文件，我们将使用Apache POI，这是一个专为处理Microsoft Office文档而设计的强大Java库。Apache POI为读取和写入Excel文件提供了广泛的支持，使其成为我们转换任务的绝佳选择。

### 2.1 POI依赖

首先，我们需要将[Apache POI依赖](https://mvnrepository.com/artifact/org.apache.poi/poi-ooxml)添加到pom.xml中：

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.2.5</version>
</dependency>
```

此依赖包括处理XLSX文件和处理各种数据结构所需的库。

### 2.2 使用POI打开XLSX文件

要打开和读取XLSX文件，我们将创建一个使用Apache POI的XSSFWorkbook类的方法，我们可以使用此类读取XLSX文件并访问其内容：

```java
public static Workbook openWorkbook(String filePath) throws IOException {
    try (FileInputStream fis = new FileInputStream(filePath)) {
        return WorkbookFactory.create(fis);
    }
}
```

上述方法使用FileInputStream打开指定的XLSX文件，它返回一个包含整个Excel工作簿的Workbook对象，并允许我们访问其工作表和数据。

我们还使用WorkbookFactory.create()方法从输入流创建Workbook对象，并在内部处理文件格式和初始化。

### 2.3 迭代行和列并输出它们

打开XLSX文件后，我们需要遍历其行和列来提取和准备数据以供进一步处理：

```java
public static List<String[]> iterateAndPrepareData(String filePath) throws IOException {
    Workbook workbook = openWorkbook(filePath);
    Sheet sheet = workbook.getSheetAt(0);
    List<String[]> data = new ArrayList<>();
    DataFormatter formatter = new DataFormatter();
    for (Row row : sheet) {
        String[] rowData = new String[row.getLastCellNum()];
        for (int cn = 0; cn < row.getLastCellNum(); cn++) {
            Cell cell = row.getCell(cn);
            rowData[cn] = cell == null ? "" : formatter.formatCellValue(cell);
        }
        data.add(rowData);
    }
    workbook.close();
    return data;
}
```

在这种方法中，我们首先使用getSheetAt(0)从工作簿中检索第一个工作表，然后遍历XLSX文件的每一行和列。

对于工作表中的每个单元格，我们使用DataFormatter将其值转换为格式化的字符串。这些格式化的值存储在字符串数组中，表示来自XLSX文件的一行数据。

最后，我们将每个rowData数组添加到名为data的List<String[]\>中，其中包含从XLSX文件中提取的所有行数据。

## 3. 使用Apache Commons CSV写入CSV文件

要用Java写入CSV文件，我们将使用Apache Commons CSV，它提供了一个用于读取和写入CSV文件的简单高效的API。

### 3.1 依赖

要使用Apache Commons CSV，我们需要将[依赖](https://mvnrepository.com/artifact/org.apache.commons/commons-csv)添加到pom.xml中：

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-csv</artifactId>
    <version>1.11.0</version>
</dependency>
```

这将包括处理CSV文件操作所需的库。

### 3.2 创建CSV文件

接下来，让我们创建一个使用Apache Commons CSV写入CSV文件的方法：

```java
public class CommonsCSVWriter {
    public static void writeCSV(List<String[]> data, String filePath) throws IOException {
        try (FileWriter fw = new FileWriter(filePath);
             CSVPrinter csvPrinter = new CSVPrinter(fw, CSVFormat.DEFAULT)) {
            for (String[] row : data) {
                csvPrinter.printRecord((Object[]) row);
            }
            csvPrinter.flush();
        }
    }
}
```

在CommonsCSVWriter.writeCSV()方法中，我们使用Apache Commons CSV将数据写入CSV文件。

我们为目标文件路径创建一个FileWriter，并初始化一个CSVPrinter来处理写入过程。

该方法迭代数据列表中的每一行，并使用csvPrinter.printRecord()将每一行写入CSV文件。写入完成后，通过刷新和关闭CSVPrinter确保所有资源得到妥善管理。

### 3.3 遍历工作簿并写入CSV文件

现在让我们结合读取XLSX文件和写入CSV文件：

```java
public class ConvertToCSV {
    public static void convertWithCommonsCSV(String xlsxFilePath, String csvFilePath) throws IOException {
        List<String[]> data = XLSXReader.iterateAndPrepareData(xlsxFilePath);
        CommonsCSVWriter.writeCSV(data, csvFilePath);
    }
}
```

在convert(String xlsxFilePath, String csvFilePath)方法中，我们首先使用之前的XLSXReader.iterateAndPrepareData()方法从指定的XLSX文件中提取数据。

然后，我们将提取的数据传递给CommonsCSVWriter.writeCSV()方法，以使用Apache Commons CSV将其写入指定位置的CSV文件。

## 4. 使用OpenCSV写入CSV文件

OpenCSV是另一个用于处理Java中CSV文件的流行库，它提供了一个简单的API来读取和写入CSV文件，让我们尝试将其作为Apache Commons CSV的替代品。

### 4.1 依赖

要使用OpenCSV，我们需要将[依赖](https://mvnrepository.com/artifact/com.opencsv/opencsv)添加到pom.xml中：

```xml
<dependency>
    <groupId>com.opencsv</groupId>
    <artifactId>opencsv</artifactId>
    <version>5.8</version>
</dependency>
```

### 4.2 创建CSV文件

接下来，让我们创建一个使用OpenCSV写入CSV文件的方法：

```java
public static void writeCSV(List<String[]> data, String filePath) throws IOException {
    try (FileWriter fw = new FileWriter(filePath);
         CSVWriter csvWriter = new CSVWriter(fw,
                 CSVWriter.DEFAULT_SEPARATOR,
                 CSVWriter.NO_QUOTE_CHARACTER,
                 CSVWriter.DEFAULT_ESCAPE_CHARACTER,
                 CSVWriter.DEFAULT_LINE_END)) {
        for (String[] row : data) {
            csvWriter.writeNext(row);
        }
    }
}
```

在OpenCSVWriter.writeCSV()方法中，我们使用OpenCSV将数据写入CSV文件。

**我们为指定路径创建一个FileWriter，并使用禁用字段引用并使用默认分隔符和行尾的配置初始化一个CSVWriter**。

该方法遍历提供的数据列表，使用csvWriter.writeNext()将每一行写入文件。try-with-resources语句确保正确关闭FileWriter和CSVWriter，有效管理资源并防止泄漏。

### 4.3 遍历工作簿并写入CSV文件

现在，我们将调整之前的XLSX到CSV的转换逻辑以使用OpenCSV：

```java
public class ConvertToCSV {
    public static void convertWithOpenCSV(String xlsxFilePath, String csvFilePath) throws IOException {
        List<String[]> data = XLSXReader.iterateAndPrepareData(xlsxFilePath);
        OpenCSVWriter.writeCSV(data, csvFilePath);
    }
}
```

## 5. 测试CSV转换

最后，让我们创建一个单元测试来检查我们的CSV转换，测试将使用示例XLSX文件并验证生成的CSV内容：

```java
class ConvertToCSVUnitTest {

    private static final String XLSX_FILE_INPUT = "src/test/resources/xlsxToCsv_input.xlsx";
    private static final String CSV_FILE_OUTPUT = "src/test/resources/xlsxToCsv_output.csv";

    @Test
    void givenXlsxFile_whenUsingCommonsCSV_thenGetValuesAsList() throws IOException {
        ConvertToCSV.convertWithCommonsCSV(XLSX_FILE_INPUT, CSV_FILE_OUTPUT);
        List<String> lines = Files.readAllLines(Paths.get(CSV_FILE_OUTPUT));
        assertEquals("1,Dulce,Abril,Female,United States,32,15/10/2017,1562", lines.get(1));
        assertEquals("2,Mara,Hashimoto,Female,Great Britain,25,16/08/2016,1582", lines.get(2));
    }

    @Test
    void givenXlsxFile_whenUsingOpenCSV_thenGetValuesAsList() throws IOException {
        ConvertToCSV.convertWithOpenCSV(XLSX_FILE_INPUT, CSV_FILE_OUTPUT);
        List<String> lines = Files.readAllLines(Paths.get(CSV_FILE_OUTPUT));
        assertEquals("1,Dulce,Abril,Female,United States,32,15/10/2017,1562", lines.get(1));
        assertEquals("2,Mara,Hashimoto,Female,Great Britain,25,16/08/2016,1582", lines.get(2));
    }
}
```

在此单元测试中，我们验证Apache Commons CSV和OpenCSV生成的CSV文件是否包含预期值，我们使用示例XLSX文件并检查生成的CSV文件中的特定行以确保转换准确。

以下是输入XLSX文件(xlsxToCsv_input.xlsx)的示例：

![](/assets/images/2025/javaio/javaxlsxcsvconversion01.png)

以下是相应的输出CSV文件(xlsxToCsv_output.csv)：

![](/assets/images/2025/javaio/javaxlsxcsvconversion02.png)

## 6. 总结

使用Apache POI读取并使用Apache Commons CSV或OpenCSV写入，可以高效地将XLSX文件转换为Java中的CSV格式。

这两个CSV库都提供了强大的工具来处理和将不同类型的数据类型写入CSV。