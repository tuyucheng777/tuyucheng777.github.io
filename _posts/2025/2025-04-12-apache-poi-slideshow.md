---
layout: post
title:  使用Java创建MS PowerPoint演示文稿
category: apache
copyright: apache
excerpt: Apache POI
---

## 1. 简介

在本文中，我们将了解如何使用[Apache POI](https://poi.apache.org/)创建演示文稿。

这个库使我们能够创建PowerPoint演示文稿、读取现有演示文稿并更改其内容。

## 2. Maven依赖

首先，我们需要在pom.xml中添加以下依赖：

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
```

这[两个库](https://mvnrepository.com/artifact/org.apache.poi/poi-ooxml)的最新版本都可以从Maven Central下载。

## 3. Apache POI

**[Apache POI](https://poi.apache.org/)库支持.ppt和.pptx文件**，并为Powerpoint'97(-2007)文件格式提供HSLF实现，为PowerPoint 2007 OOXML文件格式提供XSLF实现。

由于这两种实现不存在通用接口，因此我们必须**记住在使用较新的.pptx文件格式时使用XMLSlideShow、XSLFSlide和XSLFTextShape类**。

而且，当需要使用较旧的.ppt格式时，请使用HSLFSlideShow、HSLFSlide和HSLFTextParagraph类。

我们将在示例中使用新的.pptx文件格式，我们要做的第一件事是创建一个新的演示文稿，向其中添加幻灯片(可能使用预定义的布局)并保存它。

一旦这些操作明确了，我们就可以开始处理图像、文本和表格。

### 3.1 创建新演示文稿

让我们首先创建新的演示文稿：

```java
XMLSlideShow ppt = new XMLSlideShow();
ppt.createSlide();
```

### 3.2 添加新幻灯片

在向演示文稿中添加新幻灯片时，我们也可以选择从预定义的布局创建它。为此，我们首先必须检索包含布局的XSLFSlideMaster(第一个是默认母版)：

```java
XSLFSlideMaster defaultMaster = ppt.getSlideMasters().get(0);
```

现在，我们可以检索XSLFSlideLayout并在创建新幻灯片时使用它：

```java
XSLFSlideLayout layout = defaultMaster.getLayout(SlideLayout.TITLE_AND_CONTENT);
XSLFSlide slide = ppt.createSlide(layout);
```

让我们看看如何在模板中填充占位符：

```java
XSLFTextShape titleShape = slide.getPlaceholder(0);
XSLFTextShape contentShape = slide.getPlaceholder(1);
```

请记住，每个模板都有其占位符，即XSLFAutoShape子类的实例，不同模板之间的占位符数量可能不同。

让我们看看如何快速从幻灯片中检索所有占位符：

```java
for (XSLFShape shape : slide.getShapes()) {
    if (shape instanceof XSLFAutoShape) {
        // this is a template placeholder
    }
}
```

### 3.3 保存演示文稿

创建幻灯片后，下一步就是保存它：

```java
FileOutputStream out = new FileOutputStream("powerpoint.pptx");
ppt.write(out);
out.close();
```

## 4. 使用对象

现在我们已经了解了如何创建新的演示文稿、向其中添加幻灯片(使用或不使用预定义模板)并保存它，我们可以开始添加文本、图像、链接和表格。

让我们从文本开始。

### 4.1 文本

在处理演示文稿中的文本时，就像在MS PowerPoint中一样，我们必须在幻灯片内创建文本框，添加段落，然后将文本添加到段落中：

```java
XSLFTextBox shape = slide.createTextBox();
XSLFTextParagraph p = shape.addNewTextParagraph();
XSLFTextRun r = p.addNewTextRun();
r.setText("Baeldung");
r.setFontColor(Color.green);
r.setFontSize(24.);
```

配置XSLFTextRun时，可以通过选择字体系列以及文本是否应为粗体、斜体或下划线来定制其样式。

### 4.2 超链接

向演示文稿添加文本时，添加超链接有时很有用。

一旦我们创建了XSLFTextRun对象，我们现在就可以添加一个链接：

```java
XSLFHyperlink link = r.createHyperlink();
link.setAddress("http://www.tuyucheng.com");
```

### 4.3 图像

我们也可以添加图像：

```java
byte[] pictureData = IOUtils.toByteArray(new FileInputStream("logo-leaf.png"));

XSLFPictureData pd = ppt.addPicture(pictureData, PictureData.PictureType.PNG);
XSLFPictureShape picture = slide.createPicture(pd);
```

但是，**如果没有正确的配置，图像将被放置在幻灯片的左上角**。为了正确放置它，我们必须配置它的锚点：

```java
picture.setAnchor(new Rectangle(320, 230, 100, 92));
```

XSLFPictureShape接收一个矩形作为锚点，这允许我们使用前两个参数配置x/y坐标，使用后两个参数配置图像的宽度/高度。

### 4.4 列表

演示文稿中的文本通常以列表的形式表示，无论是否编号。

现在让我们定义一个要点列表：

```java
XSLFTextShape content = slide.getPlaceholder(1);
XSLFTextParagraph p1 = content.addNewTextParagraph();
p1.setIndentLevel(0);
p1.setBullet(true);
r1 = p1.addNewTextRun();
r1.setText("Bullet");
```

类似地，我们可以定义一个编号列表：

```java
XSLFTextParagraph p2 = content.addNewTextParagraph();
p2.setBulletAutoNumber(AutoNumberingScheme.alphaLcParenRight, 1);
p2.setIndentLevel(1);
XSLFTextRun r2 = p2.addNewTextRun();
r2.setText("Numbered List Item - 1");
```

如果我们处理多个列表，定义indentLevel以实现项目的正确缩进始终很重要。

### 4.5 表格

表格是演示文稿中的另一个关键对象，当我们想要显示数据时很有用。

让我们首先创建一个表：

```java
XSLFTable tbl = slide.createTable();
tbl.setAnchor(new Rectangle(50, 50, 450, 300));
```

现在，我们可以添加一个标题：

```java
int numColumns = 3;
XSLFTableRow headerRow = tbl.addRow();
headerRow.setHeight(50);

for (int i = 0; i < numColumns; i++) {
    XSLFTableCell th = headerRow.addCell();
    XSLFTextParagraph p = th.addNewTextParagraph();
    p.setTextAlign(TextParagraph.TextAlign.CENTER);
    XSLFTextRun r = p.addNewTextRun();
    r.setText("Header " + (i + 1));
    tbl.setColumnWidth(i, 150);
}
```

标题完成后，我们可以向表中添加行和单元格来显示数据：

```java
for (int rownum = 1; rownum < numRows; rownum++) {
    XSLFTableRow tr = tbl.addRow();
    tr.setHeight(50);

    for (int i = 0; i < numColumns; i++) {
        XSLFTableCell cell = tr.addCell();
        XSLFTextParagraph p = cell.addNewTextParagraph();
        XSLFTextRun r = p.addNewTextRun();
        r.setText("Cell " + (i * rownum + 1));
    }
}
```

使用表格时，需要注意的是，可以自定义每个单元格的边框和背景。

## 5. 改变演示文稿

在制作幻灯片时，我们并不总是需要创建一个新的幻灯片，而是需要修改现有的幻灯片。

让我们看一下我们在上一节中创建的那个，然后我们可以开始修改它：

![](/assets/images/2025/apache/apachepoislideshow01.png)

### 5.1 读取演示文稿

读取演示文稿非常简单，可以使用接收FileInputStream的XMLSlideShow重载构造函数来完成：

```java
XMLSlideShow ppt = new XMLSlideShow(new FileInputStream("slideshow.pptx"));
```

### 5.2 更改幻灯片顺序

在向演示文稿中添加幻灯片时，最好将它们按正确的顺序排列，以确保幻灯片的正确流动。

如果没有发生这种情况，可以重新排列幻灯片的顺序，让我们看看如何将第四张幻灯片移到第二张幻灯片：

```java
List<XSLFSlide> slides = ppt.getSlides();

XSLFSlide slide = slides.get(3);
ppt.setSlideOrder(slide, 1);
```

### 5.3 删除幻灯片

还可以从演示文稿中删除幻灯片。

让我们看看如何删除第四张幻灯片：

```java
ppt.removeSlide(3);
```

## 6. 总结

本快速教程从Java角度说明了如何使用Apache POI API读取和写入PowerPoint文件。