---
layout: post
title:  使用Java查找Excel电子表格中的最后一行
category: apache
copyright: apache
excerpt: Apache POI
---

## 1. 概述

在本教程中，我们将讨论如何使用Java和Apache POI查找Excel电子表格中的最后一行。

首先，我们将了解如何使用Apache POI从文件中获取单行数据。然后，我们将学习统计工作表中所有行数的方法。最后，我们将合并这些方法以获取给定工作表的最后一行。

## 2. 获取单行

众所周知，**Apache POI提供了一个抽象层，用Java来表示Microsoft文档**，[包括Excel](https://www.baeldung.com/java-microsoft-excel)。我们可以访问文件中的工作表，甚至可以读取和修改每个单元格。

首先从Excel文件中获取一行数据，在继续下一步之前，我们需要从文件中获取工作表：

```java
Workbook workbook = new XSSFWorkbook(fileLocation);
Sheet sheet = workbook.getSheetAt(0);
```

**Workbook是Excel文件的Java表示，而Sheet是Workbook中的主要结构**。Worksheet是Sheet最常见的子类型，表示单元格网格。

当我们在Java中打开工作表时，我们可以访问它包含的数据，即行数据。要获取单行，我们可以使用[getRow(int)](https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Sheet.html#getRow-int-)方法：

```java
Row row = sheet.getRow(2);
```

**该方法返回Row对象–Excel文件中单行的高级表示**，如果该行不存在，则返回null。

如我们所见，我们需要提供一个参数，即请求行的索引(从0开始)，遗憾的是，没有可用的API可以直接获取最后一行。

## 3. 查找行数

我们刚刚学习了如何使用Java从 Excel文件中获取单行数据。现在，让我们找到给定Sheet上最后一行的索引。

Apache POI提供了两种帮助计算行数的方法：getLastRowNum()和getPhysicalNumberOfRows()，让我们分别看一下。

### 3.1 使用getLastRowNum()

根据文档，[getLastRowNum()](https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Sheet.html#getLastRowNum--)方法返回工作表上最后一个初始化行的编号(从0开始)，如果不存在行，则返回-1：

```java
int lastRowNum = sheet.getLastRowNum();
```

一旦我们获取了lastRowNum，我们现在就可以使用getRow()方法轻松访问最后一行。

需要注意的是，**之前有内容，之后被设置为空的行可能仍然会被计为行**。因此，结果可能与预期不符。为了理解这一点，我们需要了解更多关于物理行的知识。

### 3.2 使用getPhysicalNumberOfRows()

检查Apache POI文档，我们可以找到一个与行相关的特殊术语-物理行。

只要行包含任何数据，它就始终被解释为物理行。**行初始化不仅在于该行中任何单元格包含文本或公式，还在于它们包含一些与格式相关的数据(例如背景颜色、行高或使用的非默认字体)**。换句话说，**每一行初始化后都是物理行**。

为了获取物理行数，Apache POI提供了[getPhysicalNumberOfRows()](https://poi.apache.org/apidocs/dev/org/apache/poi/ss/usermodel/Sheet.html#getPhysicalNumberOfRows--)方法：

```java
int physicalRows = sheet.getPhysicalNumberOfRows();
```

根据物理行解释，结果可能与使用getLastRowNum()方法获取的数字不同。

## 4. 获取最后一行

现在，让我们针对更复杂的Excel网格测试这两种方法：

![](/assets/images/2025/apache/javaexcelfindlastrow01.png)

这里，前几行包含文本数据、公式(=A1)计算出的值以及相应更改的背景颜色。然后，第4行的高度已修改，而第5行和第6行保持不变。第7行再次包含文本，在第8行，文本先前已设置格式，但后来被清除，第9行及后续行均未编辑。

让我们检查一下计数方法的结果：

```java
assertEquals(7, sheet.getLastRowNum());
assertEquals(6, sheet.getPhysicalNumberOfRows());
```

正如我们之前提到的，**最后一行的行号和物理行数在某些情况下是不同的**。

现在让我们根据索引获取行：

```java
assertNotNull(sheet.getRow(0)); // data
assertNotNull(sheet.getRow(1)); // formula
assertNotNull(sheet.getRow(2)); // green
assertNotNull(sheet.getRow(3)); // height
assertNull(sheet.getRow(4));
assertNull(sheet.getRow(5));
assertNotNull(sheet.getRow(6)); // last?
assertNotNull(sheet.getRow(7)); // cleared later
assertNull(sheet.getRow(8));
...
```

可以看到，**getPhysicalNumberOfRows()返回的是工作表中非空(即已初始化)行的总数，getLastRowNum()的值是最后一行非空行的索引**。

因此，我们可以获取工作表上的最后一行：

```java
Row lastRow = null;
int lastRowNum = sheet.getLastRowNum();
if (lastRowNum >= 0) {
    lastRow = sheet.getRow(lastRowNum);
}
```

但是，我们必须记住，**Apache POI返回的最后一行并不总是显示文本或公式的行**，尤其是在某些UI编辑器(例如Microsoft Excel)中。

## 5. 总结

在本文中，我们检查了Apache POI API并从给定的Excel文件中获取了最后一行。

我们首先回顾了在Java中打开电子表格的一些基本方法，然后，我们介绍了getRow(int)方法来检索单行数据；之后，我们检查了getLastRowNum()和getPhysicalNumberOfRows()的值，并解释了它们的区别。