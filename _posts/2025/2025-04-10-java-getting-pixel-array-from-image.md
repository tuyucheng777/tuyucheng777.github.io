---
layout: post
title:  使用Java从图像获取像素数组
category: algorithms
copyright: algorithms
excerpt: 像素
---

## 1. 概述

在本教程中，我们将学习如何从Java中的[BufferedImage](https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/java/awt/image/BufferedImage.html)实例中获取包含图像信息(RGB值)的像素数组。

## 2. 什么是BufferedImage类？

[BufferedImage](https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/java/awt/image/BufferedImage.html)类是Image的子类，它描述具有可访问的图像数据缓冲区的图形图像。BufferedImage由[ColorModel](https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/java/awt/image/ColorModel.html)和[Raster](https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/java/awt/image/Raster.html)组成。

ColorModel描述了如何使用组件组合作为值元组来表示颜色，Java中的ColorModel类包含一些可以返回特定像素颜色值的方法。例如，getBlue(int pixel)返回给定像素的蓝色值。

此外，Raster类包含像素数组中的图像数据。Raster类由存储图像值的[DataBuffer](https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/java/awt/image/DataBuffer.html)和描述像素在DataBuffer中存储方式的[SampleModel](https://docs.oracle.com/en/java/javase/21/docs/api/java.desktop/java/awt/image/SampleModel.html)组成。

## 3. 使用getRGB()

第一种方法是使用BufferedImage类中的getRGB()实例方法。

**getRGB()方法将指定像素的RGB值组合成一个整数并返回结果**，此整数包含可使用实例的ColorModel访问的RGB值。此外，要获取图像中每个像素的结果，我们必须对它们进行迭代并为每个像素单独调用该方法：
```java
public int[][] get2DPixelArraySlow(BufferedImage sampleImage) {
    int width = sampleImage.getWidth();
    int height = sampleImage.getHeight();
    int[][] result = new int[height][width];

    for (int row = 0; row < height; row++) {
        for (int col = 0; col < width; col++) {
            result[row][col] = sampleImage.getRGB(col, row);
        }
    }

    return result;
}
```

在上面的代码片段中，result数组是一个二维数组，其中包含图像中每个像素的RGB值。这种方法比下一种方法更直接，但效率也较低。

## 4. 直接从DataBuffer获取值

**在这个方法中，我们首先分别从图像中获取所有RGB值，然后手动将它们组合成一个整数**。之后，我们像第一种方法一样填充包含像素值的二维数组。这种方法比第一种方法更复杂，但速度更快：
```java
public int[][] get2DPixelArrayFast(BufferedImage image) {
    byte[] pixelData = ((DataBufferByte) image.getRaster().getDataBuffer()).getData();
    int width = image.getWidth();
    int height = image.getHeight();
    boolean hasAlphaChannel = image.getAlphaRaster() != null;

    int[][] result = new int[height][width];
    if (hasAlphaChannel) {
        int numberOfValues = 4;
        for (int valueIndex = 0, row = 0, col = 0; valueIndex + numberOfValues - 1 < pixelData.length; valueIndex += numberOfValues) {

            int argb = 0;
            argb += (((int) pixelData[valueIndex] & 0xff) << 24); // alpha value
            argb += ((int) pixelData[valueIndex + 1] & 0xff); // blue value
            argb += (((int) pixelData[valueIndex + 2] & 0xff) << 8); // green value
            argb += (((int) pixelData[valueIndex + 3] & 0xff) << 16); // red value
            result[row][col] = argb;

            col++;
            if (col == width) {
                col = 0;
                row++;
            }
        }
    } else {
        int numberOfValues = 3;
        for (int valueIndex = 0, row = 0, col = 0; valueIndex + numberOfValues - 1 < pixelData.length; valueIndex += numberOfValues) {
            int argb = 0;
            argb += -16777216; // 255 alpha value (fully opaque)
            argb += ((int) pixelData[valueIndex] & 0xff); // blue value
            argb += (((int) pixelData[valueIndex + 1] & 0xff) << 8); // green value
            argb += (((int) pixelData[valueIndex + 2] & 0xff) << 16); // red value
            result[row][col] = argb;

            col++;
            if (col == width) {
                col = 0;
                row++;
            }
        }
    }

    return result;
}
```

在上面的代码片段中，我们首先获取图像中每个像素的单独RGB值，并将它们存储在名为pixelData的字节数组中。

例如，假设图像没有alpha通道(alpha通道包含图片的透明度信息)，pixelData[0\]包含图像中第一个像素的蓝色值，而pixelData[1\]和pixelData[2\]分别包含绿色和红色值。同样，pixelData[3\]到pixelData[5\]包含第二个图像像素的RGB值，依此类推。

获取这些值后，我们必须将它们组合成每个像素对应的一个整数，但在此之前，我们需要确定图像是否有alpha通道。如果图像有alpha通道，我们需要将4个值(红色、绿色、蓝色和透明度信息)组合成一个整数。如果没有，我们只需要组合RGB值。

将所有值组合成一个整数后，我们将该整数放入二维数组中的位置。

## 5. 总结

在这篇短文中，我们学习了如何使用Java获取包含图像中每个像素的组合RGB值的二维数组。