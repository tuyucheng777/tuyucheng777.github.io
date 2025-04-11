---
layout: post
title:  在Java中计算两个坐标之间的距离
category: algorithms
copyright: algorithms
excerpt: 坐标
---

## 1. 概述

在这个教程中，我们将实现计算两个地理坐标之间距离的方法。

具体来说，我们首先会实现距离的近似计算。然后，我们将研究Haversine和Vincenty公式，它们可以提供更高的精度。

## 2. 等距矩形距离近似

让我们从实现等距矩形近似开始；具体来说，由于这个公式使用的数学运算最少，因此速度非常快：
```java
double calculateDistance(double lat1, double lon1, double lat2, double lon2) {
    double lat1Rad = Math.toRadians(lat1);
    double lat2Rad = Math.toRadians(lat2);
    double lon1Rad = Math.toRadians(lon1);
    double lon2Rad = Math.toRadians(lon2);

    double x = (lon2Rad - lon1Rad) * Math.cos((lat1Rad + lat2Rad) / 2);
    double y = (lat2Rad - lat1Rad);
    double distance = Math.sqrt(x * x + y * y) * EARTH_RADIUS;

    return distance;
}
```

上面，EARTH_RADIUS是一个等于6371的常数，它很好地近似了地球半径(以公里为单位)。

虽然这似乎是一个简单的公式，**但等距矩形近似在计算长距离时并不十分准确**。事实上，它将地球视为一个完美的球体，并将球体映射到矩形网格上。

## 3. 使用Haversine公式计算距离

