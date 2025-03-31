---
layout: post
title:  使用Java向图像添加文本
category: libraries
copyright: libraries
excerpt: ImageJ
---

## 1. 概述

有时我们需要在一张[图片](https://www.baeldung.com/java-images)或一组图片中添加一些文本，使用图片编辑工具手动完成此操作很容易。但是当我们想以相同的方式向大量图片添加相同的文本时，以编程方式执行此操作将非常有用。

在本快速教程中，我们将学习如何使用Java向图像添加一些文本。

## 2. 在图像中添加文本

要读取图像并添加一些文本，我们可以使用不同的类。在后续部分中，我们将看到几个选项。

### 2.1 ImagePlus和ImageProcessor

首先，让我们看看如何**使用[ImageJ库](https://imagej.nih.gov/ij/developer/api/index.html)中提供的[ImagePlus](https://imagej.net/ij/source/ij/ImagePlus)和[ImageProcessor](https://imagej.nih.gov/ij/developer/api/ij/ij/process/ImageProcessor.html)类**。要使用此库，我们需要在项目中包含此依赖：

```xml
<dependency>
    <groupId>net.imagej</groupId>
    <artifactId>ij</artifactId>
    <version>1.51h</version>
</dependency>
```

要读取图像，我们将使用openImage静态方法，此方法的结果将使用ImagePlus对象存储在内存中：

```java
ImagePlus image = IJ.openImage(path);
```

将图像加载到内存后，让我们使用ImageProcessor类向其中添加一些文本：

```java
Font font = new Font("Arial", Font.BOLD, 18);

ImageProcessor ip = image.getProcessor();
ip.setColor(Color.GREEN);
ip.setFont(font);
ip.drawString(text, 0, 20);
```

使用此代码，我们要做的是将指定的绿色文本添加到图像的左上角。请注意，我们使用drawString方法的第2个和第3个参数设置位置，这两个参数分别表示距离左侧和顶部的像素数。

### 2.2 BufferedImage和Graphics

接下来，我们将**了解如何使用[BufferedImage](https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/java/awt/image/BufferedImage.html)和[Graphics](https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/java/awt/Graphics.html#drawString(java.text.AttributedCharacterIterator,int,int))类实现相同的结果**。Java的标准版本包含这些类，因此不需要额外的库。

与我们使用ImageJ的openImage的方式相同，我们将使用ImageIO中可用的read方法：

```java
BufferedImage image = ImageIO.read(new File(path));
```

将图像加载到内存后，让我们使用Graphics类向其中添加一些文本：

```java
Font font = new Font("Arial", Font.BOLD, 18);

Graphics g = image.getGraphics();
g.setFont(font);
g.setColor(Color.GREEN);
g.drawString(text, 0, 20);
```

我们可以看到，这两种方法的使用方式非常相似。在本例中，方法drawString的第2个和第3个参数的指定方式与我们对ImageProcessor方法的指定方式相同。

### 2.3 基于AttributedCharacterIterator绘制

Graphics中的方法drawString允许我们使用[AttributedCharacterIterator](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/text/AttributedCharacterIterator.html)打印文本，这意味着我们可以使用具有某些关联属性的文本，而不是使用纯字符串。让我们看一个例子：

```java
Font font = new Font("Arial", Font.BOLD, 18);

AttributedString attributedText = new AttributedString(text);
attributedText.addAttribute(TextAttribute.FONT, font);
attributedText.addAttribute(TextAttribute.FOREGROUND, Color.GREEN);

Graphics g = image.getGraphics();
g.drawString(attributedText.getIterator(), 0, 20);
```

这种打印文本的方式使我们有机会将格式直接与String关联起来，这比每次我们想要更改格式时更改Graphics对象属性更清晰。

## 3. 文本对齐

现在我们已经了解了如何在图像的左上角添加简单文本，现在让我们看看如何在某些位置添加该文本。

### 3.1 居中文本

我们要处理的第一种对齐方式是使文本居中，为了动态设置我们想要写入文本的正确位置，我们需要弄清楚一些信息：

- 图片大小
- 字体大小

这些信息很容易获取。对于图像大小，可以通过BufferedImage对象的getWidth和getHeight方法访问这些数据。另一方面，要获取与字体大小相关的数据，我们需要使用FontMetrics对象。

让我们看一个例子，我们计算文本的正确位置并绘制它：

```java
Graphics g = image.getGraphics();

FontMetrics metrics = g.getFontMetrics(font);
int positionX = (image.getWidth() - metrics.stringWidth(text)) / 2;
int positionY = (image.getHeight() - metrics.getHeight()) / 2 + metrics.getAscent();

g.drawString(attributedText.getIterator(), positionX, positionY);
```

### 3.2 文本右下对齐

**我们将要看到的下一种对齐类型是右下角**。在这种情况下，我们需要动态获取正确的位置：

```java
int positionX = (image.getWidth() - metrics.stringWidth(text));
int positionY = (image.getHeight() - metrics.getHeight()) + metrics.getAscent();
```

### 3.3 文本位于左上角

最后，让我们看看**如何在左上角打印文本**：

```java
int positionX = 0;
int positionY = metrics.getAscent();
```

其余的排列可以从我们已经看到的三种排列中推断出来。

## 4. 根据图像调整文本大小

当我们在图片中绘制文本时，我们可能会发现文本超出了图片的大小。为了解决这个问题，我们必须根据图片大小调整所用字体的大小。

首先，我们需要使用基础字体获取文本的预期宽度和高度。为了实现这一点，我们将使用[FontMetrics](https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/java/awt/FontMetrics.html)、[GlyphVector](https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/java/awt/font/GlyphVector.html)和[Shape](https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/java/awt/Shape.html)类。

```java
FontMetrics ruler = graphics.getFontMetrics(baseFont);
GlyphVector vector = baseFont.createGlyphVector(ruler.getFontRenderContext(), text);
    
Shape outline = vector.getOutline(0, 0);
    
double expectedWidth = outline.getBounds().getWidth();
double expectedHeight = outline.getBounds().getHeight();
```

下一步是检查是否需要调整字体大小。为此，让我们比较一下文本的预期大小和图像的大小：

```java
boolean textFits = image.getWidth() >= expectedWidth && image.getHeight() >= expectedHeight;
```

最后，如果我们的文本不适合图像，我们必须减小字体大小，我们将使用方法deriveFont来实现这一点：

```java
double widthBasedFontSize = (baseFont.getSize2D()*image.getWidth())/expectedWidth;
double heightBasedFontSize = (baseFont.getSize2D()*image.getHeight())/expectedHeight;

double newFontSize = widthBasedFontSize < heightBasedFontSize ? widthBasedFontSize : heightBasedFontSize;
newFont = baseFont.deriveFont(baseFont.getStyle(), (float)newFontSize);
```

请注意，我们需要根据宽度和高度获取新的字体大小，并应用其中最小的一个。

## 5. 总结

在本文中，我们了解了如何使用不同的方法在图像中写入文本。

我们还学习了如何根据图像大小和字体属性动态获取想要打印文本的位置。

最后，我们看到了当文本的字体大小超出我们绘制图像的大小时，如何调整文本的字体大小。