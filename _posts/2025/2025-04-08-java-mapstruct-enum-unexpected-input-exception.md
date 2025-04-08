---
layout: post
title:  使用MapStruct枚举时出现意外输入时抛出异常
category: libraries
copyright: libraries
excerpt: Mapstruct
---

## 1. 概述

 在本教程中，我们将了解如何使用[MapStruct](https://www.baeldung.com/mapstruct)将一个[枚举](https://www.baeldung.com/a-guide-to-java-enums)的值映射到另一个枚举的值，我们还将学习如何在另一个枚举中没有相应值时抛出异常。

## 2. MapStruct

**MapStruct是一个简化Java Bean映射的代码生成工具**，可以在[Maven中央仓库](https://mvnrepository.com/artifact/org.mapstruct/mapstruct)中找到MapStruct库的最新版本。

让我们将依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>1.6.0.Beta1</version> 
</dependency>
```

此外，我们需要将commentProcessorPaths添加到maven-compiler-plugin插件中，以便自动生成项目target文件夹中的方法：

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

## 3. 问题介绍

首先，让我们创建源枚举，将其命名为InputLevel，它有三个可能的值：HIGH，MEDIUM和LOW：

```java
enum InputLevel {

    LOW, MEDIUM, HIGH
}
```

现在我们可以添加目标枚举，该枚举仅包含两个值HIGH和LOW：

```java
enum OutputLevel {

    LOW, HIGH
}
```

我们的目标是将InputLevel转换为OutputLevel。例如，输入InputLevel.LOW给出OutputLevel.LOW。**但是，没有与MEDIUM匹配的值**。因此，我们想在这种情况下抛出异常。

## 4. 当源没有对应目标时抛出异常

我们将使用Mapper标注并创建一个Mapper接口，**从MapStruct库的1.5.0.Beta1版本开始，我们可以使用@ValueMapping注解来实现我们的目标**：

```java
@Mapper
interface LevelMapper {

    @ValueMapping(source = MappingConstants.ANY_REMAINING, target = MappingConstants.THROW_EXCEPTION)
    OutputLevel inputLevelToOutputLevel(InputLevel inputLevel);
}
```

如我们所见，我们配置了注解，以便在源中映射任何没有对应目标的值都会导致异常。

现在让我们快速检查一下当给定InputLevel.HIGH时该方法是否正确返回OutputLevel.HIGH：

```java
LevelMapper levelMapper = Mappers.getMapper(LevelMapper.class);

@Test
void givenHighInputLevel_WhenInputLevelToOutputLevel_ThenHighOutputLevel() {
    assertEquals(OutputLevel.HIGH, levelMapper.inputLevelToOutputLevel(InputLevel.HIGH));
}
```

最后，我们确认一下，当我们尝试将InputLevel.MEDIUM转换为OutputLevel时，会抛出异常。具体来说，它会抛出一个IllegalArgumentException：

```java
@Test
void givenMediumInputLevel_WhenInputLevelToOutputLevel_ThenThrows() {
    assertThrows(IllegalArgumentException.class, () -> levelMapper.inputLevelToOutputLevel(InputLevel.MEDIUM));
}
```

## 5. 总结

在本文中，我们使用MapStruct库将源枚举中的值映射到目标枚举。此外，我们将映射器配置为在源值与目标枚举不匹配时抛出异常。