接下来，我们来看看[Haversine公式](https://en.wikipedia.org/wiki/Haversine_formula)。同样，它将地球视为一个完美的球体，尽管如此，它在计算长距离之间的距离时更准确。

此外，**Haversine公式基于[半正矢](https://en.wikipedia.org/wiki/Versine#Haversine)的球面定律**：
```java
double haversine(double val) {
    return Math.pow(Math.sin(val / 2), 2);
}
```

然后，使用这个辅助函数，我们可以实现计算距离的方法：
```java
double calculateDistance(double startLat, double startLong, double endLat, double endLong) {

    double dLat = Math.toRadians((endLat - startLat));
    double dLong = Math.toRadians((endLong - startLong));

    startLat = Math.toRadians(startLat);
    endLat = Math.toRadians(endLat);

    double a = haversine(dLat) + Math.cos(startLat) * Math.cos(endLat) * haversine(dLong);
    double c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));

    return EARTH_RADIUS * c;
}
```

虽然它提高了计算的准确性，但它仍然将地球视为扁平的形状。

## 4. 使用Vincenty公式计算距离

最后，如果想要最高的精度，我们必须使用[Vincenty公式](https://en.wikipedia.org/wiki/Vincenty's_formulae)。具体来说，**Vincenty公式会迭代计算距离，直到误差达到可接受的值**。此外，它还考虑到了地球的椭圆形状。

首先，该公式需要一些描述地球椭球模型的[常数](https://en.wikipedia.org/wiki/World_Geodetic_System#WGS84)：
```java
double SEMI_MAJOR_AXIS_MT = 6378137;
double SEMI_MINOR_AXIS_MT = 6356752.314245;
double FLATTENING = 1 / 298.257223563;
double ERROR_TOLERANCE = 1e-12;
```

显然，ERROR_TOLERANCE代表我们愿意接受的误差，此外，我们将在Vincenty公式中使用这些值：
```java
double calculateDistance(double latitude1, double longitude1, double latitude2, double longitude2) {
    double U1 = Math.atan((1 - FLATTENING) * Math.tan(Math.toRadians(latitude1)));
    double U2 = Math.atan((1 - FLATTENING) * Math.tan(Math.toRadians(latitude2)));

    double sinU1 = Math.sin(U1);
    double cosU1 = Math.cos(U1);
    double sinU2 = Math.sin(U2);
    double cosU2 = Math.cos(U2);

    double longitudeDifference = Math.toRadians(longitude2 - longitude1);
    double previousLongitudeDifference;

    double sinSigma, cosSigma, sigma, sinAlpha, cosSqAlpha, cos2SigmaM;

    do {
        sinSigma = Math.sqrt(Math.pow(cosU2 * Math.sin(longitudeDifference), 2) +
                Math.pow(cosU1 * sinU2 - sinU1 * cosU2 * Math.cos(longitudeDifference), 2));
        cosSigma = sinU1 * sinU2 + cosU1 * cosU2 * Math.cos(longitudeDifference);
        sigma = Math.atan2(sinSigma, cosSigma);
        sinAlpha = cosU1 * cosU2 * Math.sin(longitudeDifference) / sinSigma;
        cosSqAlpha = 1 - Math.pow(sinAlpha, 2);
        cos2SigmaM = cosSigma - 2 * sinU1 * sinU2 / cosSqAlpha;
        if (Double.isNaN(cos2SigmaM)) {
            cos2SigmaM = 0;
        }
        previousLongitudeDifference = longitudeDifference;
        double C = FLATTENING / 16 * cosSqAlpha * (4 + FLATTENING * (4 - 3 * cosSqAlpha));
        longitudeDifference = Math.toRadians(longitude2 - longitude1) + (1 - C) * FLATTENING * sinAlpha *
                (sigma + C * sinSigma * (cos2SigmaM + C * cosSigma * (-1 + 2 * Math.pow(cos2SigmaM, 2))));
    } while (Math.abs(longitudeDifference - previousLongitudeDifference) > ERROR_TOLERANCE);

    double uSq = cosSqAlpha * (Math.pow(SEMI_MAJOR_AXIS_MT, 2) - Math.pow(SEMI_MINOR_AXIS_MT, 2)) / Math.pow(SEMI_MINOR_AXIS_MT, 2);

    double A = 1 + uSq / 16384 * (4096 + uSq * (-768 + uSq * (320 - 175 * uSq)));
    double B = uSq / 1024 * (256 + uSq * (-128 + uSq * (74 - 47 * uSq)));

    double deltaSigma = B * sinSigma * (cos2SigmaM + B / 4 * (cosSigma * (-1 + 2 * Math.pow(cos2SigmaM, 2))
            - B / 6 * cos2SigmaM * (-3 + 4 * Math.pow(sinSigma, 2)) * (-3 + 4 * Math.pow(cos2SigmaM, 2))));

    double distanceMt = SEMI_MINOR_AXIS_MT * A * (sigma - deltaSigma);
    return distanceMt / 1000;
}
```

此公式计算量很大，因此，当精度是目标时，我们可能会使用它。否则，我们还是坚持使用Haversine公式。

## 5. 测试准确度

最后，我们可以测试一下上面看到的所有方法的准确性：
```java
double lat1 = 40.714268; // New York
double lon1 = -74.005974;
double lat2 = 34.0522; // Los Angeles
double lon2 = -118.2437;

double equirectangularDistance = EquirectangularApproximation.calculateDistance(lat1, lon1, lat2, lon2);
double haversineDistance = HaversineDistance.calculateDistance(lat1, lon1, lat2, lon2);
double vincentyDistance = VincentyDistance.calculateDistance(lat1, lon1, lat2, lon2);

double expectedDistance = 3944;
assertTrue(Math.abs(equirectangularDistance - expectedDistance) < 100);
assertTrue(Math.abs(haversineDistance - expectedDistance) < 10);
assertTrue(Math.abs(vincentyDistance - expectedDistance) < 0.5);
```

上面我们计算了纽约和洛杉矶之间的距离，然后评估了公里为单位的精度。

## 6. 总结

在本文中，我们了解了三种使用Java计算两个地理点之间距离的方法；首先，我们介绍了精度最低的等距矩形近似法；然后，我们研究了精度更高的Haversine公式；最后，我们使用了精度最高的[Vincenty公式](https://www.baeldung.com/java-find-distance-between-points)。