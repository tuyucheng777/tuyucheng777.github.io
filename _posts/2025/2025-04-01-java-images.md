---
layout: post
title:  使用Java处理图像
category: libraries
copyright: libraries
excerpt: AWT、ImageJ、OpenIMAJ、TwelveMonkeys
---

## 1. 概述

在本教程中，我们将了解一些可用的图像处理库，并执行简单的图像处理操作-加载图像并在其上绘制形状。

我们将尝试AWT(和一些Swing)库、ImageJ、OpenIMAJ和TwelveMonkeys。

## 2. AWT

AWT是一个内置Java库，允许用户执行与显示相关的简单操作，例如创建窗口、定义按钮和监听器等，它还包括允许用户编辑图像的方法。它不需要安装，因为它随Java一起提供。

### 2.1 加载图像

首先，我们要从磁盘驱动器上保存的图片中创建一个BufferedImage对象：

```java
String imagePath = "path/to/your/image.jpg";
BufferedImage myPicture = ImageIO.read(new File(imagePath));
```

### 2.2 编辑图像

要在图像上绘制形状，我们必须使用与加载图像相关的Graphics对象，Graphics对象封装了执行基本渲染操作所需的属性。Graphics2D是一个扩展Graphics的类，它提供了对二维形状的更多控制。

在这个特殊情况下，我们需要Graphic2D扩展形状宽度以使其清晰可见，我们通过增加其stroke属性来实现这一点。然后我们设置颜色，并绘制一个矩形，使形状距离图像边框10像素：

```java
Graphics2D g = (Graphics2D) myPicture.getGraphics();
g.setStroke(new BasicStroke(3));
g.setColor(Color.BLUE);
g.drawRect(10, 10, myPicture.getWidth() - 20, myPicture.getHeight() - 20);
```

### 2.3 显示图像

现在我们已经在图像上绘制了一些东西，我们想显示它，我们可以使用Swing库对象来实现。首先，我们创建JLabel对象，它代表文本或/和图像的显示区域：

```java
JLabel picLabel = new JLabel(new ImageIcon(myPicture));
```

然后将我们的JLabel添加到JPanel中，我们可以将其视为基于Java的GUI的<div\></div\>：

```java
JPanel jPanel = new JPanel();
jPanel.add(picLabel);
```

最后，我们将所有内容添加到屏幕上显示的窗口JFrame中。我们必须设置大小，这样我们每次运行程序时就不必扩展此窗口：

```java
JFrame f = new JFrame();
f.setSize(new Dimension(myPicture.getWidth(), myPicture.getHeight()));
f.add(jPanel);
f.setVisible(true);
```

## 3. ImageJ

