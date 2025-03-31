---
layout: post
title:  使用Java进行有损和无损图像压缩
category: libraries
copyright: libraries
excerpt: Apache Commons Imaging、Thumbnails、Pngtastic
---

## 1. 简介

在本教程中，我们将探索如何使用Java压缩图像。我们将首先使用Java中用于图像压缩的内置库来压缩图像，然后我们将介绍替代库Apache Commons Imaging。

让我们首先了解一些有关图像压缩的知识。

## 2. 什么是图像压缩？

[图像压缩](https://en.wikipedia.org/wiki/Image_compression)使我们能够减小图像文件的大小，而不会显著损害其视觉质量。压缩有两种类型；首先，**我们使用有损压缩来接受降低的图像质量，同时实现更小的文件大小**。例如，我们有JPEG和WebP格式用于有损压缩。其次，**我们使用无损压缩在压缩过程中保留数据和信息。例如，无损压缩期间使用PNG和GIF格式**。

现在，我们将重点介绍使用JPEG格式的有损压缩，因为它是互联网上使用最广泛的格式。之后，我们将了解如何压缩PNG图像，即PNG图像优化。

## 3. 使用Java Image I/O压缩图像

首先，我们将使用Java [Image I/O](https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/javax/imageio/ImageIO.html)的内置API来读取和写入图像。它支持各种图像格式，包括JPEG、PNG、BMP和GIF，让我们看看如何使用Java Image I/O压缩图像：

```java
File inputFile = new File("input_image.jpg");

BufferedImage inputImage = ImageIO.read(inputFile);

Iterator<ImageWriter> writers = ImageIO.getImageWritersByFormatName("jpg");
ImageWriter writer = writers.next();

File outputFile = new File("output.jpg");
ImageOutputStream outputStream = ImageIO.createImageOutputStream(outputFile);
writer.setOutput(outputStream);

ImageWriteParam params = writer.getDefaultWriteParam();
params.setCompressionMode(ImageWriteParam.MODE_EXPLICIT);
params.setCompressionQuality(0.5f);

writer.write(null, new IIOImage(inputImage, null, null), params);

outputStream.close();
writer.dispose();
```

首先，我们从资源文件中读取图像。然后，我们为JPG格式创建一个ImageWriter，并设置此写入器的输出文件。在写入图像之前，我们创建ImageWriteParam对象来定义压缩模式并将压缩质量设置为50%。最后，我们写入图像，关闭输出流并清理写入器。

例如，通过将示例图像压缩50%，我们几乎将文件大小从790KB减小到656KB，略低于初始大小的83%。因此，图片质量的变化并不明显：

![](/assets/images/2025/libraries/javaimagecompressionlossylossless01.png)

## 4. 使用Thumbnails库压缩图像

[Thumbnails库](https://mvnrepository.com/artifact/net.coobird/thumbnailator/0.4.19)是一个简单且多功能的库，用于调整图像大小和压缩，我们首先将该库添加到pom.xml中：

```xml
<dependency>
    <groupId>net.coobird</groupId>
    <artifactId>thumbnailator</artifactId>
    <version>0.4.19</version>
</dependency>
```

让我们看看如何使用Thumbnails类来压缩图像：

```java
File input = new File("input_image.jpg");
File output = new File("output.jpg");

Thumbnails.of(input)
    .scale(1) 
    .outputQuality(0.5)
    .toFile(output);
```

这里，scale(1)方法保持原始图像尺寸，而outputQuality(0.5)将输出质量设置为50%。

## 5. 使用Pngtastic库压缩图像

PNG优化是一种专门为PNG图像设计的压缩类型，我们将使用[Pngtastic库](https://github.com/depsypher/pngtastic/)来优化PNG图像。首先，让我们将[最新库](https://mvnrepository.com/artifact/com.github.depsypher/pngtastic/1.7)添加到pom.xml中：

```xml
<dependency>
    <groupId>com.github.depsypher</groupId>
    <artifactId>pngtastic</artifactId>
    <version>1.7</version>
</dependency>
```

最后，我们可以使用PngOptimizer类来压缩PNG文件：

```java
PngImage inputImage = new PngImage(Files.newInputStream(Paths.get(inputPath)));

PngOptimizer optimizer = new PngOptimizer();
PngImage optimized = optimizer.optimize(inputImage);

OutputStream output = Files.newOutputStream(Paths.get(outputPath));
optimized.writeDataOutputStream(output);
```

我们使用.optimize()方法让库决定最佳压缩方式，作为无损压缩，很难显著减小图像大小。在这里，我们将大小从500KB减小到了481KB。

![](/assets/images/2025/libraries/javaimagecompressionlossylossless02.png)

## 6. 总结

在本文中，我们介绍了两种使用Java进行有损压缩的方法：内置Java Image I/O API和Apache Commons Imaging库。然后，我们使用Pngtastic库对PNG图像进行无损压缩。