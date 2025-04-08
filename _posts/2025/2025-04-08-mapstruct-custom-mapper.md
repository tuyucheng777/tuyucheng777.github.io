---
layout: post
title:  使用MapStruct的自定义映射器
category: libraries
copyright: libraries
excerpt: Mapstruct
---

## 1. 概述

在本教程中，我们将学习如何使用[MapStruct](https://www.baeldung.com/mapstruct)库的自定义映射器。

**MapStruct库用于Java Bean类型之间的映射**，通过在MapStruct中使用自定义映射器，**我们可以自定义默认映射方法**。

## 2. Maven依赖

我们将[mapstruct](https://mvnrepository.com/artifact/org.mapstruct/mapstruct)库添加到Maven pom.xml中：

```xml
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>1.6.0.Beta1</version> 
</dependency>
```

要查看项目target文件夹内的自动生成的方法，我们必须将commentProcessorPaths添加到maven-compiler-plugin插件：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.5.1</version>
    <configuration>
        <source>1.8</source>
        <target>1.8</target>
        <annotationProcessorPaths>
            <path>
                <groupId>org.mapstruct</groupId>
                <artifactId>mapstruct</artifactId>
                <version>1.6.0.Beta1</version>
            </path>
        </annotationProcessorPaths>
    </configuration>
</plugin>
```

## 3. 自定义映射器

自定义映射器用于解决特定的转换要求。为此，我们必须定义一个方法来进行转换，然后我们必须将该方法通知MapStruct。最后，MapStruct将调用该方法进行从源到目标的转换。

例如，假设我们有一个计算用户身体质量指数(BMI)报告的应用。要计算BMI，我们必须收集用户的身体值。要将英制单位转换为公制单位，我们可以使用自定义映射器方法。

有两种方法可以在MapStruct中使用自定义映射器，**我们可以通过在@Mapping注解的eligibleByName属性中指定自定义方法来调用它，也可以为其创建注解**。

在开始之前，我们必须定义一个DTO类来保存英制值：

```java
public class UserBodyImperialValuesDTO {
    private int inch;
    private int pound;
    // constructor, getters, and setters
}
```

接下来，我们将定义一个DTO类来保存公制值：

```java
public class UserBodyValues {
    private double kilogram;
    private double centimeter;
    // constructor, getters, and setters
}
```

### 3.1 使用方法的自定义映射器

要开始使用自定义映射器，我们将使用@Mapper注解创建一个接口：

```java
@Mapper 
public interface UserBodyValuesMapper {
    // ...
}
```

然后，我们将使用想要的返回类型和需要转换的参数创建自定义方法，我们必须使用带有value参数的@Named注解来通知MapStruct有关自定义映射器方法的信息：

```java
@Mapper
public interface UserBodyValuesMapper {

    @Named("inchToCentimeter")
    public static double inchToCentimeter(int inch) {
        return inch * 2.54;
    }
 
    // ...
}
```

最后，我们将使用@Mapping注解定义映射器接口方法。在此注解中，我们将告诉MapStruct源类型、目标类型以及它将使用的方法：

```java
@Mapper
public interface UserBodyValuesMapper {
    UserBodyValuesMapper INSTANCE = Mappers.getMapper(UserBodyValuesMapper.class);
    
    @Mapping(source = "inch", target = "centimeter", qualifiedByName = "inchToCentimeter")
    public UserBodyValues userBodyValuesMapper(UserBodyImperialValuesDTO dto);
    
    @Named("inchToCentimeter") 
    public static double inchToCentimeter(int inch) { 
        return inch * 2.54; 
    }
}
```

让我们测试我们的自定义映射器：

```java
UserBodyImperialValuesDTO dto = new UserBodyImperialValuesDTO();
dto.setInch(10);

UserBodyValues obj = UserBodyValuesMapper.INSTANCE.userBodyValuesMapper(dto);

assertNotNull(obj);
assertEquals(25.4, obj.getCentimeter(), 0);
```

### 3.2 使用注解的自定义映射器

要使用带有注解的自定义映射器，我们必须定义一个注解而不是使用@Named注解，然后我们必须通过指定@Mapping注解的qualifiedByName参数来通知MapStruct有关新创建的注解的信息。

让我们看看如何定义注解：

```java
@Qualifier
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.CLASS)
public @interface PoundToKilogramMapper {
}
```

我们将在poundToKilogram方法中添加@PoundToKilogramMapper注解：

```java
@PoundToKilogramMapper
public static double poundToKilogram(int pound) {
    return pound * 0.4535;
}
```

现在我们将使用@Mapping注解定义映射器接口方法，在映射注解中，我们将告诉MapStruct它将使用的源类型、目标类型和注解类：

```java
@Mapper
public interface UserBodyValuesMapper {
    UserBodyValuesMapper INSTANCE = Mappers.getMapper(UserBodyValuesMapper.class);

    @Mapping(source = "pound", target = "kilogram", qualifiedBy = PoundToKilogramMapper.class)
    public UserBodyValues userBodyValuesMapper(UserBodyImperialValuesDTO dto);

    @PoundToKilogramMapper
    public static double poundToKilogram(int pound) {
        return pound * 0.4535;
    }
}
```

最后，测试我们的自定义映射器：

```java
UserBodyImperialValuesDTO dto = new UserBodyImperialValuesDTO();
dto.setPound(100);

UserBodyValues obj = UserBodyValuesMapper.INSTANCE.userBodyValuesMapper(dto);

assertNotNull(obj);
assertEquals(45.35, obj.getKilogram(), 0);

```

## 4. 总结

在本文中，我们演示了如何使用MapStruct库实现自定义映射器。