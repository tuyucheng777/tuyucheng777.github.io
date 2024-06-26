---
layout: post
title:  计算流过滤器上的匹配项
category: java-stream
copyright: java-stream
excerpt: Java Stream
---

## 1. 概述

**在本教程中，我们介绍Stream.count()方法的使用。具体来说，我们演示如何将count()方法与filter()方法结合起来计算我们应用的Predicate的匹配项**。

## 2. 使用Stream.count()

count()方法本身提供了一个很小但非常有用的功能。我们还可以将它与其他工具完美结合，例如Stream.filter()。

首先，我们重用[使用Lambda表达式的Java流过滤器](使用Lambda表达式的Java流过滤器.md)一文中使用到的Customer类：

```java
public class Customer {
	private String name;
	private int points;
	// Constructor and standard getters
}
```

此外，我们创建一个相同的Customer集合：

```java
Customer john = new Customer("John P.", 15);
Customer sarah = new Customer("Sarah M.", 200);
Customer charles = new Customer("Charles B.", 150);
Customer mary = new Customer("Mary T.", 1);

List<Customer> customers = Arrays.asList(john, sarah, charles, mary);
```

接下来，我们将在customers集合上应用Stream方法来过滤它并确定我们的过滤器获得了多少匹配项。

### 2.1 计数元素

让我们看看count()的最基本用法：

```java
long count = customers.stream().count();

assertThat(count).isEqualTo(4L);
```

请注意，count()返回一个long值。

### 2.2 使用count()和filter()

上一小节中的示例并没有给人留下深刻印象，我们可以使用List.size()方法得到相同的结果。

**当我们将Stream.count()与其他Stream方法结合使用时，它的作用就慢慢体现出来了，最常见的是与filter()结合使用**：

```java
long countBigCustomers = customers
    .stream()
    .filter(c -> c.getPoints() > 100)
    .count();

assertThat(countBigCustomers).isEqualTo(2L);
```

在此示例中，我们对customers集合应用了过滤器，并且我们还获得了满足条件的客户数量。在这种情况下，我们有两个客户的积分超过100。

当然，也可能没有元素匹配我们的过滤器：

```java
long count = customers
    .stream()
    .filter(c -> c.getPoints() > 500)
    .count();

assertThat(count).isEqualTo(0L);
```

### 2.3 将count()与高级过滤器一起使用

在我们关于filter()的教程中，我们看到了该方法的一些更高级的用例。当然，我们仍然可以统计这样的filter()操作的结果。

**我们可以使用多个条件过滤集合**：

```java
long count = customers
    .stream()
    .filter(c -> c.getPoints() > 10 && c.getName().startsWith("Charles"))
    .count();

assertThat(count).isEqualTo(1L);
```

在这里，我们过滤并统计了名字以“Charles”开头且积分超过10的客户的数量。

**我们还可以将条件提取到它自己的方法中并使用方法引用**：

```java
long count = customers
    .stream()
    .filter(Customer::hasOverHundredPoints)
    .count();

assertThat(count).isEqualTo(2L);
```

## 3. 总结

在本文中，我们演示了一些如何结合使用count()方法和filter()方法来处理流的示例。有关count()的更多用例，请查看其他返回Stream的方法，例如我们关于[使用concat()合并流]()的教程中演示的方法。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-streams-1)上获得。
