---
layout: post
title:  使用Apache POI在Excel中插入一行
category: apache
copyright: apache
excerpt: Apache POI
---

## 1. 概述

有时，我们可能需要在Java应用程序中操作Excel文件。

在本教程中，我们将特别介绍如何使用[Apache POI](https://www.baeldung.com/java-microsoft-excel)库在Excel文件的两行之间插入新行。

## 2. Maven依赖

首先，我们必须将[poi-ooxml](https://mvnrepository.com/artifact/org.apache.poi/poi-ooxml) Maven依赖添加到pom.xml文件中：

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.3.0</version>
</dependency>
```

## 3. 在两行之间插入行

### 3.1 Apache POI相关类

Apache POI是一个库的集合，每个库都专用于处理特定类型的文件，XSSF库包含用于处理xlsx Excel格式的类。下图展示了用于处理xlsx Excel文件的Apache POI相关接口和类：

![](/assets/images/2025/apache/apachepoiinsertexcelrow01.png)

### 3.2 实现行插入

要在现有Excel工作表中间插入m行，从插入点到最后一行的所有行都应向下移动m行。

首先，我们需要读取Excel文件。在这一步，我们使用XSSFWorkbook类：

```java
Workbook workbook = new XSSFWorkbook(fileLocation);
```

第二步是使用getSheet()方法访问工作簿中的工作表：

```java
Sheet sheet = workbook.getSheetAt(0);
```

第三步是移动行，从当前我们想要开始插入新行的行到工作表的最后一行：

```java
int lastRow = sheet.getLastRowNum(); 
sheet.shiftRows(startRow, lastRow, rowNumber, true, true);
```

在此步骤中，我们使用getLastRowNum()方法获取最后一行的行号，并使用shiftRows()方法移动行，此方法将startRow和lastRow之间的行移动rowNumber的大小。

最后，我们使用createRow()方法插入新行：

```java
sheet.createRow(startRow);
```

值得注意的是，上述实现将保留被移动行的格式。此外，如果我们要移动的范围内存在隐藏行，则在插入新行时，这些隐藏行也会移动。

### 3.3 单元测试

让我们编写一个测试用例，读取资源目录中的工作簿，然后在位置2处插入一行，并将内容写入新的Excel文件。最后，我们用主文件断言结果文件的行号。

让我们定义一个测试用例：

```java
public void givenWorkbook_whenInsertRowBetween_thenRowCreated() {
    int startRow = 2;
    int rowNumber = 1;
    Workbook workbook = new XSSFWorkbook(fileLocation);
    Sheet sheet = workbook.getSheetAt(0);

    int lastRow = sheet.getLastRowNum();
    if (lastRow < startRow) {
        sheet.createRow(startRow);
    }

    sheet.shiftRows(startRow, lastRow, rowNumber, true, true);
    sheet.createRow(startRow);

    FileOutputStream outputStream = new FileOutputStream(NEW_FILE_NAME);
    workbook.write(outputStream);

    File file = new File(NEW_FILE_NAME);

    final int expectedRowResult = 5;
    Assertions.assertEquals(expectedRowResult, workbook.getSheetAt(0).getLastRowNum());

    outputStream.close();
    file.delete();
    workbook.close();
}
```

## 4. 总结

总之，我们学习了如何使用Apache POI库在Excel文件的两行之间插入一行。