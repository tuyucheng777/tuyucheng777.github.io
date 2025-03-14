---
layout: post
title: Hamcrest中hasItems()、contains()和containsInAnyOrder()之间的区别
category: assertion
copyright: assertion
excerpt: Hamcrest
---

## 1. 简介

[Hamcrest](https://www.baeldung.com/hamcrest-core-matchers)是一个流行的Java匹配器对象编写框架，提供了一种定义匹配条件的富有表现力的方式。诸如hasItems()、contains()和containsInAnyOrder()之类的匹配器允许我们断言集合中元素的存在和顺序。在本教程中，我们将深入研究这些匹配器各自的功能、它们的区别以及何时使用它们。

## 2. 元素排序

在本节中，我们将探讨hasItems()、contains()和containsInAnyOrder()如何处理集合中元素的顺序。

### 2.1 hasItems()

hasItems()匹配器用于检查集合是否包含特定元素，而不关心它们的顺序：

```java
List<String> myList = Arrays.asList("apple", "banana", "cherry");

assertThat(myList, hasItems("apple", "cherry", "banana"));
assertThat(myList, hasItems("banana", "apple", "cherry"));
```

### 2.2 containsInAnyOrder()

与hasItems()类似，containsInAnyOrder()不考虑元素顺序，**它只关心指定的元素是否存在于集合中，而不管它们的顺序如何**：

```java
assertThat(myList, containsInAnyOrder("apple", "cherry", "banana"));
assertThat(myList, containsInAnyOrder("banana", "apple", "cherry"));
```

### 2.3 contains()

相比之下，contains()匹配器是顺序敏感的，它验证集合是否按提供的顺序包含指定的元素：

```java
assertThat(myList, contains("apple", "banana", "cherry"));
assertThat(myList, not(contains("banana", "apple", "cherry")));
```

第一个断言通过，因为元素的顺序完全匹配。在第二个断言中，我们使用not()匹配器，它期望myList不包含按精确顺序排列的元素。

## 3. 精确元素计数

在本节中，我们将讨论hasItems()、contains()和containsInAnyOrder()如何处理集合中元素的确切数量。

### 3.1 hasItems()

hasItems()匹配器不要求集合具有精确的元素数量，**它专注于确保特定元素的存在，而不对顺序或集合大小施加严格的要求**：

```java
assertThat(myList, hasItems("apple", "banana"));
assertThat(myList, hasItems("apple", "banana", "cherry"));
assertThat(myList, not(hasItems("apple", "banana", "cherry", "date")));
```

在第一个断言中，我们验证myList是否同时包含“apple”和“banana”。**由于这两项都存在，因此断言通过**。同样，第二个断言检查列表中是否存在“apple”、“banana”和“cherry”，因此它也通过了。

但是，第三个断言包含一个额外的元素“date”，而该元素不在myList中，因此，not()匹配器返回成功。**这说明，尽管hasItems()在验证元素是否存在方面很灵活，但如果在集合中发现除指定元素之外的额外元素，它会标记断言**。

### 3.2 containsInAnyOrder()

**hasItems()至少允许部分指定元素，而containsInAnyOrder()匹配器要求集合包含列表中的所有元素**。无论顺序如何，只要所有指定元素都存在，断言就会通过：

```java
assertThat(myList, containsInAnyOrder("apple", "banana", "cherry"));
assertThat(myList, not(containsInAnyOrder("apple", "banana")));
```

在第一个断言中，我们验证myList是否以任意顺序包含“apple”、“banana”和“cherry”。由于所有指定的元素都存在于myList中，因此断言通过。

第二个断言检查“apple”和“banana”，但忽略了“cherry”。在这里，我们期望not()会成功，因为myList必须与指定的元素完全匹配，包括“cherry”，而这个测试数据中缺少这个元素。

### 3.3 contains()

与containsInAnyOrder()类似，contains()匹配器也要求集合具有精确的元素数量，但要按照指定的顺序：

```java
assertThat(myList, contains("apple", "banana", "cherry"));
assertThat(myList, not(contains("apple", "banana")));
```

与containsInAnyOrder()类似，第二个断言期望not()匹配器成功，因为测试数据中缺少“cherry”元素。

## 4. 处理重复项

处理可能包含重复元素的集合时，必须了解hasItems()、contains()和containsInAnyOrder()的行为方式。

### 4.1 hasItems()

**hasItems()匹配器并不关心是否存在重复元素**，它只是检查指定元素是否存在，而不管它们是否重复：

```java
List<String> myListWithDuplicate = Arrays.asList("apple", "banana", "cherry", "apple");

assertThat(myListWithDuplicate, hasItems("apple", "banana", "cherry"));
```

在此断言中，我们验证myListWithDuplicate是否包含“apple”、“banana”和“cherry”。**hasItems()匹配器检查指定元素是否存在并忽略任何重复项**，由于所有指定元素都存在于列表中，因此断言通过。

### 4.2 containsInAnyOrder()

**类似地，containsInAnyOrder()不会强制元素的排序**。但是，它确保所有指定的元素都存在，无论是否有重复项：

```java
assertThat(myList, containsInAnyOrder("apple", "banana", "cherry", "apple"));
assertThat(myListWithDuplicate, not(containsInAnyOrder("apple", "banana", "cherry")));
```

在这种情况下，第二个断言测试数据中缺少“apple”重复元素，因此not()匹配器返回成功。

### 4.3 contains()

**另一方面，contains()匹配器需要精确匹配，包括元素的顺序和精确数量**。如果存在重复项但不在预期之内，则断言失败：

```java
assertThat(myList, not(contains("apple", "banana", "cherry")));
```

在这个断言中，“apple”重复元素在测试数据中缺失，因此not()匹配器返回成功。

## 5. 对比

让我们总结一下这些匹配器之间的主要区别：

|         匹配器          | 顺序 | 精确计数 |     处理重复项     |                目的                |
|:--------------------:|:--:|:----:| :----------------: | :--------------------------------: |
|      hasItems()      | 否  |  否   |     忽略重复项     | 检查指定元素是否存在，顺序无关紧要 |
| containsInAnyOrder() | 否  |  是   |   需要精确的元素   |   检查元素的精确度，顺序无关紧要   |
|      contains()      | 是  |  是   | 需要元素的精确顺序 |     检查确切的序列和确切的元素     |

## 6. 总结

在本文中，我们探讨了hasItems()、contains()和containsInAnyOrder()之间的区别。当我们只关心集合至少包含指定的元素而不管它们的顺序如何时，我们使用hasItems()。如果集合必须包含所有指定的元素，但顺序无关紧要，我们可以使用containsInAnyOrder()。但是，如果集合必须以提供的顺序包含精确的指定元素，我们应该使用contains()。