---
layout: post
title:  使用MapStruct映射枚举
category: libraries
copyright: libraries
excerpt: Mapstruct
---

## 1. 简介

在本教程中，我们将学习如何使用[MapStruct](https://www.baeldung.com/mapstruct)将一种[枚举](https://www.baeldung.com/a-guide-to-java-enums)类型映射到另一种枚举类型，将枚举映射到内置Java类型(例如int和String)，反之亦然。

## 2. Maven

让我们将以下依赖添加到Maven pom.xml中：

```xml
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>1.6.0.Beta1</version> 
</dependency>
```

[MapStruct](https://mvnrepository.com/artifact/org.mapstruct/mapstruct)的最新稳定版本可从Maven中央仓库获得。

## 3. 将一个枚举映射到另一个枚举

在本节中，我们将学习执行各种映射。

### 3.1 理解用例

现在，让我们看一些现实世界的场景。

在REST API响应映射中，MapStruct将外部API状态码转换为我们应用程序的内部状态枚举。

对于微服务中的数据转换，MapStruct通过映射类似的枚举促进服务之间的顺利数据交换。

与第三方库的集成通常涉及处理第三方枚举，MapStruct通过将它们转换为我们应用程序的枚举来简化此过程。

### 3.2 使用MapStruct实现映射

**要配置源常量值到目标常量值的映射，我们使用@ValueMapping MapStruct注解。它根据名称进行映射，但是，我们也可以将源枚举中的常量映射到目标枚举类型中具有不同名称的常量**。例如，我们可以将源枚举“Go”映射到目标枚举“Move”。

**还可以将源枚举中的多个常量映射到目标类型中的相同常量**。

TrafficSignal枚举表示交通信号，我们与之交互的外部服务使用RoadSign枚举，映射器会将枚举相互转换。

让我们定义交通信号枚举：

```java
public enum TrafficSignal {
    Off, Stop, Go
}
```

让我们定义道路标志枚举：

```java
public enum RoadSign {
    Off, Halt, Move
}
```

让我们实现@Mapper：

```java
@Mapper
public interface TrafficSignalMapper {
    TrafficSignalMapper INSTANCE = Mappers.getMapper(TrafficSignalMapper.class);

    @ValueMapping(target = "Off", source = "Off")
    @ValueMapping(target = "Go", source = "Move")
    @ValueMapping(target = "Stop", source = "Halt")
    TrafficSignal toTrafficSignal(RoadSign source);
}
```

@Mapper定义了一个名为TrafficSignalMapper的MapStruct映射器，用于将枚举转换为TrafficSignal，其方法表示映射操作。

**接口中的@ValueMapping注解指定了枚举值之间的显式映射**，例如，@ValueMapping(target= "Go", source= "Move")将Move枚举映射到TrafficSignal中的Go枚举，等等。

**我们需要确保将所有枚举值从源映射到目标以实现完整覆盖并防止意外行为**。

以下是测试：

```java
@Test
void whenRoadSignIsMapped_thenGetTrafficSignal() {
    RoadSign source = RoadSign.Move;
    TrafficSignal target = TrafficSignalMapper.INSTANCE.toTrafficSignal(source);
    assertEquals(TrafficSignal.Go, target);
}
```

它验证了RoadSign.Move到TrafficSignal.Go的映射。

我们必须使用单元测试彻底测试映射方法，以确保准确的行为并检测潜在问题。

## 4. 将字符串映射到枚举

我们将文本文字值转换为枚举值。

### 4.1 理解用例

我们的应用程序以字符串形式收集用户输入，我们将这些字符串映射到枚举值以表示不同的命令或选项。例如，我们将“add”映射到Operation.ADD，“subtract”映射到Operation.SUBTRACT等。

我们在应用程序配置中将设置指定为字符串，我们将这些字符串映射到枚举值以确保类型安全的配置。例如，我们将“EXEC”映射到Mode.EXEC，“TEST”映射到Mode.TEST，等等。

我们将外部API字符串映射到应用程序中的枚举值，例如，我们将“active”映射到Status.ACTIVE，“inactive”映射到Status.INACTIVE，等等。

### 4.2 使用MapStruct实现映射

让我们使用@ValueMapping来映射每个信号：

```java
@ValueMapping(target = "Off", source = "Off")
@ValueMapping(target = "Go", source = "Move")
@ValueMapping(target = "Stop", source = "Halt")
TrafficSignal stringToTrafficSignal(String source);
```

以下是测试：

```java
@Test
void whenStringIsMapped_thenGetTrafficSignal() {
    String source = RoadSign.Move.name();
    TrafficSignal target = TrafficSignalMapper.INSTANCE.stringToTrafficSignal(source);
    assertEquals(TrafficSignal.Go, target);
}
```

它验证“Move”映射到TrafficSignal.Go。

## 5. 处理自定义名称转换

枚举名称可能仅在命名约定方面有所不同，它可能遵循不同的大小写、前缀或后缀约定。例如，信号可以是Go、go、GO、Go_Value、Value_Go。

### 5.1 将后缀应用于源枚举

我们对源枚举应用后缀来获取目标枚举，例如，Go变为Go_Value：

```java
public enum TrafficSignalSuffixed { Off_Value, Stop_Value, Go_Value }
```

让我们定义映射：

```java
@EnumMapping(nameTransformationStrategy = MappingConstants.SUFFIX_TRANSFORMATION, configuration = "_Value")
TrafficSignalSuffixed applySuffix(TrafficSignal source);
```

**@EnumMapping为枚举类型定义自定义映射，nameTransformationStrategy指定在映射之前应用于枚举常量名称的转换策略**，我们在配置中传递适当的控制值。

以下是检查后缀的测试：

```java
@ParameterizedTest
@CsvSource({"Off,Off_Value", "Go,Go_Value"})
void whenTrafficSignalIsMappedWithSuffix_thenGetTrafficSignalSuffixed(TrafficSignal source, TrafficSignalSuffixed expected) {
    TrafficSignalSuffixed result = TrafficSignalMapper.INSTANCE.applySuffix(source);
    assertEquals(expected, result);
}
```

### 5.2 将前缀应用于源枚举

我们还可以在源枚举上添加前缀来获取目标枚举；例如，Go变为Value_Go：

```java
public enum TrafficSignalPrefixed { Value_Off, Value_Stop, Value_Go }
```

让我们定义映射：

```java
@EnumMapping(nameTransformationStrategy = MappingConstants.PREFIX_TRANSFORMATION, configuration = "Value_")
TrafficSignalPrefixed applyPrefix(TrafficSignal source);
```

**PREFIX_TRANSFORMATION告诉MapStruct将前缀“Value_”应用于源枚举**。

让我们检查一下前缀映射：

```java
@ParameterizedTest
@CsvSource({"Off,Value_Off", "Go,Value_Go"})
void whenTrafficSignalIsMappedWithPrefix_thenGetTrafficSignalPrefixed(TrafficSignal source, TrafficSignalPrefixed expected) {
    TrafficSignalPrefixed result = TrafficSignalMapper.INSTANCE.applyPrefix(source);
    assertEquals(expected, result);
}
```

### 5.3 从源枚举中删除后缀

我们从源枚举中删除后缀以获取目标枚举；例如，Go_Value变为Go。

让我们定义映射：

```java
@EnumMapping(nameTransformationStrategy = MappingConstants.STRIP_SUFFIX_TRANSFORMATION, configuration = "_Value")
TrafficSignal stripSuffix(TrafficSignalSuffixed source);
```

**STRIP_SUFFIX_TRANSFORMATION告诉MapStruct从源枚举中删除后缀“_Value”**。

以下是检查删除的后缀的测试：

```java
@ParameterizedTest
@CsvSource({"Off_Value,Off", "Go_Value,Go"})
void whenTrafficSignalSuffixedMappedWithStripped_thenGetTrafficSignal(TrafficSignalSuffixed source, TrafficSignal expected) {
    TrafficSignal result = TrafficSignalMapper.INSTANCE.stripSuffix(source);
    assertEquals(expected, result);
}
```

### 5.4 从源枚举中删除前缀

我们从源枚举中删除一个前缀，得到目标枚举；例如，Value_Go变成Go。

让我们定义映射：

```java
@EnumMapping(nameTransformationStrategy = MappingConstants.STRIP_PREFIX_TRANSFORMATION, configuration = "Value_")
TrafficSignal stripPrefix(TrafficSignalPrefixed source);
```

**STRIP_PREFIX_TRANSFORMATION告诉MapStruct从源枚举中删除前缀“Value_”**。

下面是检查删除前缀的测试：

```java
@ParameterizedTest
@CsvSource({"Value_Off,Off", "Value_Stop,Stop"})
void whenTrafficSignalPrefixedMappedWithStripped_thenGetTrafficSignal(TrafficSignalPrefixed source, TrafficSignal expected) {
    TrafficSignal result = TrafficSignalMapper.INSTANCE.stripPrefix(source);
    assertEquals(expected, result);
}
```

### 5.5 将小写字母应用于源枚举

我们将小写字母应用于源枚举以获得目标枚举；例如，Go变成go：

```java
public enum TrafficSignalLowercase { off, stop, go }
```

让我们定义映射：

```java
@EnumMapping(nameTransformationStrategy = MappingConstants.CASE_TRANSFORMATION, configuration = "lower")
TrafficSignalLowercase applyLowercase(TrafficSignal source);
```

**CASE_TRANSFORMATION和lower配置告诉MapStruct将小写应用于源枚举**。

以下是检查小写映射的测试方法：

```java
@ParameterizedTest
@CsvSource({"Off,off", "Go,go"})
void whenTrafficSignalMappedWithLower_thenGetTrafficSignalLowercase(TrafficSignal source, TrafficSignalLowercase expected) {
    TrafficSignalLowercase result = TrafficSignalMapper.INSTANCE.applyLowercase(source);
    assertEquals(expected, result);
}
```

### 5.6 将大写字母应用于源枚举

我们将源枚举变为大写，得到目标枚举；例如，Mon变为MON：

```java
public enum TrafficSignalUppercase { OFF, STOP, GO }
```

让我们定义映射：

```java
@EnumMapping(nameTransformationStrategy = MappingConstants.CASE_TRANSFORMATION, configuration = "upper")
TrafficSignalUppercase applyUppercase(TrafficSignal source);
```

**CASE_TRANSFORMATION和upper配置告诉MapStruct将大写应用于源枚举**。

以下是验证大写映射的测试：

```java
@ParameterizedTest
@CsvSource({"Off,OFF", "Go,GO"})
void whenTrafficSignalMappedWithUpper_thenGetTrafficSignalUppercase(TrafficSignal source, TrafficSignalUppercase expected) {
    TrafficSignalUppercase result = TrafficSignalMapper.INSTANCE.applyUppercase(source);
    assertEquals(expected, result);
}
```

### 5.7 将大写字母应用于源枚举

我们将首字母大小写应用于源枚举以获取目标枚举；例如，go变成Go：

```java
@EnumMapping(nameTransformationStrategy = MappingConstants.CASE_TRANSFORMATION, configuration = "captial")
TrafficSignal lowercaseToCapital(TrafficSignalLowercase source);
```

**CASE_TRANSFORMATION和capital配置告诉MapStruct将源枚举大写**。

以下是检查大写字母的测试：

```java
@ParameterizedTest
@CsvSource({"OFF_VALUE,Off_Value", "GO_VALUE,Go_Value"})
void whenTrafficSignalUnderscoreMappedWithCapital_thenGetStringCapital(TrafficSignalUnderscore source, String expected) {
    String result = TrafficSignalMapper.INSTANCE.underscoreToCapital(source);
    assertEquals(expected, result);
}
```

## 6. 枚举映射的其他用例

有一些情况我们需要将枚举映射回其他类型，本节我们将讨论这些情况。

### 6.1 将枚举映射到字符串

让我们定义映射：

```java
@ValueMapping(target = "Off", source = "Off")
@ValueMapping(target = "Go", source = "Go")
@ValueMapping(target = "Stop", source = "Stop")
String trafficSignalToString(TrafficSignal source);
```

@ValueMapping将枚举值映射到字符串，例如，我们将Go枚举映射到“Go”字符串值，依此类推。

以下是检查字符串映射的测试：

```java
@Test
void whenTrafficSignalIsMapped_thenGetString() {
    TrafficSignal source = TrafficSignal.Go;
    String targetTrafficSignalStr = TrafficSignalMapper.INSTANCE.trafficSignalToString(source);
    assertEquals("Go", targetTrafficSignalStr);
}
```

它验证映射是否将枚举TrafficSignal.Go映射到字符串文字“Go”。

### 6.2 将枚举映射到整数或其他数字类型

直接映射到整数可能会因多个构造函数而产生歧义，我们添加了一个默认的映射器方法，将枚举转换为整数。此外，我们还可以定义一个具有整数属性的类来解决这个问题。

让我们定义一个包装类：

```java
public class TrafficSignalNumber {
    private Integer number;
    // getters and setters
}
```

让我们使用默认方法将枚举映射到整数：

```java
@Mapping(target = "number", source = ".")
TrafficSignalNumber trafficSignalToTrafficSignalNumber(TrafficSignal source);

default Integer convertTrafficSignalToInteger(TrafficSignal source) {
    Integer result = null;
    switch (source) {
        case Off:
            result = 0;
            break;
        case Stop:
            result = 1;
            break;
        case Go:
            result = 2;
            break;
    }
    return result;
}
```

以下是检查整数结果的测试：

```java
@ParameterizedTest
@CsvSource({"Off,0", "Stop,1"})
void whenTrafficSignalIsMapped_thenGetInt(TrafficSignal source, int expected) {
    Integer targetTrafficSignalInt = TrafficSignalMapper.INSTANCE.convertTrafficSignalToInteger(source);
    TrafficSignalNumber targetTrafficSignalNumber = TrafficSignalMapper.INSTANCE.trafficSignalToTrafficSignalNumber(source);
    assertEquals(expected, targetTrafficSignalInt.intValue());
    assertEquals(expected, targetTrafficSignalNumber.getNumber().intValue());
}
```

## 7. 处理未知枚举值

**我们需要通过设置默认值、处理空值或根据业务逻辑抛出异常来[处理不匹配的枚举值](https://www.baeldung.com/java-mapstruct-enum-unexpected-input-exception)**。

### 7.1 MapStruct因任何未映射的属性而引发异常

如果源枚举在目标类型中没有对应的枚举，MapStruct会引发错误。此外，MapStruct还可以将剩余或未映射的值映射到默认值。

我们有两个仅适用于源的选项：ANY_REMAINING和ANY_UNMAPPED；但是，我们一次只需要使用其中一个选项。

### 7.2 映射剩余属性

**ANY_REMAINING选项将任何具有相同名称的剩余源值映射到默认值**。

让我们定义一个简单的交通信号灯：

```java
public enum SimpleTrafficSignal { Off, On }
```

值得注意的是，它的值数量比TrafficSignal少。但是，MapStruct需要我们映射所有枚举值。

让我们定义映射：

```java
@ValueMapping(target = "Off", source = "Off")
@ValueMapping(target = "On", source = "Go")
@ValueMapping(target = "Off", source = "Stop")
SimpleTrafficSignal toSimpleTrafficSignal(TrafficSignal source);
```

我们明确地映射到Off，如果有许多这样的值，映射这些值会很不方便，我们可能会错过映射一些值，这就是ANY_REMAINING有帮助的地方。

让我们定义映射：

```java
@ValueMapping(target = "On", source = "Go")
@ValueMapping(target = "Off", source = MappingConstants.ANY_REMAINING)
SimpleTrafficSignal toSimpleTrafficSignalWithRemaining(TrafficSignal source);
```

在这里，我们将Go映射到On，然后使用MappingConstants.ANY_REMAINING，我们将任何剩余值映射到Off。

以下是检查剩余映射的测试：

```java
@ParameterizedTest
@CsvSource({"Off,Off", "Go,On", "Stop,Off"})
void whenTrafficSignalIsMappedWithRemaining_thenGetTrafficSignal(TrafficSignal source, SimpleTrafficSignal expected) {
    SimpleTrafficSignal targetTrafficSignal = TrafficSignalMapper.INSTANCE.toSimpleTrafficSignalWithRemaining(source);
    assertEquals(expected, targetTrafficSignal);
}
```

它验证除值Go之外的所有其他值是否都映射到Off。

### 7.3 映射未映射的属性

我们可以指示MapStruct映射未映射的值(不考虑名称)，而不是剩余的值。

让我们定义映射：

```java
@ValueMapping(target = "On", source = "Go")
@ValueMapping(target = "Off", source = MappingConstants.ANY_UNMAPPED)
SimpleTrafficSignal toSimpleTrafficSignalWithUnmapped(TrafficSignal source);
```

以下是检查未映射的映射的测试：

```java
@ParameterizedTest
@CsvSource({"Off,Off", "Go,On", "Stop,Off"})
void whenTrafficSignalIsMappedWithUnmapped_thenGetTrafficSignal(TrafficSignal source, SimpleTrafficSignal expected) {
    SimpleTrafficSignal target = TrafficSignalMapper.INSTANCE.toSimpleTrafficSignalWithUnmapped(source);
    assertEquals(expected, target);
}
```

它验证除值Go之外的所有其他值是否都映射到Off。

### 7.4 处理空值

**MapStruct可以使用NULL关键字处理null源和null目标**。

假设我们需要将null输入映射到Off、Go到On，并将任何其他未映射的值映射到null值。

让我们定义映射：

```java
@ValueMapping(target = "Off", source = MappingConstants.NULL)
@ValueMapping(target = "On", source = "Go")
@ValueMapping(target = MappingConstants.NULL, source = MappingConstants.ANY_UNMAPPED)
SimpleTrafficSignal toSimpleTrafficSignalWithNullHandling(TrafficSignal source);
```

我们使用MappingConstants.NULL将null值设置为目标，它还用于指示null输入。

以下是检查空映射的测试：

```java
@CsvSource({",Off", "Go,On", "Stop,"})
void whenTrafficSignalIsMappedWithNull_thenGetTrafficSignal(TrafficSignal source, SimpleTrafficSignal expected) {
    SimpleTrafficSignal targetTrafficSignal = TrafficSignalMapper.INSTANCE.toSimpleTrafficSignalWithNullHandling(source);
    assertEquals(expected, targetTrafficSignal);
}
```

### 7.5 引发异常

让我们考虑这样一种场景：我们引发一个异常，而不是将其映射到默认值或null。

让我们定义映射：

```java
@ValueMapping(target = "On", source = "Go")
@ValueMapping(target = MappingConstants.THROW_EXCEPTION, source = MappingConstants.ANY_UNMAPPED)
@ValueMapping(target = MappingConstants.THROW_EXCEPTION, source = MappingConstants.NULL)
SimpleTrafficSignal toSimpleTrafficSignalWithExceptionHandling(TrafficSignal source);
```

**我们使用MappingConstants.THROW_EXCEPTION来对任何未映射的输入引发异常**。

以下是检查抛出异常的测试：

```java
@ParameterizedTest
@CsvSource({",", "Go,On", "Stop,"})
void whenTrafficSignalIsMappedWithException_thenGetTrafficSignal(TrafficSignal source, SimpleTrafficSignal expected) {
    if (source == TrafficSignal.Go) {
        SimpleTrafficSignal targetTrafficSignal = TrafficSignalMapper.INSTANCE.toSimpleTrafficSignalWithExceptionHandling(source);
        assertEquals(expected, targetTrafficSignal);
    } else {
        Exception exception = assertThrows(IllegalArgumentException.class, () -> {
            TrafficSignalMapper.INSTANCE.toSimpleTrafficSignalWithExceptionHandling(source);
        });
        assertEquals("Unexpected enum constant: " + source, exception.getMessage());
    }
}
```

它为Stop验证结果是否是异常，或者是预期的信号。

## 8. 总结

在本文中，我们学习了使用MapStruct @ValueMapping在枚举类型和其他数据类型(如字符串或整数)之间进行映射。无论是将一个枚举映射到另一个枚举，还是优雅地处理未知的枚举值，@ValueMapping都能在映射任务中提供灵活性和强度。通过遵循最佳实践并处理空输入和不匹配的值，我们可以提高代码的清晰度和可维护性。