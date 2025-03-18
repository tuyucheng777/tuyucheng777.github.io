---
layout: post
title:  断言JSON对象集合时忽略顺序
category: assertion
copyright: assertion
excerpt: JSONassert
---

## 1. 简介

断言JSON对象集合相等可能具有挑战性，尤其是在无法保证集合内元素顺序的情况下。虽然可以使用[Jackson](https://www.baeldung.com/jackson-object-mapper-tutorial)和[AssertJ](https://www.baeldung.com/introduction-to-assertj)等库，但JSONassert和hamcrest-json等更专业的工具旨在更可靠地处理此用例。

在本教程中，我们将探讨如何比较JSON对象的集合，重点是使用JSONassert和hamcrest-json忽略元素的顺序。

## 2. 问题陈述

当我们处理JSON对象集合时，列表中元素的顺序会根据数据源而变化。

考虑以下JSON数组：

```json
[
    {"id": 1, "name": "Alice", "address": {"city": "NY", "street": "5th Ave"}},
    {"id": 2, "name": "Bob", "address": {"city": "LA", "street": "Sunset Blvd"}}
]
```

```json
[
  {"id": 2, "name": "Bob", "address": {"city": "LA", "street": "Sunset Blvd"}},
  {"id": 1, "name": "Alice", "address": {"city": "NY", "street": "5th Ave"}}
]
```

虽然这些数组具有相同的元素，但顺序不同。**尽管它们的数据相同，但直接对这些数组进行字符串比较将由于顺序差异而失败**。

让我们将这些JSON数组定义为Java变量，然后探索如何在忽略顺序的情况下比较它们是否相等：

```java
String jsonArray1 = "["
        + "{\"id\": 1, \"name\": \"Alice\", \"address\": {\"city\": \"NY\", \"street\": \"5th Ave\"}}, "
        + "{\"id\": 2, \"name\": \"Bob\", \"address\": {\"city\": \"LA\", \"street\": \"Sunset Blvd\"}}"
        + "]";

String jsonArray2 = "["
        + "{\"id\": 2, \"name\": \"Bob\", \"address\": {\"city\": \"LA\", \"street\": \"Sunset Blvd\"}}, "
        + "{\"id\": 1, \"name\": \"Alice\", \"address\": {\"city\": \"NY\", \"street\": \"5th Ave\"}}"
        + "]";
```

## 3. 使用JSONassert进行JSON比较

[JSONassert](https://www.baeldung.com/jsonassert)提供了一种灵活的JSON数据比较方法，允许我们将JSON对象或数组作为JSON进行比较，而不是直接比较字符串。**具体来说，它可以比较数组而忽略元素的顺序**。

在LENIENT模式下，JSONAssert仅关注内容，而忽略顺序：

```java
@Test
public void givenJsonArrays_whenUsingJSONAssertIgnoringOrder_thenEqual() throws JSONException {
    JSONAssert.assertEquals(jsonArray1, jsonArray2, JSONCompareMode.LENIENT);
}
```

在此测试中，JSONCompareMode.LENIENT模式允许我们在忽略元素顺序的情况下断言相等，这使得JSONassert非常适合我们期望数据相同但元素顺序可能不同的情况。

### 3.1 使用JSONassert忽略额外字段

JSONassert还允许使用相同的LENIENT模式忽略JSON对象中的额外字段，**这在比较JSON数据时非常有用，因为某些字段(如元数据或时间戳)与测试无关**：

```java
@Test
public void givenJsonWithExtraFields_whenIgnoringExtraFields_thenEqual() throws JSONException {
    String jsonWithExtraFields = "["
            + "{\"id\": 1, \"name\": \"Alice\", \"address\": {\"city\": \"NY\", \"street\": \"5th Ave\"}, \"age\": 30}, "
            + "{\"id\": 2, \"name\": \"Bob\", \"address\": {\"city\": \"LA\", \"street\": \"Sunset Blvd\"}, \"age\": 25}"
            + "]";

    JSONAssert.assertEquals(jsonArray1, jsonWithExtraFields, JSONCompareMode.LENIENT);
}
```

在此示例中，测试验证jsonArray1等同于jsonWithExtraFields，允许在比较中使用age等额外字段。

## 4. 使用hamcrest-json进行JSON匹配

除了JSONassert之外，我们还可以利用[hamcrest-json](https://mvnrepository.com/artifact/uk.co.datumedge/hamcrest-json)[Hamcrest](https://www.baeldung.com/java-junit-hamcrest-guide)，这是专为JSON断言设计的插件。此插件基于Hamcrest的匹配器功能构建，允许我们在JUnit中使用JSON时编写富有表现力且可读的断言。

**hamcrest-json中最有用的功能之一是allowingAnyArrayOrdering()方法**，这使我们能够比较JSON数组，并忽略它们的顺序：

```java
@Test
public void givenJsonCollection_whenIgnoringOrder_thenEqual() {
    assertThat(jsonArray1, sameJSONAs(jsonArray2).allowingAnyArrayOrdering());
}
```

这种方法可确保JSON相等性检查使用sameJSONAs()匹配器忽略数组中元素的顺序。

### 4.1 使用hamcrest-json忽略额外字段

除了忽略数组排序之外，hamcrest-json还提供了allowingExtraUnexpectedFields()方法来处理额外字段。此方法使我们能够忽略一个JSON对象中存在的字段，但不忽略另一个JSON对象中存在的字段：

```java
@Test
public void givenJsonWithUnexpectedFields_whenIgnoringUnexpectedFields_thenEqual() {
    String jsonWithUnexpectedFields = "["
            + "{\"id\": 1, \"name\": \"Alice\", \"address\": {\"city\": \"NY\", \"street\": \"5th Ave\"}, \"extraField\": \"ignoreMe\"}, "
            + "{\"id\": 2, \"name\": \"Bob\", \"address\": {\"city\": \"LA\", \"street\": \"Sunset Blvd\"}}"
            + "]";

    assertThat(jsonWithUnexpectedFields, sameJSONAs(jsonArray1).allowingExtraUnexpectedFields());
}
```

在此示例中，我们验证jsonWithUnexpectedFields是否等于jsonArray1，即使它包含一个额外字段。通过将allowingExtraUnexpectedFields()与allowingAnyArrayOrdering()结合使用，我们确保进行稳健的比较，重点是匹配JSON数组之间的数据。

## 5. 总结

在本文中，我们演示了如何使用JSONassert和hamcrest-json等专用库来比较JSON对象集合，同时忽略元素的顺序。这些库提供了一种比手动将JSON解析为Java对象更直接的方法，从而提供更高的可靠性和易用性。