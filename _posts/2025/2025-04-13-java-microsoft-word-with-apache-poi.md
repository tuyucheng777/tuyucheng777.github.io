---
layout: post
title:  使用Apache POI在Java中处理Microsoft Word
category: apache
copyright: apache
excerpt: Apache POI
---

## 1. 概述

[Apache POI](https://poi.apache.org/)是一个Java库，用于处理基于Office Open XML标准(OOXML)和Microsoft的OLE 2复合文档格式(OLE2)的各种文件格式。

本教程重点介绍[Apache POI](https://poi.apache.org/)对最常用的Office文件格式Microsoft Word的支持，逐步讲解格式化和生成MS Word文件以及如何解析该文件所需的步骤。

## 2. Maven依赖

Apache POI处理MS Word文件所需的唯一依赖是：

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.2.5</version>
</dependency>
```

可在[此处](https://mvnrepository.com/artifact/org.apache.poi/poi-ooxml)获取该工件的最新版本。

## 3. 准备

现在让我们看一看用于促进生成MS Word文件的一些元素。

### 3.1 资源文件

我们将收集3个文本文件的内容并将它们写入一个名为rest-with-spring.docx的MS Word文件中。

此外，logo-leaf.png文件用于将图像插入到该新文件中，所有这些文件都存在于类路径中，并由几个静态变量表示：

```java
public static String logo = "logo-leaf.png";
public static String paragraph1 = "poi-word-para1.txt";
public static String paragraph2 = "poi-word-para2.txt";
public static String paragraph3 = "poi-word-para3.txt";
public static String output = "rest-with-spring.docx";
```

### 3.2 辅助方法

主要方法由用于生成MS Word文件的逻辑组成，如下一节所述，它使用了一个辅助方法：

```java
public String convertTextFileToString(String fileName) {
    try (Stream<String> stream = Files.lines(Paths.get(ClassLoader.getSystemResource(fileName).toURI()))) {
        return stream.collect(Collectors.joining(" "));
    } catch (IOException | URISyntaxException e) {
        return null;
    }
}
```

此方法提取位于类路径下的文本文件的内容，该文件的名称是传入的String参数。然后，它将文件中的各行拼接起来并返回拼接的String。

## 4. MS Word文件生成

本节介绍如何格式化并生成Microsoft Word文件，在处理文件的任何部分之前，我们需要一个XWPFDocument实例：

```java
XWPFDocument document = new XWPFDocument();
```

### 4.1 格式化标题和副标题

为了创建标题，我们需要首先实例化XWPFParagraph类并在新对象上设置对齐方式：

```java
XWPFParagraph title = document.createParagraph();
title.setAlignment(ParagraphAlignment.CENTER);
```

段落的内容需要包装在XWPFRun对象中，我们可以配置此对象来设置文本值及其相关样式：

```java
XWPFRun titleRun = title.createRun();
titleRun.setText("Build Your REST API with Spring");
titleRun.setColor("009933");
titleRun.setBold(true);
titleRun.setFontFamily("Courier");
titleRun.setFontSize(20);
```

以类似的方式，我们创建一个包含字幕的XWPFParagraph实例：

```java
XWPFParagraph subTitle = document.createParagraph();
subTitle.setAlignment(ParagraphAlignment.CENTER);
```

让我们也格式化一下字幕：

```java
XWPFRun subTitleRun = subTitle.createRun();
subTitleRun.setText("from HTTP fundamentals to API Mastery");
subTitleRun.setColor("00CC44");
subTitleRun.setFontFamily("Courier");
subTitleRun.setFontSize(16);
subTitleRun.setTextPosition(20);
subTitleRun.setUnderline(UnderlinePatterns.DOT_DOT_DASH);
```

setTextPosition方法设置字幕和后续图像之间的距离，而setUnderline确定下划线模式。

请注意，我们对标题和副标题的内容进行了硬编码，因为这些语句太短，不值得使用辅助方法。

### 4.2 插入图像

图片也需要包裹在XWPFParagraph实例中，我们希望图片水平居中并放置在字幕下方，因此必须将以下代码片段放在上述代码下方：

```java
XWPFParagraph image = document.createParagraph();
image.setAlignment(ParagraphAlignment.CENTER);
```

以下是如何设置此图像与其下方文本之间的距离：

```java
XWPFRun imageRun = image.createRun();
imageRun.setTextPosition(20);
```

从类路径上的文件中获取图像，然后将其插入到具有指定大小的MS Word文件中：

```java
Path imagePath = Paths.get(ClassLoader.getSystemResource(logo).toURI());
imageRun.addPicture(Files.newInputStream(imagePath),
    XWPFDocument.PICTURE_TYPE_PNG, imagePath.getFileName().toString(),
    Units.toEMU(50), Units.toEMU(50));
```

### 4.3 段落格式

以下是我们如何使用来自poi-word-para1.txt文件的内容创建第一个段落：

```java
XWPFParagraph para1 = document.createParagraph();
para1.setAlignment(ParagraphAlignment.BOTH);
String string1 = convertTextFileToString(paragraph1);
XWPFRun para1Run = para1.createRun();
para1Run.setText(string1);
```

显然，段落的创建与标题或副标题的创建类似，唯一的区别是这里使用了辅助方法，而不是硬编码的字符串。

以类似的方式，我们可以使用文件poi-word-para2.txt和poi-word-para3.txt中的内容创建另外两个段落：

```java
XWPFParagraph para2 = document.createParagraph();
para2.setAlignment(ParagraphAlignment.RIGHT);
String string2 = convertTextFileToString(paragraph2);
XWPFRun para2Run = para2.createRun();
para2Run.setText(string2);
para2Run.setItalic(true);

XWPFParagraph para3 = document.createParagraph();
para3.setAlignment(ParagraphAlignment.LEFT);
String string3 = convertTextFileToString(paragraph3);
XWPFRun para3Run = para3.createRun();
para3Run.setText(string3);
```

这3个段落的创建几乎相同，除了对齐或斜体等一些样式之外。

### 4.4 生成MS Word文件

现在我们准备将Microsoft Word文件从document变量写入内存：

```java
FileOutputStream out = new FileOutputStream(output);
document.write(out);
out.close();
document.close();
```

本节中的所有代码片段都包装在名为handleSimpleDoc的方法中。

## 5. 解析和测试

本节概述了MS Word文件的解析和结果的验证。

### 5.1 准备

我们在测试类中声明一个静态字段：

```java
static WordDocument wordDocument;
```

该字段用于引用包含第3节和第4节中所示的所有代码片段的类的实例。

在解析和测试之前，我们需要初始化上面声明的静态变量，并通过调用handleSimpleDoc方法在当前工作目录中生成rest-with-spring.docx文件：

```java
@BeforeClass
public static void generateMSWordFile() throws Exception {
    WordTest.wordDocument = new WordDocument();
    wordDocument.handleSimpleDoc();
}
```

最后一步：解析MS Word文件并验证结果。

### 5.2 解析MS Word文件并验证

首先，我们从项目目录中给定的MS Word文件中提取内容，并将内容存储在XWPFParagraph列表中：

```java
Path msWordPath = Paths.get(WordDocument.output);
XWPFDocument document = new XWPFDocument(Files.newInputStream(msWordPath));
List<XWPFParagraph> paragraphs = document.getParagraphs();
document.close();
```

接下来我们确保标题的内容和样式和我们之前设置的一致：

```java
XWPFParagraph title = paragraphs.get(0);
XWPFRun titleRun = title.getRuns().get(0);
 
assertEquals("Build Your REST API with Spring", title.getText());
assertEquals("009933", titleRun.getColor());
assertTrue(titleRun.isBold());
assertEquals("Courier", titleRun.getFontFamily());
assertEquals(20, titleRun.getFontSize());
```

为了简单起见，我们只验证文件其他部分的内容，省略样式；其样式的验证与标题的验证类似：

```java
assertEquals("from HTTP fundamentals to API Mastery",
    paragraphs.get(1).getText());
assertEquals("What makes a good API?", paragraphs.get(3).getText());
assertEquals(wordDocument.convertTextFileToString
    (WordDocument.paragraph1), paragraphs.get(4).getText());
assertEquals(wordDocument.convertTextFileToString
    (WordDocument.paragraph2), paragraphs.get(5).getText());
assertEquals(wordDocument.convertTextFileToString
    (WordDocument.paragraph3), paragraphs.get(6).getText());
```

现在我们可以确信rest-with-spring.docx文件的创建已经成功。

## 6. 总结

本教程介绍了Apache POI对Microsoft Word格式的支持，它介绍了生成MS Word文件并验证其内容所需的步骤。