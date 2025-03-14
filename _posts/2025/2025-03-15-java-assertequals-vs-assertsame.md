---
layout: post
title: JUnit中的assertEquals()与assertSame()
category: unittest
copyright: unittest
excerpt: JUnit
---

## 1. 概述

[JUnit](https://www.baeldung.com/junit)是一个流行的测试框架，作为其API的一部分，它提供了一种检查和比较对象的便捷方法。**然而，assertEquals()和assertSame()这两个方法之间的区别并不总是很明显**。

在本教程中，我们将检查assertEquals()和assertSame()方法。它们存在于[JUnit 4和JUnit 5](https://www.baeldung.com/junit-assertions)中，并且行为相同。但是，在本文中，我们将在示例中使用[JUnit 5](https://www.baeldung.com/junit-5)。

## 2. 标识和相等性

**当我们比较两个对象时，我们使用两个概念：标识和相等性**。标识检查两个对象或元素是否相同，例如，两个人在物理意义上不能被视为相同。在计算机科学中，这更容易，因为我们使用引用的概念。

人们在世界任何地方看到的太阳都是相同的，电影或照片中的任何太阳图像都与我们透过窗户看到的太阳相同。**因此，我们对同一个底层物体有不同的表述**。

**同时，相等性不一定考虑对象标识，而是检查它们是否可以被视为相等**。它是一个更灵活的概念，可以根据上下文以不同的方式表示。例如，人们不能被视为相同(在物理意义上)，但可以在不同方面相等：身高、年龄、职业等。

两个相同的对象默认是相等的，但是两个相等的对象不一定相同。

## 3. equals()和==

在Java中，标识和相等性的概念可以用[==和equals()](https://www.baeldung.com/java-equals-method-operator-difference)方法表示。我们可以通过[Strings](https://www.baeldung.com/java-string)看到它们的行为：

```java
@ParameterizedTest
@ValueSource(strings = {"Hello", "World"})
void givenAString_WhenCompareInJava_ThenItEqualsAndSame(String string) {
    assertTrue(string.equals(string));
    assertTrue(string == string);
}
```

**这表明，如果我们有相同的对象，它将与自身相同且相等**。但是，如果我们有具有相同内容的不同实例，我们将得到不同的结果：

```java
@ParameterizedTest
@ValueSource(strings = {"Hello", "World"})
void givenAStrings_WhenCompareNewStringsInJava_ThenItEqualsButNotSame(String string) {
    assertTrue(new String(string).equals(new String(string)));
    assertFalse(new String(string) == new String(string));
}
```

在这里，对象相等但不完全相同。有时，当我们不提供[equals()和hashCode()](https://www.baeldung.com/java-equals-hashcode-contracts)方法的实现时，我们可能会遇到类问题：

```java
public class Person {
    private final String firstName;
    private final String lastName;

    // constructors, getters, and setters
}
```

即使值相同，两个实例也不会完全相同，根据上面的解释，这是合理的。**但是，它们也不会相等**：

```java
@Test
void givePeople_WhenCompareWithoutOverridingEquals_TheyNotEqual() {
    Person firstPerson = new Person("John", "Doe");
    Person secondPerson = new Person("John", "Doe");
    assertNotEquals(firstPerson, secondPerson);
}
```

这是因为我们将使用Object类中的equals()方法的实现：

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

我们可以看到，默认行为会回归到标识比较，并检查引用而不是对象的内容。这就是为什么提供[equals()和hashCode()](https://www.baeldung.com/java-equals-hashcode-contracts)方法的有效实现很重要。

## 4. JUnit

熟悉了标识和相等性的概念之后，理解assertEquals()和assertSame()方法的行为就容易多了。**第一个是使用equals()方法来比较元素**：

```java
@ParameterizedTest
@ValueSource(strings = {"Hello", "World"})
void givenAString_WhenCompare_ThenItEqualsAndSame(String string) {
    assertEquals(string, string);
    assertSame(string, string);
}
```

同时，第二个使用标识并检查两个对象是否指向堆中的同一位置：

```java
@ParameterizedTest
@ValueSource(strings = {"Hello", "World"})
void givenAStrings_WhenCompareNewStrings_ThenItEqualsButNotSame(String string) {
    assertEquals(new String(string), new String(string));
    assertNotSame(new String(string), new String(string));
}
```

## 5. 总结

标识和相等性是相关概念，但在比较过程中它们考虑的内容有很大不同，理解这些概念可以帮助我们避免细微的错误并编写更强大的代码。