---
layout: post
title: 使用Hamcrest检查集合是否包含元素
category: assertion
copyright: assertion
excerpt: Hamcrest
---

## 1. 概述

当我们用Java编写单元测试时，尤其是使用[JUnit](https://www.baeldung.com/junit-5)框架时，通常会验证[Collection](https://www.baeldung.com/java-collections)是否包含特定元素。作为一个功能强大的库，[Hamcrest](https://www.baeldung.com/hamcrest-text-matchers)提供了一种使用Matcher执行这些检查的简单而富有表现力的方法。

在本快速教程中，我们将探索如何使用Hamcrest的Matcher检查Collection是否包含特定元素。此外，由于数组是常用的数据容器，我们还将讨论如何对数组执行相同的检查。

## 2. 设置Hamcrest

在深入研究示例之前，我们需要确保Hamcrest包含在我们的项目中。如果我们使用[Maven](https://www.baeldung.com/maven)，可以将以下依赖项添加到我们的pom.xml中：

```xml
<dependency>
    <groupId>org.hamcrest</groupId>
    <artifactId>hamcrest</artifactId>
    <version>2.2</version>
    <scope>test</scope>
</dependency>
```

因此，让我们准备一个列表和一个数组作为输入：

```java
static final List<String> LIST = List.of("a", "b", "c", "d", "e", "f");
static final String[] ARRAY = { "a", "b", "c", "d", "e", "f" };
```

接下来，让我们检查它们是否包含特定元素。

## 3. 使用HamcrestMatcher和assertThat()

Hamcrest提供了一组丰富的Matcher来处理Collection，要检查Collection是否包含特定元素，我们可以使用[org.hamcrest.Matchers](https://hamcrest.org/JavaHamcrest/javadoc/2.2/org/hamcrest/Matchers.html)类中的[hasItem()](https://hamcrest.org/JavaHamcrest/javadoc/2.2/org/hamcrest/Matchers.html#hasItem-T-) Matcher。让我们通过一些示例来了解它在实践中是如何工作的。

**首先，让我们导入静态hasItem()方法以使代码易于阅读**：

```java
import static org.hamcrest.Matchers.hasItem;
```

然后，我们可以用它来验证Collection是否包含某个元素：

```java
assertThat(LIST, hasItem("a"));
assertThat(LIST, not(hasItem("x")));
```

在上面的代码中，我们**使用[not()](https://hamcrest.org/JavaHamcrest/javadoc/2.2/org/hamcrest/Matchers.html#not-org.hamcrest.Matcher-)方法来否定匹配参数的逻辑**。

hasItem()方法可以很直接地验证Collection是否包含某个元素。但是，我们不能用它来检查数组。

要检查数组是否包含特定元素，我们可以使用[hasItemInArray()](https://hamcrest.org/JavaHamcrest/javadoc/2.2/org/hamcrest/Matchers.html#hasItemInArray-T-)方法，该方法也来自org.hamcrest.Matchers类：

```java
assertThat(ARRAY, hasItemInArray("a"));
assertThat(ARRAY, not(hasItemInArray("x")));
```

正如示例所示，我们可以通过使用Hamcrest方便的hasItem()和hasItemInArray()轻松解决我们的问题。

## 4. 使用JUnit的assertTrue()和assertFalse()

我们已经看到了Hamcrest的Matcher，它们很容易使用。或者，**我们也可以使用JUnit的assertTrue()和assertFalse()方法来实现目标**：

```java
assertTrue(LIST.contains("a"));
assertFalse(LIST.contains("x"));
```

这次，**我们使用Collection的contains()方法来检查目标元素是否存在于Collection中**。

但是，如果输入是数组，则与Collection.contains()不同，没有简单的一次性方法可以检查。幸运的是，我们有多种方法可以[在Java中检查数组是否包含值](https://www.baeldung.com/java-array-contains-value)。

接下来，让我们看看如何在JUnit assertTrue()和assertFalse()中使用这些方法：

```java
assertTrue(Arrays.stream(ARRAY).anyMatch("a"::equals));
assertFalse(Arrays.asList(ARRAY).contains("z"));
```

如代码所示，我们可以将数组转换为List或使用Stream API来检查数组中是否存在特定值。

从上面的示例中，我们还可以看到，如果我们要检查Collection是否包含某个元素，Hamcrest Matcher和使用Collection.contains()方法的JUnit断言都很简单。但是，**当我们需要对数组执行相同的检查时，Hamcrest Matcher可能是更好的选择，因为它们更紧凑且更易于理解**。

## 5. 总结

在这篇简短的文章中，我们探讨了使用JUnit和Hamcrest断言集合或数组是否包含特定元素的各种方法。

使用Hamcrest的hasItem()或hasItemInArray()，我们可以轻松地在单元测试中验证Collection或数组中特定元素的存在。此外，Matcher使我们的测试更具可读性和表现力，从而增强了测试代码的清晰度和可维护性。

或者，我们可以使用Java标准API提供的方法来检查Collection和数组是否包含目标元素，然后使用JUnit的标准assertTrue()和assertFalse()断言来完成这项工作。