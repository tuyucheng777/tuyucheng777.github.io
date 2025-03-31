---
layout: post
title:  如何使用Java调整图像大小
category: libraries
copyright: libraries
excerpt: Thumbnailator、Imgscalr、Marvin
---

## 1. 简介

在本教程中，我们将学习如何使用Java调整图像大小(缩放)。我们将探索提供图像大小调整功能的核心Java和开源第三方库。

值得一提的是，我们可以放大或缩小图像。在本教程的代码示例中，我们将图像调整为较小的尺寸，因为在实践中，这是最常见的情况。

## 2. 使用核心Java调整图像大小

核心Java提供了以下调整图像大小的选项：

- 使用[java.awt.Graphics2D](https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/java/awt/Graphics2D.html)调整大小
- 使用[Image#getScaledInstance](https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/java/awt/Image.html)调整大小

### 2.1 java.awt.Graphics2D

Graphics2D是Java平台上渲染二维形状、文本和图像的基本类。

让我们首先使用Graphics2D调整图像大小：

```java
BufferedImage resizeImage(BufferedImage originalImage, int targetWidth, int targetHeight) throws IOException {
    BufferedImage resizedImage = new BufferedImage(targetWidth, targetHeight, BufferedImage.TYPE_INT_RGB);
    Graphics2D graphics2D = resizedImage.createGraphics();
    graphics2D.drawImage(originalImage, 0, 0, targetWidth, targetHeight, null);
    graphics2D.dispose();
    return resizedImage;
}
```

让我们看看调整大小之前和之后的图像是什么样子的：

![](/assets/images/2025/libraries/javaresizeimage01.png)

![](/assets/images/2025/libraries/javaresizeimage02.png)

BufferedImage.TYPE_INT_RGB参数表示图像的颜色模型，可用值的完整列表可在[Java BufferedImage官方文档](https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/java/awt/image/BufferedImage.html)中找到。

Graphics2D接收称为RenderingHints的附加参数，**我们使用RenderingHints来影响不同的图像处理方面，最重要的是图像质量和处理时间**。

我们可以使用setRenderingHint方法添加RenderingHint：

```java
graphics2D.setRenderingHint(RenderingHints.KEY_INTERPOLATION, RenderingHints.VALUE_INTERPOLATION_BILINEAR);
```

在[这个Oracle教程](https://docs.oracle.com/javase/tutorial/2d/advanced/quality.html)中可以找到RenderingHints的完整列表。

### 2.2 Image.getScaledInstance()

这种使用图像的方法非常简单，并且可以生成令人满意的质量的图像：

```java
BufferedImage resizeImage(BufferedImage originalImage, int targetWidth, int targetHeight) throws IOException {
    Image resultingImage = originalImage.getScaledInstance(targetWidth, targetHeight, Image.SCALE_DEFAULT);
    BufferedImage outputImage = new BufferedImage(targetWidth, targetHeight, BufferedImage.TYPE_INT_RGB);
    outputImage.getGraphics().drawImage(resultingImage, 0, 0, null);
    return outputImage;
}
```

让我们看看一张美味图片会发生什么：

![](/assets/images/2025/libraries/javaresizeimage03.png)

![](/assets/images/2025/libraries/javaresizeimage04.png)

我们还可以通过为getScaledInstance()方法提供一个标志来指示缩放机制使用其中一种可用方法，该标志指示用于满足图像重采样需求的算法类型：

```java
Image resultingImage = originalImage.getScaledInstance(targetWidth, targetHeight, Image.SCALE_SMOOTH);
```

所有可用的标志均在官方[Java Image文档](https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/java/awt/Image.html)中描述。

## 3. Imgscalr

[Imgscalr](https://github.com/rkalla/imgscalr)在后台使用Graphic2D，它有一个简单的API，其中包含几种不同的图像大小调整方法。

**根据我们选择的缩放选项，Imgscalr可以为我们提供最佳效果、最快效果或均衡效果**。还提供其他图像处理功能，例如裁剪和旋转功能。让我们通过一个简单的示例来展示它的工作原理。

我们将添加以下Maven依赖：

```xml
<dependency>
    <groupId>org.imgscalr</groupId>
    <artifactId>imgscalr-lib</artifactId>
    <version>4.2</version>
</dependency>
```

检查[Maven Central](https://mvnrepository.com/artifact/org.imgscalr)以获取最新版本。

使用Imgscalr最简单的方法是：

```java
BufferedImage simpleResizeImage(BufferedImage originalImage, int targetWidth) throws Exception {
    return Scalr.resize(originalImage, targetWidth);
}
```

其中originalImage是要调整大小的BufferedImage，targetWidth是结果图像的宽度。此方法将保持原始图像比例并使用默认参数-Method.AUTOMATIC和Mode.AUTOMATIC。

它如何处理美味水果的图片？让我们看看：

![](/assets/images/2025/libraries/javaresizeimage05.png)

![](/assets/images/2025/libraries/javaresizeimage06.png)

**该库还允许多种配置选项，并且在后台处理图像透明度**。

最重要的参数是：

- mode：用于定义算法将使用的调整大小模式。例如，我们可以定义是否要保持图像的比例(选项为AUTOMATIC、FIT_EXACT、FIT_TO_HEIGHT和FIT_TO_WIDTH)
- method ：指示调整大小过程，使其重点关注速度、质量或两者。可能的值包括AUTOMATIC、BALANCED、QUALITY、SPEED、ULTRA_QUALITY

还可以定义额外的调整大小属性，为我们提供日志记录或指示库对图像进行一些颜色修改(使其更亮、更暗、灰度等)。

让我们使用完整的resize()方法参数化：

```java
BufferedImage resizeImage(BufferedImage originalImage, int targetWidth, int targetHeight) throws Exception {
    return Scalr.resize(originalImage, Scalr.Method.AUTOMATIC, Scalr.Mode.AUTOMATIC, targetWidth, targetHeight, Scalr.OP_ANTIALIAS);
}
```

现在我们得到：

![](/assets/images/2025/libraries/javaresizeimage07.png)

![](/assets/images/2025/libraries/javaresizeimage08.png)

Imgscalr适用于Java Image IO支持的所有文件-JPG、BMP、JPEG、WBMP、PNG和GIF。

## 4. Thumbnailator

[Thumbnailator](https://github.com/coobird/thumbnailator)是一个使用渐进式双线性缩放的Java开源图像大小调整库，它支持 JPG、BMP、JPEG、WBMP、PNG和GIF。

我们将通过向pom.xml添加以下Maven依赖将其包含在我们的项目中：

```xml
<dependency>
    <groupId>net.coobird</groupId>
    <artifactId>thumbnailator</artifactId>
    <version>0.4.11</version>
</dependency>
```

可以在[Maven Central](https://mvnrepository.com/artifact/net.coobird/thumbnailator)上找到可用的依赖版本。

它有一个非常简单的API，允许我们以百分比设置输出质量：

```java
BufferedImage resizeImage(BufferedImage originalImage, int targetWidth, int targetHeight) throws Exception {
    ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
    Thumbnails.of(originalImage)
            .size(targetWidth, targetHeight)
            .outputFormat("JPEG")
            .outputQuality(1)
            .toOutputStream(outputStream);
    byte[] data = outputStream.toByteArray();
    ByteArrayInputStream inputStream = new ByteArrayInputStream(data);
    return ImageIO.read(inputStream);
}
```

让我们看看这张微笑的照片在调整大小之前和之后的样子：

![](/assets/images/2025/libraries/javaresizeimage09.png)

![](/assets/images/2025/libraries/javaresizeimage10.png)

它还具有批处理选项：

```java
Thumbnails.of(new File("path/to/directory").listFiles())
    .size(300, 300)
    .outputFormat("JPEG")
    .outputQuality(0.80)
    .toFiles(Rename.PREFIX_DOT_THUMBNAIL);
```

与Imgscalr一样，Thumblinator可处理Java Image IO支持的所有文件-JPG、BMP、JPEG、WBMP、PNG和GIF。

## 5. Marvin

[Marvin](http://marvinproject.sourceforge.net/)是一款方便的图像处理工具，它提供许多有用的基本功能(裁剪、旋转、倾斜、翻转、缩放)和高级功能(模糊、浮雕、纹理)。

与以前一样，我们将添加Marvin调整大小所需的Maven依赖：

```xml
<dependency>
    <groupId>com.github.downgoon</groupId>
    <artifactId>marvin</artifactId>
    <version>1.5.5</version>
    <type>pom</type>
</dependency>
<dependency>
    <groupId>com.github.downgoon</groupId>
    <artifactId>MarvinPlugins</artifactId>
    <version>1.5.5</version>
</dependency>
```

可以在[Maven Central](https://mvnrepository.com/artifact/com.github.downgoon/marvin)上找到可用的Marvin依赖版本以及[Marvin插件](https://mvnrepository.com/artifact/com.github.downgoon/MarvinPlugins)版本。

Marvin的缺点是它不提供额外的缩放配置。此外，缩放方法需要图像和图像克隆，这有点麻烦：

```java
BufferedImage resizeImage(BufferedImage originalImage, int targetWidth, int targetHeight) {
    MarvinImage image = new MarvinImage(originalImage);
    Scale scale = new Scale();
    scale.load();
    scale.setAttribute("newWidth", targetWidth);
    scale.setAttribute("newHeight", targetHeight);
    scale.process(image.clone(), image, null, null, false);
    return image.getBufferedImageNoAlpha();
}
```

现在让我们调整一朵花的图像的大小并看看效果如何：

![](/assets/images/2025/libraries/javaresizeimage11.png)

![](/assets/images/2025/libraries/javaresizeimage12.png)

## 6. 最佳实践

**图像处理在资源方面是一项昂贵的操作**，因此当我们并不真正需要时，选择最高质量并不一定是最好的选择。

让我们看看所有方法的性能，我们拍摄一张1920 × 1920像素的图像，并将其缩放到200 × 200像素；观察到的时间如下：

- java.awt.Graphics2D：34毫秒
- Image.getScaledInstance()：235毫秒
- Imgscalr：143毫秒
- Thumbnailator：547毫秒
- Marvin：361毫秒

另外，在定义目标图像的宽度和高度时，我们应注意图像的纵横比。这样图像将保留其原始比例，不会被拉伸。

## 7. 总结

本文介绍了几种使用Java调整图像大小的方法，我们还了解了影响调整大小过程的因素有很多。