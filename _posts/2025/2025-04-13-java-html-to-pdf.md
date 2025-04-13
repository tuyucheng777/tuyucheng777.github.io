---
layout: post
title:  使用OpenPDF将HTML转换为PDF
category: libraries
copyright: libraries
excerpt: OpenPDF
---

## 1. 概述

在本快速教程中，**我们将研究如何使用Java中的OpenPDF以编程方式将HTML文件转换为PDF格式**。

## 2. OpenPDF

OpenPDF是一个免费的Java库，用于创建和编辑PDF文件，它是iText程序的一个分支。事实上，在版本5之前，使用OpenPDF生成PDF的代码几乎与iText API完全相同；它是一个维护良好的Java生成PDF的解决方案。

## 3. 使用Flying Saucer转换

Flying Saucer是一个Java库，它允许我们使用CSS 2.1呈现格式良好的XML(或XHTML)以进行样式和格式化，并生成PDF、图片和面板的输出。

### 3.1 Maven依赖

```xml
<dependency>
    <groupId>org.jsoup</groupId>
    <artifactId>jsoup</artifactId>
    <version>1.17.2</version>
</dependency>
<dependency>
    <groupId>org.xhtmlrenderer</groupId>
    <artifactId>flying-saucer-pdf-openpdf</artifactId>
    <version>9.5.1</version>
</dependency>
```

