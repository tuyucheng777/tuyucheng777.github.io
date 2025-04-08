---
layout: post
title:  在Spring Boot中将不区分大小写的@Value绑定到枚举
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 概述

Spring为我们提供了自动配置功能，我们可以使用它来绑定组件、配置Bean以及从属性源设置值。

当我们不想对值进行硬编码并希望使用属性文件或系统环境提供它们时，[@Value](https://www.baeldung.com/spring-value-annotation)注解非常有用。

在本教程中，我们将学习如何利用Spring自动配置将这些值映射到[Enum](https://www.baeldung.com/a-guide-to-java-enums)实例。

## 2. Converters<F,T\>

Spring使用转换器将@Value中的字符串值映射到所需类型，专用的BeanPostProcessor会检查所有组件，并检查它们是否需要额外配置，或者在我们的例子中是否需要注入。**之后，找到合适的转换器，并将源转换器中的数据发送到指定的目标**。Spring提供了一个现成的字符串到枚举转换器，让我们来回顾一下。

### 2.1 LenientToEnumConverter

顾名思义，该转换器可以在转换过程中非常自由地解释数据。最初，它假设提供的值正确：

```java
@Override
public E convert(T source) {
    String value = source.toString().trim();
    if (value.isEmpty()) {
        return null;
    }
    try {
        return (E) Enum.valueOf(this.enumType, value);
    }
    catch (Exception ex) {
        return findEnum(value);
    }
}
```

但是，如果无法将源映射到枚举，它会尝试不同的方法。它获取Enum的规范名称和值：

```java
private E findEnum(String value) {
    String name = getCanonicalName(value);
    List<String> aliases = ALIASES.getOrDefault(name, Collections.emptyList());
    for (E candidate : (Set<E>) EnumSet.allOf(this.enumType)) {
        String candidateName = getCanonicalName(candidate.name());
        if (name.equals(candidateName) || aliases.contains(candidateName)) {
            return candidate;
        }
    }
    throw new IllegalArgumentException("No enum constant " + this.enumType.getCanonicalName() + "." + value);
}
```

getCanonicalName(String)会过滤掉所有特殊字符并将字符串转换为小写：

```java
private String getCanonicalName(String name) {
    StringBuilder canonicalName = new StringBuilder(name.length());
    name.chars()
        .filter(Character::isLetterOrDigit)
        .map(Character::toLowerCase)
        .forEach((c) -> canonicalName.append((char) c));
    return canonicalName.toString();
}
```

**这个过程使得转换器具有很强的适应性，因此如果不考虑的话可能会引入一些问题**。同时，它免费为Enum提供对不区分大小写的匹配的出色支持，无需任何额外配置。

### 2.2 宽松的转换

让我们以一个简单的Enum类为例：

```java
public enum SimpleWeekDays {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}
```

我们将使用@Value注解将所有这些常量注入到专用的类持有者中：

```java
@Component
public class WeekDaysHolder {
    @Value("${monday}")
    private WeekDays monday;
    @Value("${tuesday}")
    private WeekDays tuesday;
    @Value("${wednesday}")
    private WeekDays wednesday;
    @Value("${thursday}")
    private WeekDays thursday;
    @Value("${friday}")
    private WeekDays friday;
    @Value("${saturday}")
    private WeekDays saturday;
    @Value("${sunday}")
    private WeekDays sunday;
    // getters and setters
}
```

使用宽松的转换，我们不仅可以使用不同的大小写传递值，而且如之前所示，我们可以在这些值的周围和内部添加特殊字符，并且转换器仍然会映射它们：

```java
@SpringBootTest(properties = {
        "monday=Mon-Day!",
        "tuesday=TuesDAY#",
        "wednesday=Wednes@day",
        "thursday=THURSday^",
        "friday=Fri:Day_%",
        "saturday=Satur_DAY*",
        "sunday=Sun+Day",
}, classes = WeekDaysHolder.class)
class LenientStringToEnumConverterUnitTest {
    @Autowired
    private WeekDaysHolder propertyHolder;

    @ParameterizedTest
    @ArgumentsSource(WeekDayHolderArgumentsProvider.class)
    void givenPropertiesWhenInjectEnumThenValueIsPresent(
            Function<WeekDaysHolder, WeekDays> methodReference, WeekDays expected) {
        WeekDays actual = methodReference.apply(propertyHolder);
        assertThat(actual).isEqualTo(expected);
    }
}
```

这不一定是一件好事，特别是如果它对开发人员隐藏的话。**不正确的假设可能会产生难以识别的微妙问题**。

### 2.3 极其宽松的转换

同时，这种类型的转换对双方都有效，即使我们打破所有命名约定并使用如下内容也不会失败：

```java
public enum NonConventionalWeekDays {
    Mon$Day, Tues$DAY_, Wednes$day, THURS$day_, Fri$Day$_$, Satur$DAY_, Sun$Day
}
```

这种情况的问题是它可能会产生正确的结果并将所有值映射到其专用枚举：

```java
@SpringBootTest(properties = {
        "monday=Mon-Day!",
        "tuesday=TuesDAY#",
        "wednesday=Wednes@day",
        "thursday=THURSday^",
        "friday=Fri:Day_%",
        "saturday=Satur_DAY*",
        "sunday=Sun+Day",
}, classes = NonConventionalWeekDaysHolder.class)
class NonConventionalStringToEnumLenientConverterUnitTest {
    @Autowired
    private NonConventionalWeekDaysHolder holder;

    @ParameterizedTest
    @ArgumentsSource(NonConventionalWeekDayHolderArgumentsProvider.class)
    void givenPropertiesWhenInjectEnumThenValueIsPresent(
            Function<NonConventionalWeekDaysHolder, NonConventionalWeekDays> methodReference, NonConventionalWeekDays expected) {
        NonConventionalWeekDays actual = methodReference.apply(holder);
        assertThat(actual).isEqualTo(expected);
    }
}
```

**将“Mon-Day!”映射到“Mon$Day”而不失败可能会隐藏问题，并建议开发人员跳过既定的惯例**。虽然它适用于不区分大小写的映射，但这些假设太过轻率。

## 3. 自定义转换器

在映射过程中解决特定规则的最佳方法是创建转换器的实现，见证了LenientToEnumConverter的功能后，让我们退一步，创建一些更具限制性的东西。

### 3.1 StrictNullableWeekDayConverter

想象一下，仅当属性正确识别其名称时，我们才决定将值映射到枚举。这可能会导致一些不遵守大写约定的初始问题，但总的来说，这是一个万无一失的解决方案：

```java
public class StrictNullableWeekDayConverter implements Converter<String, WeekDays> {
    @Override
    public WeekDays convert(String source) {
        try {
            return WeekDays.valueOf(source.trim());
        } catch (IllegalArgumentException e) {
            return null;
        }
    }
}
```

此转换器将对源字符串进行细微调整。在这里，我们唯一要做的就是修剪值周围的空白。**此外，请注意，返回null并不是最佳设计决策，因为它会允许在不正确的状态下创建上下文**。但是，我们在这里使用null来简化测试：

```java
@SpringBootTest(properties = {
        "monday=monday",
        "tuesday=tuesday",
        "wednesday=wednesday",
        "thursday=thursday",
        "friday=friday",
        "saturday=saturday",
        "sunday=sunday",
}, classes = {WeekDaysHolder.class, WeekDayConverterConfiguration.class})
class StrictStringToEnumConverterNegativeUnitTest {
    public static class WeekDayConverterConfiguration {
        // configuration
    }

    @Autowired
    private WeekDaysHolder holder;

    @ParameterizedTest
    @ArgumentsSource(WeekDayHolderArgumentsProvider.class)
    void givenPropertiesWhenInjectEnumThenValueIsNull(
            Function<WeekDaysHolder, WeekDays> methodReference, WeekDays ignored) {
        WeekDays actual = methodReference.apply(holder);
        assertThat(actual).isNull();
    }
}
```

同时，如果我们提供大写的值，则会注入正确的值。要使用这个转换器，我们需要告诉Spring：

```java
public static class WeekDayConverterConfiguration {
    @Bean
    public ConversionService conversionService() {
        DefaultConversionService defaultConversionService = new DefaultConversionService();
        defaultConversionService.addConverter(new StrictNullableWeekDayConverter());
        return defaultConversionService;
    }
}
```

在某些Spring Boot版本或配置中，类似的转换器可能是默认转换器，这比LenientToEnumConverter更有意义。

### 3.2 CaseInsensitiveWeekDayConverter

让我们找到一个愉快的中间立场，我们将能够使用不区分大小写的匹配，但同时不允许任何其他差异：

```java
public class CaseInsensitiveWeekDayConverter implements Converter<String, WeekDays> {
    @Override
    public WeekDays convert(String source) {
        try {
            return WeekDays.valueOf(source.trim());
        } catch (IllegalArgumentException exception) {
            return WeekDays.valueOf(source.trim().toUpperCase());
        }
    }
}
```

我们不考虑Enum名称不是大写或使用混合大小写的情况，**但是，这种情况是可以解决的，只需要额外的几行代码和try-catch块**。我们可以为Enum创建一个查找映射并将其缓存，但让我们这样做吧。

测试看起来相似并且会正确映射值。为简单起见，我们只检查使用此转换器正确映射的属性：

```java
@SpringBootTest(properties = {
        "monday=monday",
        "tuesday=tuesday",
        "wednesday=wednesday",
        "thursday=THURSDAY",
        "friday=Friday",
        "saturday=saturDAY",
        "sunday=sUndAy",
}, classes = {WeekDaysHolder.class, WeekDayConverterConfiguration.class})
class CaseInsensitiveStringToEnumConverterUnitTest {
    // ...
}
```

使用自定义转换器，我们可以根据我们的需求或我们想要遵循的约定来调整映射过程。

## 4. SpEL

[SpEL](https://www.baeldung.com/spring-expression-language)是一个功能强大的工具，几乎可以做任何事情。**在我们的问题中，我们将尝试在尝试映射Enum之前调整从属性文件接收的值**。为此，我们可以明确将提供的值更改为大写：

```java
@Component
public class SpELWeekDaysHolder {
    @Value("#{'${monday}'.toUpperCase()}")
    private WeekDays monday;
    @Value("#{'${tuesday}'.toUpperCase()}")
    private WeekDays tuesday;
    @Value("#{'${wednesday}'.toUpperCase()}")
    private WeekDays wednesday;
    @Value("#{'${thursday}'.toUpperCase()}")
    private WeekDays thursday;
    @Value("#{'${friday}'.toUpperCase()}")
    private WeekDays friday;
    @Value("#{'${saturday}'.toUpperCase()}")
    private WeekDays saturday;
    @Value("#{'${sunday}'.toUpperCase()}")
    private WeekDays sunday;

    // getters and setters
}
```

要检查值是否正确映射，我们可以使用之前创建的 StrictNullableWeekDayConverter：

```java
@SpringBootTest(properties = {
        "monday=monday",
        "tuesday=tuesday",
        "wednesday=wednesday",
        "thursday=THURSDAY",
        "friday=Friday",
        "saturday=saturDAY",
        "sunday=sUndAy",
}, classes = {SpELWeekDaysHolder.class, WeekDayConverterConfiguration.class})
class SpELCaseInsensitiveStringToEnumConverterUnitTest {
    public static class WeekDayConverterConfiguration {
        @Bean
        public ConversionService conversionService() {
            DefaultConversionService defaultConversionService = new DefaultConversionService();
            defaultConversionService.addConverter(new StrictNullableWeekDayConverter());
            return defaultConversionService;
        }
    }

    @Autowired
    private SpELWeekDaysHolder holder;

    @ParameterizedTest
    @ArgumentsSource(SpELWeekDayHolderArgumentsProvider.class)
    void givenPropertiesWhenInjectEnumThenValueIsNull(
            Function<SpELWeekDaysHolder, WeekDays> methodReference, WeekDays expected) {
        WeekDays actual = methodReference.apply(holder);
        assertThat(actual).isEqualTo(expected);
    }
}
```

虽然转换器仅理解大写值，但通过使用SpEL，我们可以将属性转换为正确的格式。**此技术可能有助于简单的翻译和映射，因为它直接出现在@Value注解中，并且相对而言易于使用**。但是，请避免将大量复杂逻辑放入SPEL中。

## 5. 总结

@Value注解功能强大且灵活，支持SPEL和属性注入。自定义转换器可能会使其更加强大，允许我们将其与自定义类型一起使用或实现特定约定。