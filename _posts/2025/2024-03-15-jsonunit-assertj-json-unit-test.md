---
layout: post
title: 使用JsonUnit进行JSON单元测试断言
category: assertion
copyright: assertion
excerpt: JsonUnit
---

## 1. 概述

在这篇简短的文章中，我们将探索JsonUnit库并使用它为JSON对象创建富有表现力的断言。我们将从一个简单的示例开始，展示JsonUnit和[AssertJ](https://www.baeldung.com/introduction-to-assertj)之间的顺畅集成。

之后，我们将学习如何根据模板字符串验证JSON。这种方法提供了灵活性，并允许我们使用自定义占位符将元素值与各种条件进行匹配，或者完全忽略特定的JSON路径。

## 2. 入门

JsonUnit项目包含几个可以独立导入的不同模块，[json-unit-assertj](https://mvnrepository.com/artifact/net.javacrumbs.json-unit/json-unit-assertj)模块是使用该库的推荐方式，它提供了流式的API以及与AssertJ库的顺畅集成。

将这个依赖项添加到我们的pom.xml中：

```xml
<dependency>
    <groupId>net.javacrumbs.json-unit</groupId>
    <artifactId>json-unit-assertj</artifactId>
    <version>3.5.0</version>
    <scope>test</scope>
</dependency>
```

该库假定JSON反序列化器已位于我们的类路径中，并自动尝试使用它。目前，支持的集成包括[Gson](https://www.baeldung.com/java-json#gson)、[org.json](https://www.baeldung.com/java-org-json)、[Moshi](https://www.baeldung.com/java-json-moshi)和[Jackson2](https://www.baeldung.com/java-json#jackson)。

我们现在可以使用静态工厂方法assertThatJson()创建一个断言对象，因此，**我们将能够使用JsonUnit的流式API来验证被测试的JSON是否是有效的JSON对象并验证其键值对**：

```java
@Test
void whenWeVerifyAJsonObject_thenItContainsKeyValueEntries() {
    String articleJson = """ 
            {
               "name": "A Guide to Spring Boot",
               "tags": ["java", "spring boot", "backend"]
            }
        """;

    assertThatJson(articleJson)
        .isObject()
        .containsEntry("name", "A Guide to Spring Boot")
        .containsEntry("tags", List.of("java", "spring boot", "backend"));
}
```

JsonUnit允许我们轻松访问JSON对象，同时利用AssertJ强大的断言进行验证。**例如，我们可以使用node()等方法来访问JSON，使用isArray()等方法来访问专为断言集合而定制的API**：

```java
assertThatJson(articleJson)
    .isObject()
    .containsEntry("name", "A Guide to Spring Boot")
    .node("tags")
    .isArray()
    .containsExactlyInAnyOrder("java", "spring boot", "backend");
```

**另一方面，我们可以用声明的方式将测试对象与普通的JSON字符串进行比较**。为此，我们只需要将预期内容包装在ExpectedNode实例中，这可以使用函数JsonAssertion.json()来完成：

```java
assertThatJson(articleJson)
  .isObject()
  .isEqualTo(json("""
      {
          "name": "A Guide to Spring Boot",
          "tags": ["java", "spring boot", "backend"]
      }
    """));
```

## 3. 支持的功能

**该库提供[许多功能](https://github.com/lukas-krecan/JsonUnit?tab=readme-ov-file#features)，例如导航复杂的JSON路径、忽略字段或值以及针对自定义匹配器或正则表达式模式进行断言**。虽然我们不会讨论每个功能，但我们将探讨一些关键功能。

### 3.1 Option

JsonUnit使我们能够为每个断言定义各种配置，**我们可以在Option枚举中找到支持的功能，并通过when()方法启用它们**。

让我们启用IGNORE_ARRAY_ORDER和IGNORE_EXTRA_ARRAY_ITEMS选项，以另一种方式验证JSON是否以任意顺序包含标签“java”和“backend”：

```java
assertThatJson(articleJson)
    .when(Option.IGNORING_ARRAY_ORDER)
    .when(Option.IGNORING_EXTRA_ARRAY_ITEMS)
    .node("tags")
    .isEqualTo(json("""
          ["backend", "java"]
        """));
```

### 3.2 占位符

**在根据预期内容验证JSON输出时，该库允许将预期结果模板化**。例如，\${json-unit.any-string}和\${json-unit.ignore-element}等占位符可以将文章name验证为字符串，同时忽略其tags列表：

```java
assertThatJson(articleJson)
  .isEqualTo(json(""" 
      {
         "name": "${json-unit.any-string}",
         "tags": "${json-unit.ignore-element}"
      }
    """));
```

类似地，我们可以使用诸如\${json-unit.any-number}、\${json-unit.any-boolean}、\${json-unit.regex}之类的占位符，甚至可以创建针对特定用例的自定义占位符。

### 3.3 忽略路径

到目前为止，我们已经学习了如何使用占位符来忽略JSON对象的元素。**但是，我们还可以利用whenIgnoringPaths()方法来定义执行断言时要忽略的JSON路径**：

```java
assertThatJson(articleJson)
    .whenIgnoringPaths("tags")
    .isEqualTo(json(""" 
          {
             "name": "A Guide to Spring Boot",
             "tags": [ "ignored", "tags" ]
          }
      """));
    }
```

可以看到，当我们使用whenIgnoringPaths(“tags”)时，即使两个JSON对象的tags键具有不同的值，断言也会通过。

## 4. 总结

在本实践教程中，我们讨论了JsonUnit的基础知识，并探索了用于创建流式和声明性断言的API，使JSON验证变得简单而富有表现力。

我们还学习了如何通过将JSON内容与模板字符串进行比较来验证JSON内容，当与其他库功能(例如配置选项、自定义占位符和类型匹配器)结合使用时，此技术尤其强大。