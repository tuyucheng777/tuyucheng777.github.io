---
layout: post
title:  使用MapStruct忽略未映射的属性
category: libraries
copyright: libraries
excerpt: Mapstruct
---

## 1. 概述

在Java应用程序中，**我们可能希望将值从一种类型的Java Bean到另一种类型的Java Bean**，为了避免冗长且容易出错的代码，我们可以使用Bean映射器，例如[MapStruct](https://www.baeldung.com/mapstruct)。

虽然映射具有相同字段名称的相同字段非常简单，但我们经常会遇到不匹配的Bean。在本教程中，我们将了解MapStruct如何处理部分映射。

## 2. 映射

MapStruct是一个Java注解处理器，因此，我们需要做的就是定义映射器接口并声明映射方法，**MapStruct将在编译期间生成此接口的实现**。

为了简单起见，让我们从两个具有相同字段名称的类开始：

```java
public class CarDTO {
    private int id;
    private String name;
}
```

```java
public class Car {
    private int id;
    private String name;
}
```

接下来我们创建一个映射器接口：

```java
@Mapper
public interface CarMapper {
    CarMapper INSTANCE = Mappers.getMapper(CarMapper.class);
    CarDTO carToCarDTO(Car car);
}
```

最后，让我们测试一下映射器：

```java
@Test
public void givenCarEntitytoCar_whenMaps_thenCorrect() {
    Car entity = new Car();
    entity.setId(1);
    entity.setName("Toyota");

    CarDTO carDto = CarMapper.INSTANCE.carToCarDTO(entity);

    assertThat(carDto.getId()).isEqualTo(entity.getId());
    assertThat(carDto.getName()).isEqualTo(entity.getName());
}
```

## 3. 未映射的属性

由于MapStruct在编译时运行，因此它比动态映射框架更快。**如果映射不完整(即并非所有目标属性都映射)，它还可以生成错误报告**：

```text
Warning:(X,X) java: Unmapped target property: "propertyName".
```

虽然这在发生事故时是一个有用的警告，但如果故意缺少这些字段，我们可能更愿意采取不同的处理方式。

让我们通过映射两个简单对象的示例来探索这一点：

```java
public class DocumentDTO {
    private int id;
    private String title;
    private String text;
    private List<String> comments;
    private String author;
}
```

```java
public class Document {
    private int id;
    private String title;
    private String text;
    private Date modificationTime;
}
```

这两个类中都有一些唯一字段，这些字段在映射期间不应被填充，它们是：

- DocumentDTO中的comment
- DocumentDTO中的author
- Document中的modificationTime

如果我们定义一个映射器接口，它将导致构建期间出现警告消息：

```java
@Mapper
public interface DocumentMapper {
    DocumentMapper INSTANCE = Mappers.getMapper(DocumentMapper.class);

    DocumentDTO documentToDocumentDTO(Document entity);
    Document documentDTOToDocument(DocumentDTO dto);
}
```

由于我们不想映射这些字段，我们可以通过几种方式将它们排除在映射之外。

## 4. 忽略特定字段

要跳过特定映射方法中的几个属性，我们可以**使用@Mapping注解中的ignore属性**：

```java
@Mapper
public interface DocumentMapperMappingIgnore {

    DocumentMapperMappingIgnore INSTANCE = Mappers.getMapper(DocumentMapperMappingIgnore.class);

    @Mapping(target = "comments", ignore = true)
    @Mapping(target = "author", ignore = true)
    DocumentDTO documentToDocumentDTO(Document entity);

    @Mapping(target = "modificationTime", ignore = true)
    Document documentDTOToDocument(DocumentDTO dto);
}
```

在这里，我们提供字段名称作为目标，并将ignore设置为true以表明它不是映射所必需的。

但是这种技术在某些情况下并不方便，例如，当使用包含大量字段的大型模型时，我们可能会发现它很难使用。

## 5. 未映射目标策略

为了使代码更清晰、更具可读性，我们可以**指定未映射的目标策略**。

为此，当映射没有源字段时，我们使用MapStruct unmappedTargetPolicy来提供我们所需的行为：

- ERROR：任何未映射的目标属性都将导致构建失败-这可以帮助我们避免意外取消映射的字段
- WARN：(默认)构建期间的警告消息
- IGNORE：没有输出或错误

**为了忽略未映射的属性并且不输出警告，我们应该将IGNORE值分配给unmappedTargetPolicy**。根据目的，有几种方法可以做到这一点。

### 5.1 为每个Mapper设置策略

我们可以**将unmappedTargetPolicy设置给@Mapper注解**，这样，它的所有方法都将忽略未映射的属性：

```java
@Mapper(unmappedTargetPolicy = ReportingPolicy.IGNORE)
public interface DocumentMapperUnmappedPolicy {
    // mapper methods
}
```

### 5.2 使用共享MapperConfig

我们可以**通过@MapperConfig设置unmappedTargetPolicy**来忽略多个映射器中未映射的属性，从而在多个映射器之间共享设置。

首先我们创建一个带注解的接口：

```java
@MapperConfig(unmappedTargetPolicy = ReportingPolicy.IGNORE)
public interface IgnoreUnmappedMapperConfig {
}
```

然后我们将该共享配置应用到映射器：

```java
@Mapper(config = IgnoreUnmappedMapperConfig.class)
public interface DocumentMapperWithConfig { 
    // mapper methods 
}
```

我们应该注意，这是一个简单的例子，展示了@MapperConfig的最低限度的使用，这似乎并不比在每个映射器上设置策略好多少。当有多个设置需要在多个映射器之间标准化时，共享配置非常有用。

### 5.3 配置选项

最后，我们可以配置MapStruct代码生成器的注解处理器选项，当使用[Maven](https://www.baeldung.com/maven-compiler-plugin)时，我们可以使用处理器插件的compilerArgs参数传递处理器选项：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>${maven-compiler-plugin.version}</version>
            <configuration>
                <source>${maven.compiler.source}</source>
                <target>${maven.compiler.target}</target>
                <annotationProcessorPaths>
                    <path>
                        <groupId>org.mapstruct</groupId>
                        <artifactId>mapstruct-processor</artifactId>
                        <version>${org.mapstruct.version}</version>
                    </path>
                </annotationProcessorPaths>
                <compilerArgs>
                    <compilerArg>
                        -Amapstruct.unmappedTargetPolicy=IGNORE
                    </compilerArg>
                </compilerArgs>
            </configuration>
        </plugin>
    </plugins>
</build>
```

在这个例子中，我们忽略了整个项目中未映射的属性。

## 6. 优先顺序

我们研究了几种可以帮助我们处理部分映射并完全忽略未映射属性的方法，我们还了解了如何在映射器上独立应用它们，但我们也可以将它们组合起来。

假设我们有一个包含大量Bean和映射器的代码库，这些代码库都使用默认的MapStruct配置。除了少数情况外，我们不希望允许部分映射。我们可能会轻松地向Bean或其映射对应项添加更多字段，并在没有注意到的情况下获得部分映射。

因此，通过Maven配置添加全局设置以使在部分映射的情况下构建失败可能是一个好主意。

现在，为了允许某些映射器中未映射的属性并覆盖全局行为，我们可以结合这些技术，同时牢记优先顺序(从高到低)：

- 在映射器方法级别忽略特定字段
- 映射器上的策略
- 共享的MapperConfig
- 全局配置

## 7. 总结

在本教程中，我们研究了如何配置MapStruct以忽略未映射的属性。