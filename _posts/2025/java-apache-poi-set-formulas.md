---
layout: post
title:  使用Apache POI在Excel中设置公式
category: apache
copyright: apache
excerpt: Apache POI
---

## 1. 概述

在本快速教程中，我们将通过一个简单的示例讨论如何使用[Apache POI](https://poi.apache.org/)在Microsoft Excel电子表格中设置公式。

## 2. Apache POI

Apache POI是一个流行的开源Java库，它为程序员提供创建、修改和显示MS Office文件的API。

它使用[Workbook](https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Workbook.html)来表示Excel文件及其元素，Excel文件中的[Cell](https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Cell.html)可以具有不同的类型，例如FORMULA。

为了查看Apache POI的实际运行情况，我们将设置一个公式来减去[Excel](https://github.com/eugenp/tutorials/blob/master/apache-poi-2/src/main/resources/com/baeldung/poi/excel/setformula/SetFormulaTest.xlsx)文件中A列和B列值的总和，链接的文件包含以下数据：

![](/assets/images/2025/apache/javaapachepoisetformulas01.png)

## 3. 依赖

首先，我们需要将POI依赖添加到项目的pom.xml文件中；**要使用Excel 2007+工作簿，我们应该使用[poi-ooxml](https://mvnrepository.com/artifact/org.apache.poi/poi-ooxml)**：

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.2.5</version>
</dependency>
```

**请注意，对于早期版本的Excel，我们应该使用[poi](https://mvnrepository.com/artifact/org.apache.poi/poi)依赖**。

## 4. 单元格查找

首先，让我们打开文件并构建适当的工作簿：

```java
FileInputStream inputStream = new FileInputStream(new File(fileLocation));
XSSFWorkbook excel = new XSSFWorkbook(inputStream);
```

然后，**我们需要创建或查找要使用的单元格**；使用之前共享的数据，我们想要编辑单元格C1。

这是在第一张表的第一行，我们可以向POI询问第一个空白列：

```java
XSSFSheet sheet = excel.getSheetAt(0);
int lastCellNum = sheet.getRow(0).getLastCellNum();
XSSFCell formulaCell = sheet.getRow(0).createCell(lastCellNum + 1);
```

## 5. 公式

接下来，我们要在查找的单元格上设置一个公式。

如前所述，让我们从A列的总和中减去B列的总和，在Excel中，这将是：

```text
=SUM(A:A)-SUM(B:B)
```

我们可以使用setCellFormula方法将其写入我们的formulaCell中：

```java
formulaCell.setCellFormula("SUM(A:A)-SUM(B:B)");
```

**现在，这还不能计算公式的值**；为此，我们需要使用POI的XSSFFormulaEvaluator：

```java
XSSFFormulaEvaluator formulaEvaluator = excel.getCreationHelper().createFormulaEvaluator();
formulaEvaluator.evaluateFormulaCell(formulaCell);
```

结果将设置在下一个空列的第一个单元格中：

![](/assets/images/2025/apache/javaapachepoisetformulas02.png)

我们可以看到，结果被计算并保存在C列的第一个单元格中，公式也显示在公式栏中。

请注意，[FormulaEvaluator](https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/FormulaEvaluator.html)类为我们提供了其他方法来评估Excel工作簿中的FORMULA，例如evaluateAll，它将循环遍历所有单元格并对其进行评估。

## 6. 总结

在本教程中，我们展示了如何使用Apache POI API在Java中为Excel文件中的单元格设置公式。