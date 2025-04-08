---
layout: post
title:  使用MapStruct和Lombok
category: libraries
copyright: libraries
excerpt: Mapstruct
---

## 1. 概述

[Project Lombok](https://www.baeldung.com/intro-to-project-lombok)是一个帮助使用Java编写样板代码的库，它使我们能够更加专注于核心应用程序逻辑。同样，[MapStruct](https://www.baeldung.com/mapstruct)是另一个在两个Java Bean之间进行映射时可以减少样板代码的库。

因为它们都使用了注解处理器(一种高级Java功能)，所以在设置项目时很容易出错。

在本教程中，我们将介绍如何正确配置Maven，然后在最后展示一个可以运行的项目。

## 2. 类路径

### 2.1 依赖

让我们将[mapstruct](https://mvnrepository.com/artifact/org.mapstruct/mapstruct)和[lombok](https://mvnrepository.com/artifact/org.projectlombok/lombok)依赖添加到项目的pom.xml文件中：

```xml
<dependencies>
    <dependency>
        <groupId>org.mapstruct</groupId>
        <artifactId>mapstruct</artifactId>
        <version>1.6.3</version>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.32</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

为Lombok依赖设置的provided[依赖范围](https://www.baeldung.com/maven-dependency-scopes)意味着它们仅在编译时使用，并且不应该出现在运行时类路径中。

### 2.2 注解处理器

我们还需要在maven-compiler-plugin声明中使用[mapstruct-processor](https://mvnrepository.com/artifact/org.mapstruct/mapstruct-processor)和[lombok-mapstruct-binding](https://mvnrepository.com/artifact/org.projectlombok/lombok-mapstruct-binding)配置注解处理器路径：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
        <annotationProcessorPaths>
            <path>
                <groupId>org.mapstruct</groupId>
                <artifactId>mapstruct-processor</artifactId>
                <version>1.6.3</version>
            </path>
            <path>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>1.18.32</version>
            </path>
            <path>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok-mapstruct-binding</artifactId>
                <version>0.2.0</version>
            </path>
        </annotationProcessorPaths>
    </configuration>
</plugin>
```

这里很容易犯错误，因为注解处理器是一项高级功能，**主要错误是忘记我们的运行时将搜索注解处理器路径或注解处理器的类路径，但不会同时搜索两者**。

需要注意的是，对于Lombok 1.18.16及以上版本，我们需要将Lombok lombok-mapstruct-binding和mapstruct-processor依赖添加到commentProcessorPaths元素中，如果不这样做，我们可能会收到编译错误：“Unknown Property in Result Type...”。

我们需要lombok-mapstruct-binding依赖才能使Lombok和MapStruct协同工作，本质上，它告诉MapStruct等到Lombok完成所有注解处理后再为Lombok增强Bean生成映射器类。

## 3. MapStruct与Lombok集成

我们将在设置中使用@Builder和@Data Lombok注解，**前者允许通过[构建器模式](https://www.baeldung.com/java-builder-pattern)创建对象，而后者通过Setter提供基于构造函数的对象创建**。

### 3.1 Java POJO设置

现在，让我们首先为映射器定义一个简单的源类：

```java
@Data
public class SimpleSource {
    private String name;
    private String description;
}
```

接下来，我们为映射器定义一个简单的目标类：

```java
@Data
public class SimpleDestination {
    private String name;
    private String description;
}
```

最后，我们还将定义另一个目标类，但使用@Builder Lombok注解：

```java
@Builder
@Getter
public class LombokDestination {
    private String name;
    private String description;
}
```

### 3.2 使用@Mapper注解

**当我们使用@Mapper注解时，MapStruct会自动创建映射器实现**。

让我们定义映射器接口：

```java
@Mapper
public interface LombokMapper {
    SimpleDestination sourceToDestination(SimpleSource source);
    LombokDestination sourceToLombokDestination(SimpleSource source);
}
```

当我们执行mvn clean install命令时，会在/target/generated-sources/annotations/文件夹下创建Mapper实现类。

我们来看一下生成的实现类：

```java
public class LombokMapperImpl implements LombokMapper {

    @Override
    public SimpleDestination sourceToDestination(SimpleSource source) {
        if ( source == null ) {
            return null;
        }

        SimpleDestination simpleDestination = new SimpleDestination();

        simpleDestination.setName( source.getName() );
        simpleDestination.setDescription( source.getDescription() );

        return simpleDestination;
    }

    @Override
    public LombokDestination sourceToLombokDestination(SimpleSource source) {
        if ( source == null ) {
            return null;
        }

        LombokDestination.LombokDestinationBuilder lombokDestination = LombokDestination.builder();

        lombokDestination.name( source.getName() );
        lombokDestination.description( source.getDescription() );

        return lombokDestination.build();
    }
}
```

正如我们在这里看到的，实现有两种方法可以将源对象映射到不同的目标。但是，主要的区别在于目标对象的构建方式。

**该实现使用LombokDestination类的builder()方法，另一方面，它使用构造函数创建SimpleDestination对象并使用Setter映射变量**。

### 3.3 测试用例

现在，让我们看一个简单的测试用例来观察映射器的实际实现：

```java
@Test
void whenDestinationIsMapped_thenIsSuccessful() {
    SimpleSource simpleSource = new SimpleSource();
    simpleSource.setName("file");
    simpleSource.setDescription("A text file.");

    SimpleDestination simpleDestination = lombokMapper.sourceToDestination(simpleSource);
    Assertions.assertNotNull(simpleDestination);
    Assertions.assertEquals(simpleSource.getName(), simpleDestination.getName());
    Assertions.assertEquals(simpleSource.getDescription(), simpleDestination.getDescription());

    LombokDestination lombokDestination = lombokMapper.sourceToLombokDestination(simpleSource);
    Assertions.assertNotNull(lombokDestination);
    Assertions.assertEquals(simpleSource.getName(), lombokDestination.getName());
    Assertions.assertEquals(simpleSource.getDescription(), lombokDestination.getDescription());
}
```

正如我们可以在上面的测试用例中验证的那样，映射器实现成功地将源POJO映射到两个目标POJO。

## 4. 总结

在本文中，我们研究了如何结合使用MapStruct和Lombok来帮助我们减少样板代码的编写，从而增强代码的可读性并提高开发过程的效率。

我们研究了如何在从一个POJO映射到另一个POJO时将@Builder和@Data Lombok注解与MapStruct结合使用。