---
layout: post
title:  使用MapStruct将枚举映射到字符串
category: libraries
copyright: libraries
excerpt: Mapstruct
---

## 1. 简介

[MapStruct](https://www.baeldung.com/mapstruct)是一个高效、类型安全的库，它简化了Java对象之间的数据映射，无需手动转换逻辑。

在本教程中，我们将探讨使用[MapStruct](http://www.mapstruct.org/)将[枚举](https://www.baeldung.com/a-guide-to-java-enums)映射到字符串。

## 2. 将枚举映射到字符串

使用Java枚举作为字符串而不是序数简化了与外部API的数据交换，使数据检索更容易并增强了UI的可读性。

假设我们要将DayOfWeek枚举转换为字符串。

[DayOfWeek](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/time/DayOfWeek.html)是Java [Date Time API](https://www.baeldung.com/java-8-date-time-intro)中的一个枚举，代表一周的七天，从星期一到星期日。

让我们实现MapStruct映射器：

```java
@Mapper
public interface DayOfWeekMapper {
    DayOfWeekMapper INSTANCE = Mappers.getMapper(DayOfWeekMapper.class);

    String toString(DayOfWeek dayOfWeek);

    // additional mapping methods as needed
}
```

DayOfWeekMapper接口是一个MapStruct映射器，由@Mapper指定。我们定义toString()方法，该方法接收DayOfWeek枚举并将其转换为字符串表示形式。**默认情况下，MapStruct使用name()方法获取枚举的字符串值**：

```java
class DayOfWeekMapperUnitTest {
    private DayOfWeekMapper dayOfWeekMapper = DayOfWeekMapper.INSTANCE;

    @ParameterizedTest
    @CsvSource({"MONDAY,MONDAY", "TUESDAY,TUESDAY", "WEDNESDAY,WEDNESDAY", "THURSDAY,THURSDAY",
                "FRIDAY,FRIDAY", "SATURDAY,SATURDAY", "SUNDAY,SUNDAY"})
    void whenDayOfWeekMapped_thenGetsNameString(DayOfWeek source, String expected) {
        String target = dayOfWeekMapper.toString(source);
        assertEquals(expected, target);
    }
}
```

这验证了toString()将DayOfWeek枚举值映射到其预期的字符串名称，这种类型的[参数化测试](https://www.baeldung.com/parameterized-tests-junit-5)还允许我们通过单个测试测试所有可能的变异。

### 2.1 处理Null

现在，让我们看看MapStruct如何处理null。**默认情况下，MapStruct将null源映射到null目标**。但是，此行为可以修改。

让我们验证映射器是否为null输入返回null结果：

```java
@Test
void whenNullDayOfWeekMapped_thenGetsNullResult() {
    String target = dayOfWeekMapper.toString(null);
    assertNull(target);
}
```

**MapStruct提供MappingConstants.NULL来管理null值**：

```java
@Mapper
public interface DayOfWeekMapper {

    @ValueMapping(target = "MONDAY", source = MappingConstants.NULL)
    String toStringWithDefault(DayOfWeek dayOfWeek);
}
```

对于null值，此映射返回默认值MONDAY：

```java
@Test
void whenNullDayOfWeekMappedWithDefaults_thenReturnsDefault() {
    String target = dayOfWeekMapper.toStringWithDefault(null);
    assertEquals("MONDAY", target);
}
```

## 3. 将字符串映射到枚举

现在，让我们看一下将字符串转换回枚举的映射器方法：

```java
@Mapper
public interface DayOfWeekMapper {

    DayOfWeek nameStringToDayOfWeek(String day);
}
```

该映射器将代表星期几的字符串转换为相应的DayOfWeek枚举值：

```java
@ParameterizedTest
@CsvSource(
        {"MONDAY,MONDAY", "TUESDAY,TUESDAY", "WEDNESDAY,WEDNESDAY", "THURSDAY,THURSDAY",
         "FRIDAY,FRIDAY", "SATURDAY,SATURDAY", "SUNDAY,SUNDAY"})
void whenNameStringMapped_thenGetsDayOfWeek(String source, DayOfWeek expected) {
    DayOfWeek target = dayOfWeekMapper.nameStringToDayOfWeek(source);
    assertEquals(expected, target);
}
```

我们验证nameStringToDayOfWeek()将某天的字符串表示形式映射到其对应的枚举。

### 3.1 处理未映射的值

如果字符串与枚举名称或通过@ValueMapping的另一个常量不匹配，MapStruct将抛出错误。通常，这样做是为了确保所有值都安全且可预测地映射。**如果出现无法识别的源值，MapStruct创建的映射方法将抛出IllegalStateException**：

```java
@Test
void whenInvalidNameStringMapped_thenThrowsIllegalArgumentException() {
    String source = "Mon";
    IllegalArgumentException exception = assertThrows(IllegalArgumentException.class, () -> {
        dayOfWeekMapper.nameStringToDayOfWeek(source);
    });
    assertTrue(exception.getMessage().equals("Unexpected enum constant: " + source));
}
```

为了改变这种行为，MapStruct还提供了MappingConstants.ANY_UNMAPPED，**这指示MapStruct将任何未映射的源值映射到目标常量值**：

```java
@Mapper
public interface DayOfWeekMapper {

    @ValueMapping(target = "MONDAY", source = MappingConstants.ANY_UNMAPPED)
    DayOfWeek nameStringToDayOfWeekWithDefaults(String day);
}
```

最后，此@ValueMapping注解设置未映射源的默认行为。因此，任何未映射的输入都默认为MONDAY：

```java
@ParameterizedTest
@CsvSource({"Mon,MONDAY"})
void whenInvalidNameStringMappedWithDefaults_thenReturnsDefault(String source, DayOfWeek expected) {
    DayOfWeek target = dayOfWeekMapper.nameStringToDayOfWeekWithDefaults(source);
    assertEquals(expected, target);
}
```

## 4. 将枚举映射到自定义字符串

现在，我们将枚举转换为DayOfWeek的自定义简短表示，如“Mon”、“Tue”等：

```java
@Mapper
public interface DayOfWeekMapper {

    @ValueMapping(target = "Mon", source = "MONDAY")
    @ValueMapping(target = "Tue", source = "TUESDAY")
    @ValueMapping(target = "Wed", source = "WEDNESDAY")
    @ValueMapping(target = "Thu", source = "THURSDAY")
    @ValueMapping(target = "Fri", source = "FRIDAY")
    @ValueMapping(target = "Sat", source = "SATURDAY")
    @ValueMapping(target = "Sun", source = "SUNDAY")
    String toShortString(DayOfWeek dayOfWeek);
}
```

相反，这个toShortString()映射配置使用@ValueMapping将DayOfWeek枚举转换为简短字符串：

```java
@ParameterizedTest
@CsvSource(
        {"MONDAY,Mon", "TUESDAY,Tue", "WEDNESDAY,Wed", "THURSDAY,Thu",
         "FRIDAY,Fri", "SATURDAY,Sat", "SUNDAY,Sun"})
void whenDayOfWeekMapped_thenGetsShortString(DayOfWeek source, String expected) {
    String target = dayOfWeekMapper.toShortString(source);
    assertEquals(expected, target);
}
```

## 5. 将自定义字符串映射到枚举

最后，我们看看如何将缩写字符串转换为DayOfWeek枚举：

```java
@Mapper
public interface DayOfWeekMapper {

    @InheritInverseConfiguration(name = "toShortString")
    DayOfWeek shortStringToDayOfWeek(String day);
}
```

此外，**@InheritInverseConfiguration注解定义了一个反向映射**，它允许shortStringToDayOfWeek()从toShortString()方法继承其配置，将缩写的星期名称转换为相应的DayOfWeek枚举：

```java
@ParameterizedTest
@CsvSource(
        {"Mon,MONDAY", "Tue,TUESDAY", "Wed,WEDNESDAY", "Thu,THURSDAY",
         "Fri,FRIDAY", "Sat,SATURDAY", "Sun,SUNDAY"})
void whenShortStringMapped_thenGetsDayOfWeek(String source, DayOfWeek expected) {
    DayOfWeek target = dayOfWeekMapper.shortStringToDayOfWeek(source);
    assertEquals(expected, target);
}
```

## 6. 总结

在本文中，我们学习了如何使用MapStruct的@ValueMapping注解将枚举映射到字符串，我们还使用了@InheritInverseConfiguration来在来回映射时保持一致性。使用这些技巧，我们可以顺利地处理枚举到字符串的转换，并使我们的代码清晰易维护。