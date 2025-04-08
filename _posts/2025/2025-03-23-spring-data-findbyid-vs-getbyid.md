---
layout: post
title:  Spring Data中findById和getById的区别
category: springdata
copyright: springdata
excerpt: Spring Data JPA
---

## 1. 概述

[Spring Data](https://www.baeldung.com/spring-data)提供了从数据存储中检索实体的便捷方法，包括findById和getById。虽然乍一看它们似乎很相似，但存在一些细微的差别，可能会影响我们代码的功能。

本教程将探讨这些差异并帮助我们确定何时有效地使用每种方法。

## 2. 理解findById

首先我们来看一下findById方法。

### 2.1 方法签名

findById方法在CrudRepository接口中定义：

```java
Optional<T> findById(ID id);
```

### 2.2 行为和返回类型

findById方法通过其唯一标识符(id)检索实体，它返回一个[Optional](https://www.baeldung.com/java-optional)包装器，表示该实体可能存在于数据存储中，也可能不存在。**如果找到该实体，它将被包装在Optional中。否则，Optional将为空**。

### 2.3 使用场景

在一些用例中，findById是更好的选择。

第一种情况是，**我们预计实体可能在数据存储中不存在**，并且想要优雅地处理两种情况(找到和未找到)。

此外，当我们想将它与Java 8 Optional API结合使用来执行条件操作或在未找到实体时执行回退逻辑时，它也很方便。

## 3. 理解getById

接下来我们来探索一下getById方法。

### 3.1 方法签名

JpaRepository接口定义了getById方法：

```java
T getById(ID id);
```

### 3.2 行为和返回类型

与findById不同，getById方法直接返回实体，而不是将其包装在Optional中。**如果数据存储中不存在该实体，则会抛出EntityNotFoundException**。

### 3.3 使用场景

当我们Spring Data 中 findById 和 getById 的区别，我们可以使用getById。此方法还提供了更简洁的语法，并避免了额外的Optional处理。

## 4. 选择正确的方法

最后，让我们考虑一些可以帮助我们确定是否使用findById或getById的因素：

- 存在性保证：如果保证实体的存在，并且如果缺失则会出现异常情况，则首选getById，这种方法避免了不必要的可选处理并简化了代码。
- 可能缺失：如果不确定实体是否存在，或者我们需要妥善处理找到和未找到的情况，请结合使用findById和Optional API，这种方法允许我们执行条件操作而不会引发异常。
- 从Spring Boot 2.7版本开始，**getById方法被标记为已弃用**，文档建议改用getReferenceById方法。

## 5. 总结

在本教程中，我们了解了Spring Data 中findById和getById方法之间的区别。findById返回Optional并妥善处理实体缺失的情况，而getById则直接返回实体，如果实体不存在则抛出异常。这两种方法之间的选择取决于实体的存在是有保证的还是不确定的，以及是否需要异常处理或条件操作。

因此我们可以选择最符合我们应用程序的逻辑和要求的那个。