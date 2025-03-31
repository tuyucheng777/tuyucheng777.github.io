---
layout: post
title:  如何在Java中缩放缓冲图像
category: libraries
copyright: libraries
excerpt: Java Image
---

## 1. 简介

在本教程中，我们将介绍如何使用基本Java API重新缩放图像。我们将展示如何从文件加载和保存图像，并解释重新缩放过程的一些技术方面。

## 2. 用Java加载图像

在本教程中，我们将使用一个简单的JPG图像文件。我们将使用基本Java SDK中捆绑的[ImageIO](https://www.baeldung.com/java-images#twelvemonkeys-imageio) API加载它，此API有一些针对JPEG和PNG等格式的预设ImageReader，ImageReader知道如何读取各自的图像格式并从图像文件中获取位图。

我们将使用的方法是ImageIO中的read方法，此方法有几个重载，但我们选择最简单的一个：

```java
BufferedImage srcImg = ImageIO.read(new File("src/main/resources/images/sampleImage.jpg"));
```

我们可以看到，read()方法提供了一个BufferedImage对象，它是图像位图的主要Java表示。

## 3. 重新缩放图像

在重新调整加载的图像的大小之前，我们必须做一些准备。

### 3.1 创建目标图像

首先，我们必须创建一个新的BufferedImage对象来表示内存中的缩放图像，也称为目标图像。由于我们要重新缩放，这意味着生成的图像的宽度和高度将与原始图像不同。

**我们必须在新的BufferedImage中设置缩放尺寸**：

```java
float scaleW = 2.0f, scaleH = 2.0f;
int w = srcImg.getWidth() * (int) scaleW;
int h = srcImg.getHeight() * (int) scaleH;
BufferedImage dstImg = new BufferedImage(w, h, srcImg.getType());
```

如代码所示，宽度和高度的缩放因子不必相同。但是，它们通常是相同的，因为使用不同的缩放因子会导致结果失真。

BufferedImage构造函数还需要一个imageType参数，不要将其与图像文件格式(例如PNG或JPEG)混淆；**图像类型决定了新BufferedImage的颜色空间**。该类本身为受支持的值提供静态int成员，例如分别用于彩色和灰度图像的BufferedImage.TYPE_INT_RGB和BufferedImage.TYPE_BYTE_GRAY。在我们的例子中，我们将使用与源图像相同的类型，因为我们只更改比例。

下一步是应用转换，将源图像变为我们的目标尺寸。

### 3.2 应用AffineTransform

**我们将通过应用[缩放仿射变换](https://www.baeldung.com/cs/homography-vs-affine-transformation#affine-transformation)来缩放图像**，这些线性变换可以将点从一个2D平面映射到另一个2D平面。根据变换，目标平面可以是原始平面的放大版本，甚至是旋转版本。

在我们的例子中，我们只应用缩放。**最简单的理解方法是取组成图像的所有点，并通过缩放因子来增加它们之间的距离**。

让我们创建一个AffineTransform及其相应的操作：

```java
AffineTransform scalingTransform = new AffineTransform();
scalingTransform.scale(scaleW, scaleH);
AffineTransformOp scaleOp = new AffineTransformOp(scalingTransform, AffineTransformOp.TYPE_BILINEAR);
```

**AffineTransform定义我们将应用什么操作，而AffineTransformOp定义如何应用它**。我们创建一个将使用scalingTransform的操作并使用双线性插值应用它。

**所选的插值算法是根据具体情况确定的，并决定了如何选择新图像的像素值**。这些插值算法的作用以及为什么它们是强制性的超出了本文的范围，理解它们需要知道我们为什么使用这些线性变换以及它们如何应用于2D图像。

一旦scaleOp准备就绪，我们就可以将其应用到srcImg并将结果放入dstImg中：

```java
dstImg = scaleOp.filter(srcImg, dstImg);
```

最后，我们可以将dstImg保存到文件中，以便查看结果：

```java
ImageIO.write(dstImg, "jpg", new File("src/main/resources/images/resized.jpg"));
```

## 4. 总结

在本文中，我们学习了如何按任意比例缩放图像。我们展示了如何从文件系统加载/保存图像以及如何使用Java的AffineTransform应用缩放操作。