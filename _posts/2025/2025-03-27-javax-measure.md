---
layout: post
title:  javax.measure简介
category: libraries
copyright: libraries
excerpt: javax.measure
---

## 1. 概述

在本文中，我们将介绍测量单位API-它提供了一种在Java中表示测量和单位的统一方法。

在使用包含物理量的程序时，我们需要消除对所用单位的不确定性。我们必须管理数字及其单位，以防止计算错误。

JSR-363(以前称为JSR-275或javax.measure库)帮助我们节省了开发时间，同时使代码更具可读性。

## 2. Maven依赖

让我们简单地从Maven依赖开始引入库：

```xml
<dependency>
    <groupId>javax.measure</groupId>
    <artifactId>unit-api</artifactId>
    <version>1.0</version>
</dependency>

```

最新版本可以在[Maven Central](https://mvnrepository.com/search?q=unit-api)上找到。

unit-api项目包含一组接口，定义如何使用数量和单位。为了举例说明，我们将使用JSR-363的参考实现，即[unit-ri](https://mvnrepository.com/search?q=unit-ri)：

```xml
<dependency>
    <groupId>tec.units</groupId>
    <artifactId>unit-ri</artifactId>
    <version>1.0.3</version>
</dependency>
```

## 3. 探索API

让我们看一个在水箱中储水的例子。

传统的实现方式如下：

```java
public class WaterTank {
    public void setWaterQuantity(double quantity);
}
```

可以看出，上述代码没有提到水量的单位，由于double类型的存在，不适合进行精确的计算。

如果开发人员错误地传递了与我们预期不同的计量单位的值，则会导致严重的计算错误。此类错误很难检测和解决。

**JSR-363 API为我们提供了Quantity和Unit接口**，解决了这种混淆，并将这些类型的错误排除在我们程序的范围之外。

### 3.1 简单示例

现在，让我们探索一下，看看这在我们的示例中如何有用。

如前所述，**JSR-363包含Quantity接口，该接口表示体积或面积等定量属性**。该库提供了许多子接口，用于模拟最常用的可量化属性。一些示例包括：体积、长度、电荷、能量、温度。

我们可以定义Quantity<Volume\>对象，它应该存储我们示例中的水量：

```java
public class WaterTank {
    public void setCapacityMeasure(Quantity<Volume> capacityMeasure);
}
```

除了Quantity接口之外，我们还可以使用Unit接口来标识属性的度量单位。常用单位的定义可以在unit-ri库中找到，例如：KELVIN、METRE、NEWTON、CELSIUS。

Quantity<Q extends Quantity<Q\>\>类型的对象具有检索单位和值的方法：getUnit()和getValue()。

让我们看一个设置水量值的例子：

```java
@Test
public void givenQuantity_whenGetUnitAndConvertValue_thenSuccess() {
    WaterTank waterTank = new WaterTank();
    waterTank.setCapacityMeasure(Quantities.getQuantity(9.2, LITRE));
    assertEquals(LITRE, waterTank.getCapacityMeasure().getUnit());

    Quantity<Volume> waterCapacity = waterTank.getCapacityMeasure();
    double volumeInLitre = waterCapacity.getValue().doubleValue();
    assertEquals(9.2, volumeInLitre, 0.0f);
}
```

我们还可以快速将“LITRE”中的体积转换为任何其他单位：

```java
double volumeInMilliLitre = waterCapacity
    .to(MetricPrefix.MILLI(LITRE)).getValue().doubleValue();
assertEquals(9200.0, volumeInMilliLitre, 0.0f);
```

但是，当我们尝试将水的量转换为另一个单位(不是Volume类型)时，我们会收到编译错误：

```java
// compilation error
waterCapacity.to(MetricPrefix.MILLI(KILOGRAM));
```

### 3.2 类参数化

为了保持维度的一致性，框架自然利用了泛型。

类和接口通过其数量类型进行参数化，这使得我们可以在编译时检查单位。编译器将根据其识别的内容给出错误或警告：

```java
Unit<Length> Kilometer = MetricPrefix.KILO(METRE);
Unit<Length> Centimeter = MetricPrefix.CENTI(LITRE); // compilation error
```

总是有可能使用asType()方法绕过类型检查：

```java
Unit<Length> inch = CENTI(METER).times(2.54).asType(Length.class);
```

如果我们不确定数量的类型，我们也可以使用通配符：

```java
Unit<?> kelvinPerSec = KELVIN.divide(SECOND);
```

## 4. 单位换算

Unit可以从SystemOfUnits中检索，规范的参考实现包含接口的Units实现，该实现提供了一组表示最常用单位的静态常量。

此外，我们还可以创建一个全新的自定义单位，或者通过对现有单位应用代数运算来创建一个单位。

使用标准单位的好处是我们不会陷入转换陷阱。

我们还可以使用MetricPrefix类中的前缀或乘数，例如KILO(Unit<Q\> unit)和CENTI(Unit<Q\> unit)，它们分别相当于乘以和除以10的幂。

例如，我们可以将“Kilometer”和“Centimeter”定义为：

```java
Unit<Length> Kilometer = MetricPrefix.KILO(METRE);
Unit<Length> Centimeter = MetricPrefix.CENTI(METRE);
```

当我们想要的单位不能直接使用时，就可以使用这些。

### 4.1 自定义单位

无论如何，如果单位制中不存在某个单位，我们可以使用新符号创建新单位：

- AlternateUnit：具有相同维度但不同符号和性质的新单位
- ProductUnit：由其他单位的合理幂乘积创建的新单位

让我们使用这些类创建一些自定义单位，压力的AlternateUnit示例：

```java
@Test
public void givenUnit_whenAlternateUnit_ThenGetAlternateUnit() {
    Unit<Pressure> PASCAL = NEWTON.divide(METRE.pow(2))
        .alternate("Pa").asType(Pressure.class);
    assertTrue(SimpleUnitFormat.getInstance().parse("Pa")
        .equals(PASCAL));
}
```

同样，ProductUnit及其转换的示例：

```java
@Test
public void givenUnit_whenProduct_ThenGetProductUnit() {
    Unit<Area> squareMetre = METRE.multiply(METRE).asType(Area.class);
    Quantity<Length> line = Quantities.getQuantity(2, METRE);
    assertEquals(line.multiply(line).getUnit(), squareMetre);
}
```

在这里，我们通过将METRE与其自身相乘创建了一个平方米复合单位。

接下来，对于单位类型，框架还提供了一个UnitConverter类，它可以帮助我们将一个单位转换为另一个单位，或者创建一个名为TransformedUnit的新派生单位。

让我们看一个将double值的单位从米转换为公里的示例：

```java
@Test
public void givenMeters_whenConvertToKilometer_ThenConverted() {
    double distanceInMeters = 50.0;
    UnitConverter metreToKilometre = METRE.getConverterTo(MetricPrefix.KILO(METRE));
    double distanceInKilometers = metreToKilometre.convert(distanceInMeters );
    assertEquals(0.05, distanceInKilometers, 0.00f);
}
```

为了促进数量与其单位的明确电子通信，该库提供了UnitFormat接口，将系统范围的标签与单位关联起来。

让我们使用SimpleUnitFormat实现检查一些系统单位的标签：

```java
@Test
public void givenSymbol_WhenCompareToSystemUnit_ThenSuccess() {
    assertTrue(SimpleUnitFormat.getInstance().parse("kW")
        .equals(MetricPrefix.KILO(WATT)));
    assertTrue(SimpleUnitFormat.getInstance().parse("ms")
        .equals(SECOND.divide(1000)));
}
```

## 5. 执行数量运算

Quantity接口包含最常见的数学运算方法：add()，subtract()，multiply()，divide()。使用这些方法，我们可以在Quantity对象之间执行运算：

```java
@Test
public void givenUnits_WhenAdd_ThenSuccess() {
    Quantity<Length> total = Quantities.getQuantity(2, METRE)
        .add(Quantities.getQuantity(3, METRE));
    assertEquals(total.getValue().intValue(), 5);
}
```

这些方法还会验证所操作对象的单位。例如，尝试将米乘以升将导致编译错误：

```java
// compilation error
Quantity<Length> total = Quantities.getQuantity(2, METRE)
    .add(Quantities.getQuantity(3, LITRE));
```

另一方面，可以相加两个以具有相同维度的单位表示的对象：

```java
Quantity<Length> totalKm = Quantities.getQuantity(2, METRE)
    .add(Quantities.getQuantity(3, MetricPrefix.KILO(METRE)));
assertEquals(totalKm.getValue().intValue(), 3002);
```

在此示例中，米和公里单位都与长度维度相对应，因此可以将它们相加。结果以第一个对象的单位表示。

## 6. 总结

在本文中，我们看到了Units of Measurement API为我们提供了一个方便的测量模型。并且，除了Quantity和Unit的使用之外，我们还看到了通过多种方式将一个单位转换为另一个单位是多么方便。