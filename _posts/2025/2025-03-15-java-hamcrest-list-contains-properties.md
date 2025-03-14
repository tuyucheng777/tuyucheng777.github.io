---
layout: post
title: 在Hamcrest中检查List是否包含具有特定属性的元素
category: assertion
copyright: assertion
excerpt: Hamcrest
---

## 1. 概述

在用Java编写单元测试时，尤其是使用[JUnit](https://www.baeldung.com/junit-5)框架时，我们经常需要验证列表中的元素是否具有特定属性。

[Hamcrest](https://www.baeldung.com/hamcrest-text-matchers)是一个广泛使用的Matcher库，它提供了直接且富有表现力的方法来执行这些检查。

在本教程中，我们将探讨如何使用JUnit和Hamcrest的Matcher检查列表是否包含具有特定属性的元素。

## 2. 设置Hamcrest和示例

在设置示例之前，让我们快速将Hamcrest依赖项添加到我们的pom.xml中：

```xml
<dependency>
    <groupId>org.hamcrest</groupId>
    <artifactId>hamcrest</artifactId>
    <version>2.2</version>
    <scope>test</scope>
</dependency>
```

可以在[Maven Central](https://mvnrepository.com/artifact/org.hamcrest/hamcrest)中检查该依赖的最新版本。

现在，让我们创建一个简单的[POJO类](https://www.baeldung.com/java-pojo-class)：

```java
public class Developer {
    private String name;
    private int age;
    private String os;
    private List<String> languages;
 
    public Developer(String name, int age, String os, List<String> languages) {
        this.name = name;
        this.age = age;
        this.os = os;
        this.languages = languages;
    }
    // ... getters are omitted
}
```

如代码所示，Developer类拥有一些描述开发人员的属性，比如姓名、年龄、操作系统以及开发人员主要使用的编程语言。

接下来，让我们创建一个Developer实例列表：

```java
private static final List<Developer> DEVELOPERS = List.of(
    new Developer("Kai", 28, "Linux", List.of("Kotlin", "Python")),
    new Developer("Liam", 26, "MacOS", List.of("Java", "C#")),
    new Developer("Kevin", 24, "MacOS", List.of("Python", "Go")),
    new Developer("Saajan", 22, "MacOS", List.of("Ruby", "Php", "Typescript")),
    new Developer("Eric", 27, "Linux", List.of("Java", "C"))
);
```

我们将以DEVELOPERS列表为例，介绍如何使用JUnit和Hamcrest检查元素是否包含特定属性。

## 3. 使用hasItem()和hasProperty()

Hamcrest提供了一组丰富便捷的[Matcher](https://hamcrest.org/JavaHamcrest/javadoc/2.2/org/hamcrest/Matchers.html)，我们可以将[hasProperty()](https://hamcrest.org/JavaHamcrest/javadoc/2.2/org/hamcrest/Matchers.html#hasProperty-java.lang.String-org.hamcrest.Matcher-) Matcher与[hasItem()](https://hamcrest.org/JavaHamcrest/javadoc/2.2/org/hamcrest/Matchers.html#hasItem-org.hamcrest.Matcher-) Matcher结合使用：

```java
assertThat(DEVELOPERS, hasItem(hasProperty("os", equalTo("Linux"))));
```

此示例显示如何检查**至少一个元素的os是否为“Linux”**。 

我们可以将不同的属性名称传递给hasProperty()来验证另一个属性，例如：

```java
assertThat(DEVELOPERS, hasItem(hasProperty("name", is("Kai"))));
```

在上面的例子中，我们使用**is()(equalTo()的别名)**来检查列表是否有name等于“Kai”的元素。

当然，除了equalTo()和is()之外，**我们还可以在hasProperty()中使用其他Matcher以不同的方式验证元素的属性**：

```java
assertThat(DEVELOPERS, hasItem(hasProperty("age", lessThan(28))));
assertThat(DEVELOPERS, hasItem(hasProperty("languages", hasItem("Go"))));
```

断言语句读起来就像自然语言一样，例如：“断言DEVELOPERS列表有一个元素，该元素的属性名为age，并且值小于28”。

## 4. anyOf()和allOf()匹配器

hasProperty()可以方便地检查单个属性，**我们还可以通过使用[anyOf()](https://hamcrest.org/JavaHamcrest/javadoc/2.2/org/hamcrest/Matchers.html#anyOf-org.hamcrest.Matcher...-)和[allOf()](https://hamcrest.org/JavaHamcrest/javadoc/2.2/org/hamcrest/Matchers.html#allOf-org.hamcrest.Matcher...-)组合多个hasProperty()调用来检查列表中的元素是否满足多个属性**。 

anyOf()内部的任何一个Matcher匹配成功，则整个anyOf()都满足。接下来我们通过一个例子来理解：

```java
assertThat(DEVELOPERS, hasItem(
    anyOf(
        hasProperty("languages", hasItem("C")),
        hasProperty("os", is("Windows"))) // <-- No dev has the OS "Windows"
));
```

如示例所示，尽管DEVELOPERS列表中没有元素的os等于“Windows”，但断言通过，因为我们有一个元素(“Eric”)的language包含“C”。

因此，**anyOf()对应于“OR”逻辑**：如果有任何元素的language包含“C”或os为“Windows”。

相反，**allOf()执行“AND”逻辑**，接下来我们看一个例子：

```java
assertThat(DEVELOPERS, hasItem(
    allOf(
        hasProperty("languages", hasItem("C")),
        hasProperty("os", is("Linux")),
        hasProperty("age", greaterThan(25)))
));
```

在上面的测试中，我们**检查DEVELOPERS中是否至少有一个元素同时满足allOf()中的三个hasProperty() Matcher**。

由于“Eric”的属性通过了三个hasProperty() Matcher检查，因此测试通过。

接下来我们对测试做一些修改：

```java
assertThat(DEVELOPERS, not(hasItem( // <-- not() matcher
    allOf(
        hasProperty("languages", hasItem("C#")),
        hasProperty("os", is("Linux")),
        hasProperty("age", greaterThan(25)))
)));
```

这次，我们没有匹配，因为没有元素可以通过所有三个hasProperty() Matcher。

## 5. 使用JUnit的assertTrue()和Stream.anyMatch()

Hamcrest提供了方便的方法来验证列表中的元素是否具有特定属性。或者，**我们可以使用标准JUnit assertTrue()断言和[Java Stream API](https://www.baeldung.com/java-8-streams-introduction)中的anyMatch()来执行相同的检查**。

如果流中的任何元素通过了检查函数，则anyMatch()方法返回true。接下来我们看一些例子：

```java
assertTrue(DEVELOPERS.stream().anyMatch(dev -> dev.getOs().equals("Linux")));
assertTrue(DEVELOPERS.stream().anyMatch(dev -> dev.getAge() < 28));
assertTrue(DEVELOPERS.stream().anyMatch(dev -> dev.getLanguages().contains("Go")));
```

值得注意的是，当我们使用[Lambda](https://www.baeldung.com/java-8-lambda-expressions-tips)检查元素的属性时，我们可以直接调用getter方法来获取它们的值，这比Hamcrest的hasProperty() Matcher更简单，后者要求属性名称为文字字符串。

当然，如果需要的话，我们可以轻松扩展Lambda表达式来执行复杂的检查：

```java
assertTrue(DEVELOPERS.stream().anyMatch(dev -> dev.getLanguages().contains("C") && dev.getOs().equals("Linux")));
```

上面的测试展示了一个Lambda表达式来检查流中元素的多个属性。

## 6. 总结

在本文中，我们探讨了使用JUnit和Hamcrest断言列表是否包含具有某些属性的元素的各种方法。

无论是使用简单属性还是组合多个条件，Hamcrest都提供了强大的工具包来验证集合中元素的属性。此外，Hamcrest Matcher使我们的测试更具可读性和表现力，从而提高了测试代码的清晰度和可维护性。

或者，我们可以使用JUnit assertTrue()和Stream API中的anyMatch()执行这种检查。