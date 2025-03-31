---
layout: post
title:  使用Tesseract进行光学字符识别
category: libraries
copyright: libraries
excerpt: Tesseract
---

## 1. 概述

随着人工智能和机器学习技术的进步，我们需要工具来[识别图像中的文本](https://www.baeldung.com/cs/images-find-text-blocks)。

在本教程中，我们将探索[光学字符识别](https://www.baeldung.com/cs/ocr)(OCR)引擎Tesseract，并提供一些图像到文本处理的示例。

## 2. Tesseract

**[Tesseract](https://github.com/tesseract-ocr/tesseract)是HP开发的一款开源OCR引擎，可识别100多种语言，同时支持表意文字和从右到左的语言**。此外，**我们还可以训练Tesseract识别其他语言**。

**它包含两个用于图像处理的OCR引擎**-一个[LSTM](https://www.baeldung.com/cs/rnns-transformers-nlp)(长短期记忆) OCR引擎和一个通过识别字符模式工作的传统OCR引擎。

OCR引擎使用[Leptonica库](https://github.com/DanBloomberg/leptonica)打开图像并支持各种输出格式，如纯文本、hOCR(OCR的HTML)、PDF和TSV。

## 3. 设置

Tesseract可在所有主流操作系统上下载/安装。

例如，如果我们使用的是macOS，我们可以使用[Homebrew](https://brew.sh/)安装OCR引擎：

```shell
brew install tesseract
```

我们会发现该包默认包含一组语言数据文件，如英语，以及方向和脚本检测(OSD)：

```text
==> Installing tesseract 
==> Downloading https://homebrew.bintray.com/bottles/tesseract-4.1.1.high_sierra.bottle.tar.gz
==> Pouring tesseract-4.1.1.high_sierra.bottle.tar.gz
==> Caveats
This formula contains only the "eng", "osd", and "snum" language data files.
If you need any other supported languages, run `brew install tesseract-lang`.
==> Summary
/usr/local/Cellar/tesseract/4.1.1: 65 files, 29.9MB
```

但是，我们可以安装tesseract-lang模块来支持其他语言：

```shell
brew install tesseract-lang
```

对于Linux，我们可以使用yum命令安装Tesseract ：

```shell
yum install tesseract
```

同样，让我们添加语言支持：

```shell
yum install tesseract-langpack-eng
yum install tesseract-langpack-spa
```

在这里，我们添加了英语和西班牙语的语言训练数据。

对于Windows，我们可以从[UB Mannheim的Tesseract](https://github.com/UB-Mannheim/tesseract/wiki)获取安装程序。

## 4. Tesseract命令行

### 4.1 运行

我们可以使用Tesseract命令行工具从图像中提取文本。

例如，让我们拍摄我们的网站快照：

![](/assets/images/2025/libraries/javaocrtesseract01.png)

然后，我们将运行tesseract命令来读取tuyucheng.png快照并将文本写入output.txt文件中：

```shell
tesseract tuyucheng.png output
```

output.txt文件将如下所示：

```text
a REST with Spring Learn Spring (new!)
The canonical reference for building a production
grade API with Spring.
From no experience to actually building stuff.
y
Java Weekly Reviews
```

我们可以观察到Tesseract并没有处理图像的全部内容，因为**输出的准确性取决于各种参数，例如图像质量、语言、页面分割、训练数据和用于图像处理的引擎**。

### 4.2 语言支持

默认情况下，OCR引擎在处理图像时使用英语。但是，我们可以使用-l参数声明语言：

我们来看看另一个包含多语言文本的例子：

![](/assets/images/2025/libraries/javaocrtesseract02.png)

首先，让我们用默认的英语处理图像：

```shell
tesseract multiLanguageText.png output
```

输出将如下所示：

```text
Der ,.schnelle” braune Fuchs springt
iiber den faulen Hund. Le renard brun
«rapide» saute par-dessus le chien
paresseux. La volpe marrone rapida
salta sopra il cane pigro. El zorro
marron rapido salta sobre el perro
perezoso. A raposa marrom rapida
salta sobre 0 cao preguicoso.
```

然后，我们用葡萄牙语处理图像：

```shell
tesseract multiLanguageText.png output -l por
```

因此，OCR引擎还会检测葡萄牙语字母：

```text
Der ,.schnelle” braune Fuchs springt
iber den faulen Hund. Le renard brun
«rapide» saute par-dessus le chien
paresseux. La volpe marrone rapida
salta sopra il cane pigro. El zorro
marrón rápido salta sobre el perro
perezoso. A raposa marrom rápida
salta sobre o cão preguiçoso.
```

类似地，我们可以声明多种语言的组合：

```shell
tesseract multiLanguageText.png output -l spa+por
```

在这里，OCR引擎将主要使用西班牙语，然后使用葡萄牙语进行图像处理。但是，输出可能会根据我们指定的语言顺序而有所不同。

### 4.3 页面分割模式

Tesseract支持各种页面分割模式，如OSD、自动页面分割和稀疏文本。

我们可以使用–psm参数来声明页面分割模式，各种模式的值为0到13：

```shell
tesseract multiLanguageText.png output --psm 1
```

在这里，通过定义值1，我们声明了使用OSD自动页面分割来进行图像处理。

让我们来看看所有支持的页面分割模式：

![](/assets/images/2025/libraries/javaocrtesseract03.png)

### 4.4 OCR引擎模式

类似地，我们可以在处理图像时使用各种引擎模式，如传统引擎和LSTM引擎。

为此，我们可以使用–oem参数，其值为0到3：

```shell
tesseract multiLanguageText.png output --oem 1
```

OCR引擎模式包括：

![](/assets/images/2025/libraries/javaocrtesseract04.png)

### 4.5 Tessdata

**Tesseract包含两组用于LSTM OCR引擎的训练数据-[最佳训练的LSTM模型](https://github.com/tesseract-ocr/tessdata_best)和[训练有素的LSTM模型的快速整数版本](https://github.com/tesseract-ocr/tessdata_fast)**。

前者提供更好的准确性，后者提供更好的图像处理速度。

此外，Tesseract还提供[组合训练数据](https://github.com/tesseract-ocr/tessdata)，支持传统OCR引擎和LSTM OCR引擎。

如果我们使用Legacy OCR引擎而不提供支持的训练数据，Tesseract将抛出错误：

```text
Error: Tesseract (legacy) engine requested, but components are not present in /usr/local/share/tessdata/eng.traineddata!!
Failed loading language 'eng'
Tesseract couldn't load any languages!
```

因此，我们应该下载所需的.traineddata文件并将它们保存在默认的tessdata位置或使用–tessdata-dir参数声明位置：

```shell
tesseract multiLanguageText.png output --tessdata-dir /image-processing/tessdata
```

### 4.6. 输出

我们可以声明一个参数来获取所需的输出格式。

例如，要获取可搜索的PDF输出：

```shell
tesseract multiLanguageText.png output pdf
```

这将在所提供图像上创建带有可搜索文本层(包含识别的文本)的output.pdf文件。

同样，对于hOCR输出：

```shell
tesseract multiLanguageText.png output hocr
```

另外，我们可以使用tesseract –help和tesseract –help-extra命令来获取有关tesseract命令行使用的更多信息。

## 5. Tess4J

Tess4J是Tesseract API的Java包装器，为JPEG、GIF、PNG和BMP等各种图像格式提供OCR支持。

首先，让我们将最新的[tess4j](https://mvnrepository.com/artifact/net.sourceforge.tess4j/tess4j) Maven依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>net.sourceforge.tess4j</groupId>
    <artifactId>tess4j</artifactId>
    <version>4.5.1</version>
</dependency>
```

然后，我们就可以使用tess4j提供的[Tesseract](http://tess4j.sourceforge.net/docs/docs-4.4/net/sourceforge/tess4j/Tesseract.html)类来处理图像了：

```java
File image = new File("src/main/resources/images/multiLanguageText.png");
Tesseract tesseract = new Tesseract();
tesseract.setDatapath("src/main/resources/tessdata");
tesseract.setLanguage("eng");
tesseract.setPageSegMode(1);
tesseract.setOcrEngineMode(1);
String result = tesseract.doOCR(image);
```

在这里，我们将datapath的值设置为包含osd.traineddata和eng.traineddata文件的目录位置。

最后，我们可以验证处理后的图像的字符串输出：

```java
Assert.assertTrue(result.contains("Der ,.schnelle” braune Fuchs springt"));
Assert.assertTrue(result.contains("salta sopra il cane pigro. El zorro"));
```

此外，我们可以使用setHocr方法来获取HTML输出：

```java
tesseract.setHocr(true);
```

默认情况下，该库会处理整个图像。但是，我们可以在调用doOCR方法时使用[java.awt.Rectangle](https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/java/awt/Rectangle.html)对象来处理图像的特定部分：

```java
result = tesseract.doOCR(imageFile, new Rectangle(1200, 200));
```

与Tess4J类似，我们可以使用[Tesseract Platform](https://github.com/bytedeco/javacpp-presets/tree/master/tesseract)将Tesseract集成到Java应用程序中。这是基于[JavaCPP Presets](https://github.com/bytedeco/javacpp-presets)库的Tesseract API的JNI包装器。

## 6. 总结

在本文中，我们通过一些图像处理示例探索了Tesseract OCR引擎。

首先，我们检查了tesseract命令行工具来处理图像，以及一组参数，如-l、–psm和–oem。

然后，我们探索了tess4j，一个用于将Tesseract集成到Java应用程序中的Java包装器。