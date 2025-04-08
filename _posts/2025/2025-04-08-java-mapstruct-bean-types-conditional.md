---
layout: post
title:  如何使用MapStruct进行条件映射
category: libraries
copyright: libraries
excerpt: Mapstruct
---

## 1. 简介

**[MapStruct](https://www.baeldung.com/mapstruct)是一种代码生成工具，可简化Java Bean类型之间的映射**。在本文中，我们将探讨如何使用MapStruct进行条件映射，并研究实现它的不同配置。

## 2. 使用MapStruct进行条件映射

在对象之间映射数据时，我们经常发现需要根据某些条件映射属性，MapStruct提供了一些配置选项来实现这一点。

让我们检查一下需要根据一些条件映射属性的目标License对象的实例：

```java
public class License {
    private UUID id;
    private OffsetDateTime startDate;
    private OffsetDateTime endDate;
    private boolean active;
    private boolean renewalRequired;
    private LicenseType licenseType;

    public enum LicenseType {
        INDIVIDUAL, FAMILY
    }
    // getters and setters
}
```

输入LicenseDto包含可选的startDate、endDate和licenseType：

```java
public class LicenseDto {
    private UUID id;
    private LocalDateTime startDate;
    private LocalDateTime endDate;
    private String licenseType;
    // getters and setters
}
```

以下是从LicenseDto到License的映射规则：

- id：如果输入LicenseDto有id
- startDate：如果输入的LicenseDto在请求中没有startDate，我们将startDate设置为当前日期
- endDate：如果输入的LicenseDto在请求中没有endDate，我们将endDate设置为当前日期加一年
- active：如果endDate在未来，我们将其设置为true
- renewingRequired：如果结束日期在接下来的两周内，我们将其设置为true
- licenseType：如果输入licenseType可用并且是预期值INDIVIDUAL或FAMILY之一

让我们探索一下使用MapStruct提供的配置实现这一点的一些方法。

### 2.1 使用表达式

**MapStruct提供了在映射[表达式](https://mapstruct.org/documentation/stable/api/org/mapstruct/Mapping.html#expression())中使用任何有效Java表达式来生成映射的功能**，让我们利用此功能来映射startDate：

```java
@Mapping(target = "startDate", expression = "java(mapStartDate(licenseDto))")
License toLicense(LicenseDto licenseDto);
```

现在我们可以定义方法mapStartDate：

```java
default OffsetDateTime mapStartDate(LicenseDto licenseDto) {
    return licenseDto.getStartDate() != null ? licenseDto.getStartDate().atOffset(ZoneOffset.UTC) : OffsetDateTime.now();
}
```

在编译时，MapStruct生成代码：

```java
@Override
public License toLicense(LicenseDto licenseDto) {
    License license = new License();
    license.setStartDate( mapStartDate(licenseDto) );
    
    // Rest of generated mapping...
    return license;
}
```

或者，不使用此方法，方法内的三元运算可以直接传递到表达式中，因为它是一个有效的Java表达式。

### 2.2 使用条件表达式

与表达式类似，**[条件表达式](https://mapstruct.org/documentation/stable/api/org/mapstruct/Mapping.html#conditionExpression())是MapStruct中的一项功能，允许将基于字符串内的条件表达式的属性映射为Java代码**。生成的代码包含if块内的条件，因此，让我们利用此功能来映射License中的renewingRequired：

```java
@Mapping(target = "renewalRequired", conditionExpression = "java(isEndDateInTwoWeeks(licenseDto))", source = ".")
License toLicense(LicenseDto licenseDto);
```

我们可以在java()方法中传入任何有效的Java布尔表达式，让我们在映射器接口中定义isEndDateInTwoWeeks方法：

```java
default boolean isEndDateInTwoWeeks(LicenseDto licenseDto) {
    return licenseDto.getEndDate() != null && Duration.between(licenseDto.getEndDate(), LocalDateTime.now()).toDays() <= 14;
}
```

在编译时，如果满足以下条件，MapStruct将生成代码来设置renewalRequired：

```java
@Override
public License toLicense(LicenseDto licenseDto) {
    License license = new License();

    if ( isEndDateInTwoWeeks(licenseDto) ) {
        license.setRenewalRequired( isEndDateInTwoWeeks( licenseDto ) );
    }

    // Rest of generated mapping...

    return license;
}
```

当条件匹配时，也可以设置源中属性的值。在这种情况下，映射器将使用源中相应的值填充所需的属性。

### 2.3 使用前/后映射

**在某些情况下，如果我们想通过自定义在映射之前或之后修改对象，我们可以使用MapStruct中的@BeforeMapping和@AfterMapping注解**；让我们使用此功能来映射endDate：

```java
@Mapping(target = "endDate", ignore = true)
License toLicense(LicenseDto licenseDto);
```

我们可以定义[AfterMapping](https://mapstruct.org/documentation/stable/api/org/mapstruct/AfterMapping.html)注解来有条件地映射endDate，这样，我们就可以根据特定的条件来控制映射：

```java
@AfterMapping
default void afterMapping(LicenseDto licenseDto, @MappingTarget License license) {
    OffsetDateTime endDate = licenseDto.getEndDate() != null ? licenseDto.getEndDate()
        .atOffset(ZoneOffset.UTC) : OffsetDateTime.now()
        .plusYears(1);
    license.setEndDate(endDate);
}
```

我们需要将输入LicenseDto和目标License对象作为参数传递给此afterMapping方法。因此，这可确保MapStruct在返回License对象之前生成调用此方法的代码作为映射的最后一步：

```java
@Override
public License toLicense(LicenseDto licenseDto) {
    License license = new License();

    // Rest of generated mapping...

    afterMapping( licenseDto, license );

    return license;
}
```

或者，我们可以使用BeforeMapping注解来实现相同的结果。

### 2.4 使用@Condition

**映射时，我们可以使用[@Condition](https://mapstruct.org/documentation/dev/api/org/mapstruct/Condition.html)向属性添加自定义存在性检查。默认情况下，MapStruct对每个属性执行存在性检查，但如果可用，则优先使用@Condition注解的方法**。

让我们使用这个特性来映射licenseType，输入LicenseDto以String形式接收licenseType，在映射期间，如果它不为空，我们需要将其映射到目标，并解析为预期的枚举INDIVIDUAL或FAMILY之一：

```java
@Condition
default boolean mapsToExpectedLicenseType(String licenseType) {
    try {
        return licenseType != null && License.LicenseType.valueOf(licenseType) != null;
    } catch (IllegalArgumentException e) {
        return false;
    }
}
```

MapStruct在映射licenseType时生成使用此方法mapToExpectedLicenseType()的代码，因为签名字符串与LicenseDto中的licenseType匹配：

```java
@Override
public License toLicense(LicenseDto licenseDto) {
    if ( LicenseMapper.mapsToExpectedLicenseType( licenseDto.getLicenseType() ) ) {
        license.setLicenseType( Enum.valueOf( License.LicenseType.class, licenseDto.getLicenseType() ) );
    }

    // Rest of generated mapping...
    return license;
}
```

## 3. 总结

在本文中，我们探讨了使用MapStruct有条件地在Java Bean类型之间映射属性的不同方法。