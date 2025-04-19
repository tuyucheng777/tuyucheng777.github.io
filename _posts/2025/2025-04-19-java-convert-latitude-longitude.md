---
layout: post
title:  在Java中将经度和纬度转换为二维点
category: algorithms
copyright: algorithms
excerpt: 二维点
---

## 1. 概述

在实现地图应用程序时，我们通常会遇到坐标转换的问题。大多数情况下，我们需要**将经纬度转换为二维点**来显示。幸运的是，我们可以利用墨卡托投影的公式来解决这个问题。

在本教程中，我们将介绍墨卡托投影并学习如何实现它的两种变体。

## 2. 墨卡托投影

[墨卡托投影](https://en.wikipedia.org/wiki/Mercator_projection)是由佛兰德制图师杰拉尔杜斯·墨卡托于1569年发明的一种地图投影，地图投影将地球上的经纬度坐标转换为平面上的一个点。换句话说，**它将地球表面上的一个点平移到平面地图上的一个点**。

实现墨卡托投影有两种方法，**伪墨卡托投影将地球视为球体，真墨卡托投影将地球建模为椭球体**，我们将实现这两个版本。

让我们从两种墨卡托投影实现的基类开始：

```java
abstract class Mercator {
    final static double RADIUS_MAJOR = 6378137.0;
    final static double RADIUS_MINOR = 6356752.3142;

    abstract double yAxisProjection(double input);
    abstract double xAxisProjection(double input);
}
```

这个类还提供了以米为单位的地球长半径和短半径，众所周知，地球并非完全球形。因此，我们需要两个半径。首先，**长半径是从地心到赤道的距离**。其次，**短半径是从地心到南北极的距离**。

### 2.1 球面墨卡托投影

伪投影模型将地球视为球体，与椭圆投影相比，椭圆投影会将地球投影到更精确的形状上，这种方法使我们能够快速估算出更精确但计算量更大的椭圆投影。因此，在该投影中直接测量的距离将是近似值。

此外，地图上形状的比例也会略有变化。因此，地图上国家、湖泊、河流等物体的纬度和形状比例无法精确保存。

这也称为[Web Mercator](https://en.wikipedia.org/wiki/Web_Mercator_projection)投影-通常用于包括Google Maps在内的Web应用程序中。

让我们实现这个方法：

```java
public class SphericalMercator extends Mercator {

    @Override
    double xAxisProjection(double input) {
        return Math.toRadians(input) * RADIUS_MAJOR;
    }

    @Override
    double yAxisProjection(double input) {
        return Math.log(Math.tan(Math.PI / 4 + Math.toRadians(input) / 2)) * RADIUS_MAJOR;
    }
}
```

关于这种方法，首先要注意的是，它用一个常数来表示地球半径，而不是实际的两个常数。其次，我们可以看到，我们已经实现了两个函数，分别用于转换为x轴投影和y轴投影。在上面的类中，我们使用了Java提供的Math库来简化代码。

让我们测试一个简单的转换：

```java
Assert.assertEquals(2449028.7974520186, sphericalMercator.xAxisProjection(22));
Assert.assertEquals(5465442.183322753, sphericalMercator.yAxisProjection(44));
```

值得注意的是，此投影将把点映射到(-20037508.34, -23810769.32,20037508.34,23810769.32)的边界框(左, 下, 右, 上)中。

### 2.2 椭圆墨卡托投影

真投影将地球建模为椭球体，这种投影可以精确地反映地球上任何位置的物体。当然，它尊重地图上的物体，但并非100%准确。然而，由于计算复杂，这种方法并非最常用。

让我们实现这个方法：

```java
class EllipticalMercator extends Mercator {
    @Override
    double yAxisProjection(double input) {
        input = Math.min(Math.max(input, -89.5), 89.5);
        double earthDimensionalRateNormalized = 1.0 - Math.pow(RADIUS_MINOR / RADIUS_MAJOR, 2);

        double inputOnEarthProj = Math.sqrt(earthDimensionalRateNormalized) * Math.sin( Math.toRadians(input));

        inputOnEarthProj = Math.pow(((1.0 - inputOnEarthProj) / (1.0+inputOnEarthProj)), 0.5 * Math.sqrt(earthDimensionalRateNormalized));

        double inputOnEarthProjNormalized = Math.tan(0.5 * ((Math.PI * 0.5) - Math.toRadians(input))) / inputOnEarthProj;

        return (-1) * RADIUS_MAJOR * Math.log(inputOnEarthProjNormalized);
    }

    @Override
    double xAxisProjection(double input) {
        return RADIUS_MAJOR * Math.toRadians(input);
    }
}
```

上图我们可以看出，这种方法在y轴上的投影非常复杂，这是因为它需要考虑地球形状并非圆形。虽然真正的墨卡托投影方法看起来很复杂，但它比球面投影方法更精确，因为它使用一个半径来表示地球的一个大半球和一个小半球。

让我们测试一个简单的转换：

```java
Assert.assertEquals(2449028.7974520186, ellipticalMercator.xAxisProjection(22));
Assert.assertEquals(5435749.887511954, ellipticalMercator.yAxisProjection(44));
```

该投影将点映射到(-20037508.34, -34619289.37,20037508.34,34619289.37)的边界框中。

## 3. 总结

如果我们需要将经纬度坐标转换到二维平面，可以使用墨卡托投影。根据我们实现所需的精度，我们可以使用球面或椭圆投影。