我们将使用[jsoup](https://mvnrepository.com/artifact/org.jsoup/jsoup)库来解析HTML文件、输入流、URL甚至字符串，它提供DOM(文档对象模型)遍历功能、CSS以及类似jQuery的选择器，用于从HTML中提取数据。

**[flying-saucer-pdf-openpdf](https://mvnrepository.com/artifact/org.xhtmlrenderer/flying-saucer-pdf-openpdf)库接收HTML文件的XML表示作为输入，应用CSS格式和样式**，并输出PDF。

### 3.2 HTML转PDF

在本教程中，我们将尝试涵盖你在使用Flying Saucer和OpenPDF将HTML转换为PDF时可能遇到的简单示例，例如HTML中的图像和样式；我们还将讨论如何自定义代码以接收外部样式、图像和字体。

让我们看一下示例HTML代码：

```xml
<html>
    <head>
        <style>
            .center_div {
                border: 1px solid gray;
                margin-left: auto;
                margin-right: auto;
                width: 90%;
                background-color: #d0f0f6;
                text-align: left;
                padding: 8px;
            }
        </style>
        <link href="style.css" rel="stylesheet">
    </head>
    <body>
        <div class="center_div">
            <h1>Hello Tuyucheng!</h1>
            <img src="Java_logo.png">
            <div class="myclass">
                <p>This is the tutorial to convert html to pdf.</p>
            </div>
        </div>
    </body>
</html>
```

要将HTML转换为PDF，我们首先从定义的位置读取HTML文件：

```java
File inputHTML = new File(HTML);
```

下一步，我们将使用[jsoup](https://www.baeldung.com/java-with-jsoup)将上述HTML文件转换为jsoup文档以呈现XHTML。

下面给出的是XHTML输出：

```java
Document document = Jsoup.parse(inputHTML, "UTF-8");
document.outputSettings().syntax(Document.OutputSettings.Syntax.xml);
return document;
```

现在，作为最后一步，让我们从上一步生成的XHTML文档创建一个PDF，ITextRenderer将获取此XHTML文档并创建一个输出PDF文件。请注意，**我们将代码包装在[try-with-resources](https://www.baeldung.com/java-try-with-resources)块中，以确保输出流已关闭**：

```java
try (OutputStream outputStream = new FileOutputStream(outputPdf)) {
    ITextRenderer renderer = new ITextRenderer();
    SharedContext sharedContext = renderer.getSharedContext();
    sharedContext.setPrint(true);
    sharedContext.setInteractive(false);
    renderer.setDocumentFromString(xhtml.html());
    renderer.layout();
    renderer.createPDF(outputStream);
}
```

### 3.3 定制外部样式

我们可以将HTML输入文档中使用的其他字体注册到ITextRenderer，以便它可以在生成PDF时包含它们：

```java
renderer.getFontResolver().addFont(getClass().getClassLoader().getResource("fonts/PRISTINA.ttf").toString(), true);
```

ITextRenderer可能需要注册相对URL来访问外部样式：

```java
String baseUrl = FileSystems.getDefault()
    .getPath("src/main/resources/")
    .toUri().toURL().toString();
renderer.setDocumentFromString(xhtml, baseUrl);
```

我们可以通过实现ReplacedElementFactory来定制与图像相关的属性：

```java
public ReplacedElement createReplacedElement(LayoutContext lc, BlockBox box, UserAgentCallback uac, int cssWidth, int cssHeight) {
    Element e = box.getElement();
    String nodeName = e.getNodeName();
    if (nodeName.equals("img")) {
        String imagePath = e.getAttribute("src");
        try {
            InputStream input = new FileInputStream("src/main/resources/"+imagePath);
            byte[] bytes = IOUtils.toByteArray(input);
            Image image = Image.getInstance(bytes);
            FSImage fsImage = new ITextFSImage(image);
            if (cssWidth != -1 || cssHeight != -1) {
                fsImage.scale(cssWidth, cssHeight);
            } else {
                fsImage.scale(2000, 1000);
            }
            return new ITextImageElement(fsImage);
        } catch (Exception e1) {
            e1.printStackTrace();
        }
    }
    return null;
}
```

注意：上述代码将基本路径作为图像路径的前缀，并在未提供时设置默认图像大小。

然后，我们可以将自定义ReplacedElementFactory添加到SharedContext：

```java
sharedContext.setReplacedElementFactory(new CustomElementFactoryImpl());
```

## 4. 使用Open HTML进行转换

Open HTML To PDF是一个Java库，它使用CSS 2.1(及更高版本的标准)进行布局和格式化，将格式良好的XML/XHTML(甚至一些HTML5)输出为PDF或图片。

### 4.1 Maven依赖

除了上面显示的jsoup库之外，我们还需要在pom.xml文件中添加几个Open HTML To PDF库：

```xml
<dependency>
    <groupId>com.openhtmltopdf</groupId>
    <artifactId>openhtmltopdf-core</artifactId>
    <version>1.0.10</version>
</dependency>
<dependency>
    <groupId>com.openhtmltopdf</groupId>
    <artifactId>openhtmltopdf-pdfbox</artifactId>
    <version>1.0.10</version>
</dependency>
```

**库[openhtmltopdf-core](https://mvnrepository.com/artifact/com.openhtmltopdf/openhtmltopdf-core)呈现格式良好的XML/XHTML，而[openhtmltopdf-pdfbox](https://mvnrepository.com/artifact/com.openhtmltopdf/openhtmltopdf-pdfbox)根据呈现的XHTML表示生成PDF文档**。

### 4.2 HTML转PDF

在这个程序中，为了使用Open HTML将HTML转换为PDF，我们将使用3.2节中提到的相同HTML。首先，我们将HTML文件转换为jsoup文档，就像我们在前面的示例中展示的那样。

最后一步，为了从XHTML文档创建PDF，**PdfRendererBuilder将获取此XHTML文档并创建一个PDF作为输出文件**。同样，我们使用try-with-resources来包装我们的逻辑：


```java
try (OutputStream os = new FileOutputStream(outputPdf)) {
    PdfRendererBuilder builder = new PdfRendererBuilder();
    builder.withUri(outputPdf);
    builder.toStream(os);
    builder.withW3cDocument(new W3CDom().fromJsoup(doc), "/");
    builder.run();
}
```

### 4.3 定制外部样式

我们可以将HTML输入文档中使用的其他字体注册到PdfRendererBuilder，以便它可以将它们包含在PDF中：

```java
builder.useFont(new File(getClass().getClassLoader().getResource("fonts/PRISTINA.ttf").getFile()), "PRISTINA");
```

PdfRendererBuilder库可能还需要注册相对URL来访问外部样式，类似于我们之前的示例：

```java
String baseUrl = FileSystems.getDefault()
    .getPath("src/main/resources/")
    .toUri().toURL().toString();
builder.withW3cDocument(new W3CDom().fromJsoup(doc), baseUrl);
```

## 5. 总结

在本文中，我们学习了如何使用Flying Saucer和Open HTML将HTML转换为PDF。我们还讨论了如何注册外部字体、样式和自定义设置。