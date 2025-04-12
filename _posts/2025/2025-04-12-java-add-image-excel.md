---
layout: post
title:  使用Java将图像添加到Excel文件中的单元格
category: apache
copyright: apache
excerpt: Apache POI
---

## 1. 概述

在本教程中，我们将学习如何使用Java向Excel文件中的单元格添加图像。

我们将使用[Apache POI](https://www.baeldung.com/java-microsoft-excel)动态创建一个Excel文件并向单元格添加图像。

## 2. 项目设置和依赖

Java应用程序可以使用apache-poi动态读取、写入和修改Excel电子表格的内容，它支持.xls和.xlsx Excel格式。

### 2.1 Apache POI API的Maven依赖

首先，让我们将[poi](https://mvnrepository.com/artifact/org.apache.poi/poi-ooxml)依赖添加到项目中：

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>5.2.5</version>
</dependency>
```

### 2.2 Excel工作簿创建

首先，让我们创建一个用于写入的工作簿和工作表，我们可以选择XSSFWorkbook(适用于.xlsx文件)或HSSFWorkbook(适用于.xls文件)，我们使用XSSFWorkbook：

```java
Workbook workbook = new XSSFWorkbook();
Sheet sheet = workbook.createSheet("Avengers");
Row row1 = sheet.createRow(0);
row1.createCell(0).setCellValue("IRON-MAN");
Row row2 = sheet.createRow(1);
row2.createCell(0).setCellValue("SPIDER-MAN");
```

这里，我们创建了一个“Avengers”工作表，并在A1和A2单元格中填写了两个名字。接下来，我们将把复仇者联盟的图片添加到单元格B1和B2中。

## 3. 在工作簿中插入图像

### 3.1 从本地文件读取图像

要添加图片，我们首先需要从项目目录中读取它们。对于我们的项目，资源目录中有两张图片：

- /src/main/resources/ironman.png
- /src/main/resources/spiderman.png

```java
InputStream inputStream1 = TestClass.class.getClassLoader()
    .getResourceAsStream("ironman.png");
InputStream inputStream2 = TestClass.class.getClassLoader()
    .getResourceAsStream("spiderman.png");
```

### 3.2 将图像输入流转换为字节数组

接下来，我们将图像转换为字节数组，这里我们将使用apache-poi中的IOUtils：

```java
byte[] inputImageBytes1 = IOUtils.toByteArray(inputStream1);
byte[] inputImageBytes2 = IOUtils.toByteArray(inputStream2);
```

### 3.3 在工作簿中添加图片

现在，我们将使用字节数组向工作簿添加图片，[支持的图片类型](http://poi.apache.org/components/spreadsheet/quick-guide.html#Images)包括PNG、JPG和DIB，我们在这里使用PNG：

```java
int inputImagePictureID1 = workbook.addPicture(inputImageBytes1, Workbook.PICTURE_TYPE_PNG);
int inputImagePictureID2 = workbook.addPicture(inputImageBytes2, Workbook.PICTURE_TYPE_PNG);
```

完成此步骤后，我们将获得用于创建绘图对象的每张图片的索引。

### 3.4 创建Drawing容器

绘图父类是所有形状的顶层容器，它将返回一个Drawing接口-在我们的例子中是XSSFDrawing对象，我们将使用这个对象来创建图片，并将其放入我们定义的单元格中。

让我们来创建绘图族长：

```java
XSSFDrawing drawing = (XSSFDrawing) sheet.createDrawingPatriarch();
```

## 4. 在单元格中添加图像

现在，我们准备将图像添加到我们的单元格中。

### 4.1 创建锚点对象

首先，我们将创建一个客户端锚点对象，该对象将附加到Excel工作表，用于设置图像在Excel工作表中的位置，它锚定在左上角和右下角的单元格上。

我们将创建两个锚点对象，每个图像一个：

```java
XSSFClientAnchor ironManAnchor = new XSSFClientAnchor();
XSSFClientAnchor spiderManAnchor = new XSSFClientAnchor();
```

接下来，我们需要指定图像与锚对象的相对位置。

我们将第一张图片放在单元格B1中：

```java
ironManAnchor.setCol1(1); // Sets the column (0 based) of the first cell.
ironManAnchor.setCol2(2); // Sets the column (0 based) of the Second cell.
ironManAnchor.setRow1(0); // Sets the row (0 based) of the first cell.
ironManAnchor.setRow2(1); // Sets the row (0 based) of the Second cell.
```

以同样的方式，我们将第二张图像放在单元格B2中：

```java
spiderManAnchor.setCol1(1);
spiderManAnchor.setCol2(2);
spiderManAnchor.setRow1(1);
spiderManAnchor.setRow2(2);
```

### 4.2 向绘图容器添加锚对象和图片索引

现在，让我们在绘图类中调用createPicture方法来添加图像，**我们将使用之前创建的锚点对象和图像的索引**：

```java
drawing.createPicture(ironManAnchor, inputImagePictureID1);
drawing.createPicture(spiderManAnchor, inputImagePictureID2);
```

## 5. 保存工作簿

在保存之前，让我们使用autoSizeColumn确保单元格足够宽，可以容纳我们添加的图片：

```java
for (int i = 0; i < 3; i++) {
    sheet.autoSizeColumn(i);
}
```

最后，让我们保存工作簿：

```java
try (FileOutputStream saveExcel = new FileOutputStream("target/tuyucheng-apachepoi.xlsx")) {
    workbook.write(saveExcel);
}
```

生成的Excel表应如下所示：

![](/assets/images/2025/apache/javaaddimageexcel01.png)

## 6. 总结

在本文中，我们学习了如何使用Apache POI库在Java中将图像添加到Excel工作表的单元格。

我们需要加载图像，将其转换为字节，附加到工作表，然后使用绘图工具将图像定位到正确的单元格中。最后，我们能够调整列的大小并保存工作簿。