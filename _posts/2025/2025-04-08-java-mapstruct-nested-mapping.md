---
layout: post
title:  使用Mapstruct和Java在另一个Mapper中使用Mapper
category: libraries
copyright: libraries
excerpt: Mapstruct
---

## 1. 概述

MapStruct是一个库，它帮助我们在处理Java Bean映射时尽量减少样板代码，它仅使用提供的接口来生成映射器。

在本教程中，我们将学习如何使用简单映射器构建复杂映射器并映射嵌套结构。

## 2. 数据

我们将Article类映射到DTO实例；Article包含一些简单字段，但也包含Person类型的author字段，我们也将把该字段映射到相应的DTO。以下是源类：

```java
@Getter
@Setter
public class Article {
    private int id;
    private String name;
    private Person author;
}
```

```java
@Getter
@Setter
public class Person {  
    private String id;
    private String name;
}
```

目标类别如下：

```java
@Getter
@Setter
public class ArticleDTO {
    private int id;
    private String name;
    private PersonDTO author;
}
```

```java
@Getter
@Setter
public class PersonDTO {
    private String id;
    private String name;
}
```

## 3. 将嵌套映射器定义为方法

让我们首先定义一个映射Article类的简单映射器：

```java
@Mapper
public interface ArticleMapper {
   
    ArticleMapper INSTANCE = Mappers.getMapper(ArticleMapper.class);
    
    ArticleDTO articleToArticleDto(Article article);
}
```

此映射器将正确映射源类中的所有字面量字段，但不会映射author字段，因为它不知道如何映射。让我们定义PersonMapper接口：

```java
@Mapper
public interface PersonMapper {
    
    PersonMapper INSTANCE = Mappers.getMapper(PersonMapper.class);
    
    PersonDTO personToPersonDTO(Person person);
}
```

现在我们可以做的最简单的事情是在ArticleMapper中创建一个方法，定义从Person到PersonDTO的映射：

```java
default PersonDTO personToPersonDto(Person person) {
    return Mappers.getMapper(PersonMapper.class).personToPersonDTO(person);
}
```

MapStruct将自动选择此方法并使用它来映射author字段。

## 4. 使用现有的映射器

虽然上述解决方案有效，但它有点麻烦。我们不需要定义新方法，而是可以使用“uses”参数在@Mapper注解中直接指向我们想要使用的映射器：

```java
@Mapper(uses = PersonMapper.class)
public interface ArticleUsingPersonMapper {
    
    ArticleUsingPersonMapper INSTANCE = Mappers.getMapper(ArticleUsingPersonMapper.class);
    
    ArticleDTO articleToArticleDto(Article article);
}
```

## 5. 总结

在本文中，我们学习了如何在另一个映射器中使用MapStruct映射器。