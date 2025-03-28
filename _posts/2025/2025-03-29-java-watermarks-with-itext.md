---
layout: post
title:  在Java中通过iText使用水印
category: libraries
copyright: libraries
excerpt: iText
---

## 1. 概述

**[iText](https://www.baeldung.com/java-pdf-creation) PDF是一个用于创建和处理PDF文件的Java库，水印有助于保护机密信息**。

在本教程中，我们将通过创建带有水印的新PDF文件来探索iText PDF库，我们还将向现有PDF文件添加水印。

## 2. Maven依赖

在本教程中，我们将使用Maven来管理依赖，我们需要[iText](https://mvnrepository.com/artifact/com.itextpdf/itext7-core)依赖才能开始使用iText PDF库，此外，我们需要[AssertJ](https://mvnrepository.com/artifact/org.assertj/assertj-core)依赖来进行测试。我们将两个依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>com.itextpdf</groupId>
    <artifactId>itext7-core</artifactId>
    <version>7.2.4</version>
    <type>pom</type>
</dependency>
<dependency>
    <groupId>org.assertj</groupId>
    <artifactId>assertj-core</artifactId>
    <version>3.25.3</version>
    <scope>test</scope>
</dependency>
```

## 3. 水印

水印有助于在文档或图像文件上叠加或隐藏文本或徽标，**它对于版权保护、数字产品营销、防止假冒等至关重要**。

在本教程中，我们将在生成的PDF中添加机密水印，水印将防止未经授权使用我们生成的PDF：

![](/assets/images/2025/libraries/javawatermarkswithitext01.png)

## 4. 使用iText生成PDF

在本文中，让我们整理一个故事，并使用iText PDF库将我们的故事转换为PDF格式。我们将编写一个简单的程序StoryTime。首先，我们将声明两个String类型的变量，我们将把我们的故事存储在声明的变量中：

```java
public class StoryTime {
    String aliceStory = "I am ...";
    String paulStory = "I am Paul ..";
}
```

为了简单起见，我们将缩短字符串值。然后，让我们声明一个字符串类型的变量，它将存储生成的PDF的输出路径：

```java
public static final String OUTPUT_DIR = "output/alice.pdf";
```

最后，让我们创建一个包含程序逻辑的方法，我们将创建一个PdfWriter实例来指定我们的输出路径和名称。

接下来，我们将创建一个PdfDocument实例来处理我们的PDF文件。为了将字符串值添加到PDF文档，我们将创建一个新的Document实例：

```java
public void createPdf(String output) throws IOException {
    PdfWriter writer = new PdfWriter(output);
    PdfDocument pdf = new PdfDocument(writer);
    try (Document document = new Document(pdf, PageSize.A4, false)) {
        document.add(new Paragraph(aliceSpeech)
                .setFont(PdfFontFactory.createFont(StandardFonts.TIMES_ROMAN)));
        document.add(new Paragraph(paulSpeech)
                .setFont(PdfFontFactory.createFont(StandardFonts.TIMES_ROMAN)));
        document.close();
    }
}
```

我们的方法将生成一个新的PDF文件并将其存储在OUTPUT_DIR中。 

## 5. 为生成的PDF添加水印

在上一节中，我们使用iText PDF库生成了一个PDF文件，**首先生成PDF有助于了解页面大小、旋转和页数，这有助于有效地添加水印**。让我们为我们的简单程序添加更多逻辑，我们的程序将为生成的PDF添加水印。 

首先，让我们创建一个方法来指定水印的属性。我们将设置水印的Font、fontSize和Opacity：

```java
public Paragraph createWatermarkParagraph(String watermark) throws IOException {
    PdfFont font = PdfFontFactory.createFont(StandardFonts.HELVETICA);
    Text text = new Text(watermark);
    text.setFont(font);
    text.setFontSize(56);
    text.setOpacity(0.5f);
    return new Paragraph(text);
}
```

接下来，让我们创建一个包含向PDF文档添加水印逻辑的方法，该方法将以Document、Paragraph和offset作为参数。我们将计算放置水印段落的位置和旋转：

```java
public void addWatermarkToGeneratedPDF(Document document, int pageIndex, 
  Paragraph paragraph, float verticalOffset) {
    
    PdfPage pdfPage = document.getPdfDocument().getPage(pageIndex);
    PageSize pageSize = (PageSize) pdfPage.getPageSizeWithRotation();
    float x = (pageSize.getLeft() + pageSize.getRight()) / 2;
    float y = (pageSize.getTop() + pageSize.getBottom()) / 2;
    float xOffset = 100f / 2;
    float rotationInRadians = (float) (PI / 180 * 45f);
    document.showTextAligned(paragraph, x - xOffset, y + verticalOffset, pageIndex, CENTER, TOP, rotationInRadians);
}
```

我们通过调用showTextAligned()方法将水印段落添加到文档中。接下来，让我们编写一个生成新PDF并添加水印的方法，我们将调用createWatermarkParagraph()方法和addWatermarkToGeneratedPDF()方法：

```java
public void createNewPDF() throws IOException {
    StoryTime storyTime = new StoryTime();
    String waterMark = "CONFIDENTIAL";
    PdfWriter writer = new PdfWriter(storyTime.OUTPUT_FILE);
    PdfDocument pdf = new PdfDocument(writer);

    try (Document document = new Document(pdf)) {
        document.add(new Paragraph(storyTime.alice)
                .setFont(PdfFontFactory.createFont(StandardFonts.TIMES_ROMAN)));
        document.add(new Paragraph(storyTime.paul));
        Paragrapgh paragraph = storyTime.createWatermarkParagraph(waterMark);
        for (int i = 1; i <= document.getPdfDocument().getNumberOfPages(); i++) {
            storyTime.addWatermarkToGeneratedPDF(document, i, paragraph, 0f);
        }
    }
}
```

最后我们来写一个单元测试来验证水印的存在：

```java
@Test
public void givenNewTexts_whenGeneratingNewPDFWithIText() throws IOException {
    StoryTime storyTime = new StoryTime();
    String waterMark = "CONFIDENTIAL";
    LocationTextExtractionStrategy extStrategy = new LocationTextExtractionStrategy();
    try (PdfDocument pdfDocument = new PdfDocument(new PdfReader(storyTime.OUTPUT_FILE))) {
        for (int i = 1; i <= pdfDocument.getNumberOfPages(); i++) {
            String textFromPage = getTextFromPage(pdfDocument.getPage(i), extStrategy);
            assertThat(textFromPage).contains(waterMark);
        }
    }
}
```

我们的测试验证了我们生成的PDF中是否存在水印。

## 6. 向现有PDF添加水印

**iText PDF库使向现有PDF添加水印变得容易**，我们首先将PDF文档加载到我们的程序中，然后使用iText库来操作我们现有的PDF。

首先，我们需要创建一个方法来添加水印段落。由于我们在上一节中创建了一个方法，因此我们也可以在这里使用它。

接下来，我们将创建一个方法，其中包含可帮助我们向现有PDF添加水印的逻辑。该方法将接收Document、Paragraph、PdfExtGState、pageIndex和offSet作为参数。在该方法中，我们将创建一个新的PdfCanvas实例，以将数据写入我们的PDF内容流。 

然后，我们将计算PDF上水印的位置和旋转，我们将刷新文档并释放状态以提高性能：

```java
public void addWatermarkToExistingPDF(Document document, int pageIndex, Paragraph paragraph, PdfExtGState graphicState, float verticalOffset) {
    PdfDocument pdfDocument = document.getPdfDocument();
    PdfPage pdfPage = pdfDocument.getPage(pageIndex);
    PageSize pageSize = (PageSize) pdfPage.getPageSizeWithRotation();
    float x = (pageSize.getLeft() + pageSize.getRight()) / 2;
    float y = (pageSize.getTop() + pageSize.getBottom()) / 2;

    PdfCanvas over = new PdfCanvas(pdfDocument.getPage(pageIndex));
    over.saveState();
    over.setExtGState(graphicState);
    float xOffset = 14 / 2;
    float rotationInRadians = (float) (PI / 180 * 45f);

    document.showTextAligned(paragraph, x - xOffset, y + verticalOffset,
            pageIndex, CENTER, TOP, rotationInRadians);
    document.flush();
    over.restoreState();
    over.release();
}
```

最后，让我们编写一个方法来向现有PDF添加水印。我们将调用createWatermarkParagraph()来添加水印段落。此外，我们将调用addWatermarkToExistingPDF()来处理向页面添加水印的任务：

```java
public void addWatermarkToExistingPdf() throws IOException {
    StoryTime storyTime = new StoryTime();
    String outputPdf = "output/aliceNew.pdf";
    String watermark = "CONFIDENTIAL";

    try (PdfDocument pdfDocument = new PdfDocument(new PdfReader("output/alice.pdf"),
            new PdfWriter(outputPdf))) {
        Document document = new Document(pdfDocument);
        Paragraph paragraph = storyTime.createWatermarkParagraph(watermark);
        PdfExtGState transparentGraphicState = new PdfExtGState().setFillOpacity(0.5f);
        for (int i = 1; i <= document.getPdfDocument().getNumberOfPages(); i++) {
            storyTime.addWatermarkToExistingPage(document, i, paragraph,
                    transparentGraphicState, 0f);
        }
    }
}
```

让我们编写一个单元测试来验证水印的存在：

```java
@Test
public void givenAnExistingPDF_whenManipulatedPDFWithITextmark() throws IOException {
    StoryTime storyTime = new StoryTime();
    String outputPdf = "output/aliceupdated.pdf";
    String watermark = "CONFIDENTIAL";

    LocationTextExtractionStrategy extStrategy = new LocationTextExtractionStrategy();
    try (PdfDocument pdfDocument = new PdfDocument(new PdfReader(outputPdf))) {
        for (int i = 1; i <= pdfDocument.getNumberOfPages(); i++) {
            String textFromPage = getTextFromPage(pdfDocument.getPage(i), extStrategy);
            assertThat(textFromPage).contains(watermark);
        }
    }
}
```

我们的测试验证了现有PDF中是否存在水印。

## 7. 总结

在本教程中，我们通过生成新的PDF探索了iText PDF库。我们在生成的PDF中添加了水印，随后又在现有PDF中添加了水印。