ImageJ是一款基于Java的图像处理软件，它有相当多的插件，可在[此处](https://imagej.net/ij/plugins/)获取。我们将仅使用API，因为我们想自己进行处理。

这是一个非常强大的库，比Swing和AWT更好，因为它的创建目的是图像处理而不是GUI操作。插件包含许多可免费使用的算法，当我们想学习图像处理并快速查看结果而不是解决IP算法下的数学和优化问题时，这是一件好事。

### 3.1 Maven依赖

要开始使用ImageJ，只需向项目的pom.xml文件添加依赖：

```xml
<dependency>
    <groupId>net.imagej</groupId>
    <artifactId>ij</artifactId>
    <version>1.51h</version>
</dependency>
```

你可以在[Maven仓库](https://mvnrepository.com/artifact/net.imagej/ij)中找到最新版本。

### 3.2 加载图像

要加载图像，你需要使用IJ类中的openImage()静态方法：

```java
ImagePlus imp = IJ.openImage("path/to/your/image.jpg");
```

### 3.3 编辑图像

要编辑图像，我们必须使用附加到ImagePlus对象的ImageProcessor对象的方法。可以将其视为AWT中的Graphics对象：

```java
ImageProcessor ip = imp.getProcessor();
ip.setColor(Color.BLUE);
ip.setLineWidth(4);
ip.drawRect(10, 10, imp.getWidth() - 20, imp.getHeight() - 20);
```

### 3.4 显示图像

你只需要调用ImagePlus对象的show()方法：

```java
imp.show();
```

## 4. OpenIMAJ

OpenIMAJ是一套Java库，不仅专注于计算机视觉和视频处理，还专注于机器学习、音频处理、与Hadoop协同工作等等。OpenIMAJ项目的所有部分都可以在[这里](http://openimaj.org/)找到，在“Modules”下，我们只需要图像处理部分。

### 4.1 Maven依赖

要开始使用OpenIMAJ，只需向项目的pom.xml文件添加依赖：

```xml
<dependency>
    <groupId>org.openimaj</groupId>
    <artifactId>core-image</artifactId>
    <version>1.3.5</version>
</dependency>
```

你可以在[这里](https://mvnrepository.com/artifact/org.openimaj/core-image)找到最新版本。

### 4.2 加载图像

要加载图像，请使用ImageUtilities.readMBF()方法：

```java
MBFImage image = ImageUtilities.readMBF(new File("path/to/your/image.jpg"));
```

MBF代表多波段浮点图像(此例中为RGB，但它不是表示颜色的唯一方式)。

### 4.3 编辑图像

要绘制矩形，我们需要定义它的形状，它是由4个点(左上、左下、右下、右上)组成的多边形：

```java
Point2d tl = new Point2dImpl(10, 10);
Point2d bl = new Point2dImpl(10, image.getHeight() - 10);
Point2d br = new Point2dImpl(image.getWidth() - 10, image.getHeight() - 10);
Point2d tr = new Point2dImpl(image.getWidth() - 10, 10);
Polygon polygon = new Polygon(Arrays.asList(tl, bl, br, tr));
```

你可能已经注意到，在图像处理中Y轴是反转的。定义形状后，我们需要绘制它：

```java
image.drawPolygon(polygon, 4, new Float[] { 0f, 0f, 255.0f });
```

绘图方法需要3个参数：形状、线条粗细和以Float数组表示的RGB通道值。

### 4.4 显示图像

我们需要使用DisplayUtilities：

```java
DisplayUtilities.display(image);
```

## 5. TwelveMonkeys ImageIO

TwelveMonkeys ImageIO库旨在作为Java ImageIO API的扩展，支持更多格式。

大多数情况下，代码看起来与内置的Java代码相同，但在添加必要的依赖后，它将可以与其他图像格式一起运行。

默认情况下，Java仅支持以下5种图像格式：JPEG、PNG、BMP、WEBMP、GIF。

如果我们尝试使用不同格式的图像文件，我们的应用程序将无法读取它，并且在访问BufferedImage变量时会抛出NullPointerException。

TwelveMonkeys增加了对以下格式的支持：PNM、PSD、TIFF、HDR、IFF、PCX、PICT、SGI、TGA、ICNS、ICO、CUR、Thumbs.db、SVG、WMF。

**要处理特定格式的图像，我们需要添加相应的依赖，例如[imageio-jpeg](https://mvnrepository.com/artifact/com.twelvemonkeys.imageio/imageio-jpeg)或[imageio-tiff](https://mvnrepository.com/artifact/com.twelvemonkeys.imageio/imageio-tiff)**。

你可以在[TwelveMonkeys](https://github.com/haraldk/TwelveMonkeys)文档中找到依赖的完整列表。

让我们创建一个读取.ico图像的示例，代码看起来与AWT部分相同，只是我们将打开不同的图像：

```java
String imagePath = "path/to/your/image.ico";
BufferedImage myPicture = ImageIO.read(new File(imagePath));
```

为了使此示例正常工作，我们需要添加包含对.ico图像的支持的TwelveMonkeys依赖，即[imageio-bmp](https://mvnrepository.com/artifact/com.twelvemonkeys.imageio/imageio-bmp)依赖，以及[imageio-core](https://mvnrepository.com/artifact/com.twelvemonkeys.imageio/imageio-core)依赖：

```xml
<dependency>
    <groupId>com.twelvemonkeys.imageio</groupId>
    <artifactId>imageio-bmp</artifactId>
    <version>3.3.2</version>
</dependency>
<dependency>
    <groupId>com.twelvemonkeys.imageio</groupId>
    <artifactId>imageio-core</artifactId>
    <version>3.3.2</version>
</dependency>
```

就这些了！**内置的ImageIO Java API会在运行时自动加载插件**，现在我们的项目也可以使用.ico图像了。

## 6. 总结

我们介绍了4个可以帮助你处理图像的库。进一步讲，你可能想要寻找一些图像处理算法，例如提取边缘、增强对比度、使用滤镜或人脸检测。

出于这些目的，最好开始学习ImageJ或OpenIMAJ，两者都很容易包含在项目中，并且在图像处理方面比AWT强大得多。