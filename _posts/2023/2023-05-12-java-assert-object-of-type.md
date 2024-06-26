---
layout: post
title:  断言对象是否为特定类型
category: assertion
copyright: assertion
excerpt: Hamcrest
---

## 1. 概述

在本文中，我们介绍如何验证对象是否属于特定类型；使用不同的测试库以及它们提供的断言对象类型的方法。

我们需要执行此操作的场景可能会有所不同，一种常见的情况是，当我们使用接口作为方法的返回类型，然后根据返回的具体对象，我们想要执行不同的操作。**单元测试可以帮助我们确定返回的对象是否是我们期望的类型**。

## 2. 示例场景

想象一下，我们根据树木是否在冬天落叶来对它们进行分类；我们有两个类Evergreen和Deciduous，它们都实现了Tree接口，并且有一个简单的排序器，它根据树的名称返回正确的类型：

```java
protected Tree sortTree(String name) {
	final List<String> deciduous = List.of("Beech", "Birch", "Ash", "Whitebeam", "Hornbeam", "Hazel & Willow");
	final List<String> evergreen = List.of("Cedar", "Holly", "Laurel", "Olive", "Pine");
    
	if (deciduous.contains(name)) {
		return new Deciduous(name);
	} else if (evergreen.contains(name)) {
		return new Evergreen(name);
	} else {
		throw new RuntimeException("Tree could not be classified");
	}
}
```

接下来我们介绍如何测试实际返回的Tree类型。

### 2.1 使用JUnit 5

如果我们想使用JUnit 5，**我们可以使用assertEquals方法检查对象的类是否等于我们正在测试的类**：

```java
@Test
void sortTreeShouldReturnEvergreen_WhenPineIsPassed() {
    Tree tree = tested.sortTree("Pine");
    assertEquals(tree.getClass(), Evergreen.class);
}
```

### 2.2 使用Hamcrest

使用Hamcrest库时，**我们可以使用assertThat和instanceOf方法**：

```java
@Test
void sortTreeShouldReturnEvergreen_WhenPineIsPassed() {
    Tree tree = tested.sortTree("Pine");
    assertThat(tree, instanceOf(Evergreen.class));
}
```

当我们使用org.hamcrest.Matchers.isA导入时，我们可以使用另一个快捷版本：

```java
assertThat(tree, isA(Evergreen.class));
```

### 2.3 使用AssertJ

我们还可以**使用AssertJ核心库的isExactlyInstanceOf方法**：

```java
@Test
void sortTreeShouldReturnEvergreen_WhenPineIsPassed() {
    Tree tree = tested.sortTree("Pine");
    assertThat(tree).isExactlyInstanceOf(Evergreen.class);
}
```

实现相同测试功能的另一种方法是**使用hasSameClassAs**：

```java
@Test
void sortTreeShouldReturnDecidious_WhenBirchIsPassed() {
    Tree tree = tested.sortTree("Birch");
    assertThat(tree).hasSameClassAs(new Deciduous("Birch"));
}
```

## 3. 总结

在本教程中，我们演示了一些在单元测试中验证对象类型的不同示例，我们演示了一个简单的Junit 5示例以及使用Hamcrest和AssertJ库的方法；Hamcrest和AssertJ在其错误消息中都提供了额外的有用信息。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/software.test/testing-assertions)上获得。