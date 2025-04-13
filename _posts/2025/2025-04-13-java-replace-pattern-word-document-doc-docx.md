---
layout: post
title:  使用Java替换文档模板中的变量
category: apache
copyright: apache
excerpt: Apache POI
---

## 1. 概述

在本教程中，我们将替换Word文档中不同位置的图案，我们将处理.doc和.docx文件。

## 2. Apache POI库

**[Apache POI库](https://www.baeldung.com/java-microsoft-word-with-apache-poi)提供了Java API，用于处理Microsoft Office应用程序使用的各种文件格式**，例如Excel电子表格、Word文档和PowerPoint演示文稿；它允许以编程方式读取、写入和修改此类文件。

要编辑.docx文件，我们将最新版本的[poi-ooxml](https://mvnrepository.com/artifact/org.apache.poi/poi-ooxml)添加到pom.xml中：

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
   .<artifactId>poi-ooxml</artifactId>
    <version>5.2.5</version>
</dependency>
```

此外，我们还需要最新版本的[poi-scratchpad](https://mvnrepository.com/artifact/org.apache.poi/poi-scratchpad)来处理.doc文件：

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-scratchpad</artifactId>
    <version>5.2.5</version>
</dependency>
```

## 3. 文件处理

我们需要创建示例文件，读取它们，替换文件中的部分文本，然后写入结果文件；我们先来讨论一下与文件处理相关的所有内容。

### 3.1 示例文件

让我们创建一个Word文档，**用Hello替换其中的Baeldung一词**。因此，我们会在文件的多个位置(尤其是在表格、文档的各个部分和段落中)写入Baeldung。我们还希望使用多种格式样式，包括在Word内部更改格式的一次。我们将使用同一份文档，一次保存为.doc文件，一次保存为.docx文件：

![](/assets/images/2025/apache/javareplacepatternworddocumentdocdocx01.png)

### 3.2 读取输入文件

首先，我们需要读取文件，**我们将它放在resources文件夹中，以便它在[类路径](https://www.baeldung.com/reading-file-in-java#Read)中可用**；这样，我们将获得一个InputStream。对于.doc文档，我们将基于此InputStream创建一个POIFSFileSystem对象。最后，我们可以检索要修改的HWPFDocument对象。我们将使用[try-with-resources](https://www.baeldung.com/java-try-with-resources)，以便自动关闭InputStream和POIFSFileSystem对象，但是，由于我们将对HWPFDocument进行修改，因此我们将手动关闭它：

```java
public void replaceText() throws IOException {
    String filePath = getClass().getClassLoader()
            .getResource("baeldung.doc")
            .getPath();
    try (InputStream inputStream = new FileInputStream(filePath); POIFSFileSystem fileSystem = new POIFSFileSystem(inputStream)) {
        HWPFDocument doc = new HWPFDocument(fileSystem);
        // replace text in doc and save changes
        doc.close();
    }
}
```

处理.docx文档时，它会更直接一些，因为我们可以直接从InputStream派生出XWPFDocument对象：

```java
public void replaceText() throws IOException {
    String filePath = getClass().getClassLoader()
            .getResource("baeldung.docx")
            .getPath();
    try (InputStream inputStream = new FileInputStream(filePath)) {
        XWPFDocument doc = new XWPFDocument(inputStream);
        // replace text in doc and save changes
        doc.close();
    }
}
```

### 3.3 写入输出文件

我们将把输出文档写入同一个文件，因此，修改后的文件将位于target文件夹中。**HWPFDocument和XWPFDocument类都公开了一个write()方法，用于将文档写入[OuputStream](https://www.baeldung.com/java-outputstream)**。例如，对于.doc文档，其操作步骤如下：

```java
private void saveFile(String filePath, HWPFDocument doc) throws IOException {
    try (FileOutputStream out = new FileOutputStream(filePath)) {
        doc.write(out);
    }
}
```

## 4. 替换.docx文档中的文本

让我们尝试替换.docx文档中出现的Baeldung一词，看看在此过程中我们面临哪些挑战。

### 4.1 简单的实现

我们已经将文档解析为XWPFDocument对象，XWPFDocument分为多个段落，文件核心中的段落可以直接访问。但是，要访问表格中的段落，需要循环遍历表格的所有行和单元格。方法replaceTextInParagraph()的编写留待以后再说，下面我们将如何将其重复应用于所有段落：

```java
private XWPFDocument replaceText(XWPFDocument doc, String originalText, String updatedText) {
    replaceTextInParagraphs(doc.getParagraphs(), originalText, updatedText);
    for (XWPFTable tbl : doc.getTables()) {
        for (XWPFTableRow row : tbl.getRows()) {
            for (XWPFTableCell cell : row.getTableCells()) {
                replaceTextInParagraphs(cell.getParagraphs(), originalText, updatedText);
            }
        }
    }
    return doc;
}

private void replaceTextInParagraphs(List<XWPFParagraph> paragraphs, String originalText, String updatedText) {
    paragraphs.forEach(paragraph -> replaceTextInParagraph(paragraph, originalText, updatedText));
}
```

在Apache POI中，段落被划分为XWPFRun对象。**首先，我们尝试遍历所有段落：如果在段落中检测到要替换的文本，我们将更新该段落的内容**：

```java
private void replaceTextInParagraph(XWPFParagraph paragraph, String originalText, String updatedText) {
    List<XWPFRun> runs = paragraph.getRuns();
    for (XWPFRun run : runs) {
        String text = run.getText(0);
        if (text != null && text.contains(originalText)) {
            String updatedRunText = text.replace(originalText, updatedText);
            run.setText(updatedRunText, 0);
        }
    }
}
```

最后，我们更新replaceText()以包含所有步骤：

```java
public void replaceText() throws IOException {
    String filePath = getClass().getClassLoader()
            .getResource("baeldung-copy.docx")
            .getPath();
    try (InputStream inputStream = new FileInputStream(filePath)) {
        XWPFDocument doc = new XWPFDocument(inputStream);
        doc = replaceText(doc, "Baeldung", "Hello");
        saveFile(filePath, doc);
        doc.close();
    }
}
```

现在让我们通过[单元测试](https://www.baeldung.com/java-unit-testing-best-practices#whatIsUnitTesting)来运行这段代码，我们可以看一下更新后的文档的截图：

![](/assets/images/2025/apache/javareplacepatternworddocumentdocdocx02.png)

### 4.2 局限性

正如我们在屏幕截图中看到的，大多数出现的“Baeldung”一词已被替换为“Hello”。但是，我们可以看到剩余的两个“Baeldung”。

现在让我们更深入地了解XWPFRun的含义，**每个XWPFRun代表一个连续的文本序列，具有一组通用的格式属性**。格式属性包括字体样式、大小、颜色、粗体、斜体、下划线等。每当格式发生变化时，就会产生一个新的XWPFRun，这就是为什么表格中具有各种格式的出现不会被替换的原因：它的内容分布在多个XWPFRun中。

然而，底部蓝色的Baeldung字符也没有被替换。事实上，Apache POI并不能保证具有相同格式属性的字符属于同一次运行。简而言之，对于最简单的情况，这种简单的实现已经足够好了。在这种情况下，这种解决方案是值得的，因为它不需要任何复杂的决策。但是，如果我们面临这种限制，就需要转向其他解决方案。

### 4.3 处理跨越多个字符的文本

为了简单起见，**我们做以下假设：当我们在段落中找到单词“Baeldung”时，丢失段落的格式是可以的**。因此，我们可以删除段落中所有现有的连续语句，并用一个新的语句替换它们。让我们重写replaceTextInParagraph()：

```java
private void replaceTextInParagraph(XWPFParagraph paragraph, String originalText, String updatedText) {
    String paragraphText = paragraph.getParagraphText();
    if (paragraphText.contains(originalText)) {
        String updatedParagraphText = paragraphText.replace(originalText, updatedText);
        while (paragraph.getRuns().size() > 0) {
            paragraph.removeRun(0);
        }
        XWPFRun newRun = paragraph.createRun();
        newRun.setText(updatedParagraphText);
    }
}
```

我们来看看结果文件：

![](/assets/images/2025/apache/javareplacepatternworddocumentdocdocx03.png)

正如预期的那样，现在所有出现的字符都被替换了。但是，大多数格式都丢失了，最后一种格式没有丢失。在这种情况下，Apache POI似乎以不同的方式处理格式属性。

最后需要注意的是，根据我们的用例，我们也可以决定保留原始段落的某些格式。然后，我们需要遍历所有段落，并根据需要保留或更新属性。

## 5. 替换.doc文档中的文本

对于doc文件来说，事情要简单得多，我们确实可以访问整个文档的Range对象，然后，**我们可以通过它的replaceText()方法修改该范围的内容**：

```java
private HWPFDocument replaceText(HWPFDocument doc, String originalText, String updatedText) {
    Range range = doc.getRange();
    range.replaceText(originalText, updatedText);
    return doc;
}
```

运行此代码将生成以下更新的文件：

![](/assets/images/2025/apache/javareplacepatternworddocumentdocdocx04.png)

我们可以看到，替换操作在整个文件中进行。我们还注意到，对于多次运行的文本，默认行为是保留第一次运行的格式。

## 6. 总结

在本文中，我们替换了Word文档中的一个模式。在.doc文档中，替换过程非常简单。然而，在.docx文档中，我们遇到了一些限制，我们展示了一个通过简化假设来克服这一限制的示例。