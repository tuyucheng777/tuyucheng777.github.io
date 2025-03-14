---
layout: post
title: 使用Hamcrest检查变量是否为空
category: assertion
copyright: assertion
excerpt: Hamcrest
---

## 1. 概述

当我们用Java编写单元测试时，特别是使用[JUnit](https://www.baeldung.com/junit-5)框架时，我们经常需要验证某些变量是否为null。[Hamcrest](https://www.baeldung.com/hamcrest-text-matchers)是一个用于创建灵活测试的流行Matcher库，它提供了一种方便的方法来实现这一点。

在此快速教程中，我们将探讨如何使用JUnit和Hamcrest的Matcher检查变量是否为空或非空。

## 2. 使用Hamcrest的assertThat()

要使用Hamcrest，我们需要将依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>org.hamcrest</groupId>
    <artifactId>hamcrest</artifactId>
    <version>2.2</version>
    <scope>test</scope>
</dependency>
```

最新版本可以在[Maven Central](https://mvnrepository.com/artifact/org.hamcrest/hamcrest)上找到。

Hamcrest的assertThat()方法及其Matcher允许我们编写灵活的断言，接下来，让我们看看如何使用这种方法断言变量是否为空。

Hamcrest在[org.hamcrest.core.IsNull](https://hamcrest.org/JavaHamcrest/javadoc/2.2/org/hamcrest/core/IsNull.html)类中提供了与null相关的匹配器方法。**例如，我们可以使用静态方法nullValue()和notNullValue()来获取用于检查null和非null的Matcher**。

此外，Hamcrest将常用的Matcher归类到[org.hamcrest.Matchers](https://hamcrest.org/JavaHamcrest/javadoc/2.2/org/hamcrest/Matchers.html)类中。**在Matchers类中，也提供了nullValue()和notNullValue()方法，它们只是调用IsNull类中的相应方法**。因此，我们可以从任一类中静态导入这些方法来使用这些方法，并使代码易于阅读：

```java
import static org.hamcrest.Matchers.notNullValue;
import static org.hamcrest.Matchers.nullValue;
```

我们可以像这样使用它们：

```java
String theNull = null;
assertThat(theNull, nullValue());
 
String theNotNull = "I am a good String";
assertThat(theNotNull, notNullValue());
```

对于非空检查，我们还可以组合Matcher并使用not(nullValue())作为替代：

```java
assertThat(theNotNull, not(nullValue()));
```

值得注意的是，**not()方法来自[org.hamcrest.core.IsNot](https://hamcrest.org/JavaHamcrest/javadoc/2.2/org/hamcrest/core/IsNot.html)类，它否定了匹配参数的逻辑**。

## 3. 使用JUnit的Null和非Null断言

正如我们所见，Hamcrest的匹配器可以方便地执行null或非null断言。或者，**JUnit附带的assertNull()和assertNotNull()方法可以直接执行以下检查**：

```java
String theNull = null;
assertNull(theNull); 
 
String theNotNull = "I am a good String";
assertNotNull(theNotNull);
```

如代码所示，即使没有Matcher，JUnit断言方法在检查null时仍然易于使用和阅读。

## 4. 总结

在这篇简短的文章中，我们探讨了使用JUnit和Hamcrest有效断言空变量和非空变量的不同方法。

通过使用Hamcrest的nullValue()和notNullValue()Matcher，我们可以轻松检查单元测试中的变量是否为空或非空，该库的表达性语法使我们的测试更具可读性和可维护性。

另外，JUnit的标准assertNull()和assertNotNull()断言可以直接完成这项工作。