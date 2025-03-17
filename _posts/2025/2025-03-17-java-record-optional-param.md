---
layout: post
title:  Java中作为记录参数的Optional参数
category: java
copyright: java
excerpt: Java Record
---

## 1. 简介

在本教程中，我们将讨论使用Optional作为[记录](https://www.baeldung.com/java-record-keyword)参数的可能性以及为什么这是一种不好的做法。

## 2. Optional的预期用途

在讨论Optional和记录的关系之前，让我们快速回顾一下Java中[Optional的预期用途](https://www.baeldung.com/java-optional-uses)。

通常，在Java 8之前，我们使用null来表示对象的空状态。但是，将null作为返回值需要在运行时从调用方代码进行null检查验证。如果调用方未进行验证，则可能会收到NullPointerException，获取异常有时用于识别值的缺失。

**Optional的主要目标是表示一个表示值缺失的方法返回值**，我们可以使用[Optional作为返回值](https://www.baeldung.com/java-optional-return)，而不是让我们的应用程序因NullPointerException而崩溃，以识别值的缺失。因此，我们在编译时就知道返回值是包含内容还是不包含内容。

此外，正如[Java文档](https://docs.oracle.com/en/java/javase/21/docs//api/java.base/java/util/Optional.html)中所述：

> Optional旨在**为库方法返回类型提供一种有限的机制**，这种情况下，明确需要表示“无结果”，而使用null极有可能导致错误

因此，还必须注意Optional不打算做什么。在这方面，我们可以强调**Optional不打算用作任何类的实例字段**。

## 3. Java记录的用例

我们还可以了解一些有关记录的概念，以便更好地了解如何使用Optional作为记录参数。

记录只是数据载体，当我们想将数据从一个地方传输到另一个地方时，它很适合，例如从[数据库传输到我们的应用程序](https://www.baeldung.com/spring-jpa-java-records)。

让我们解释一下[JEP-395](https://openjdk.org/jeps/395)：

> 记录是作为不可变数据的透明载体的类。

记录的一个关键定义是它们是不可变的，因此，**一旦我们实例化一条记录，其所有数据在程序的其余部分都保持不可修改**。这对于传输数据的对象来说非常好，因为不可变对象不容易出错。

记录还会自动定义具有相同字段名称的访问器方法，因此，通过定义它们，我们可以获得与定义字段相同的Getter。

JDK记录定义还表明记录所保存的数据应该是透明的。因此，**如果我们调用访问器方法，我们应该获得有效数据**。在这种情况下，有效数据意味着真正代表对象状态的值。让我们用[Project Amber](https://openjdk.org/projects/amber/design-notes/records-and-sealed-classes)的话来解释一下：

> 数据类(记录)的API对状态、整个状态以及仅状态进行建模。

不变性和透明度是捍卫记录不能有可选参数这一论点的基本定义。

## 4. Optional作为记录参数

现在我们对这两个概念有了更好的理解，我们将明白为什么必须避免使用Optional作为记录参数。

首先我们定义一个记录示例：

```java
public record Product(String name, double price, String description) {
}
```

我们为Product定义了一个数据持有者，其中包含name、price和description。我们可以想象数据持有者是由数据库查询或HTTP调用产生的。

现在，我们假设有时未设置产品描述。在这种情况下，描述可为空。解决该问题的一种方法是将description字段包装到Optional对象中：

```java
public record Product(String name, double price, Optional<String> description) {
}
```

尽管上述代码编译正确，**但我们破坏了产品记录的数据透明度**。

此外，**记录不可变性使得处理Optional实例比处理空变量更困难**，让我们通过一个简单的测试来实际看看：

```java
@Test
public void givenRecordCreationWithOptional_thenCreateItProperly() {
    var emptyDescriptionProduct = new Product("television", 1699.99, Optional.empty());
    Assertions.assertEquals("television", emptyDescriptionProduct.name());
    Assertions.assertEquals(1699.99, emptyDescriptionProduct.price());
    Assertions.assertNull(emptyDescriptionProduct.description().orElse(null));
}
```

我们创建了一个具有一些值的产品，并使用生成的Getter来断言记录已正确实例化。

在我们的产品中，我们定义了变量name、price和description。但是，由于description是Optional，因此我们在检索它之后不会立即获得值。我们需要做一些逻辑来打开它以获取值。换句话说，我们在调用访问器方法后没有获得正确的对象状态。因此，它破坏了Java记录数据透明度的定义。

在这种情况下，我们可能会想，我们该如何处理null呢？好吧，我们可以简单地让它们存在。null表示对象的空状态，在这种情况下，它比空的Optional实例更有意义。在这些情况下，我们可以使用@Nullable注解或其他[处理null的良好做法](https://www.baeldung.com/java-avoid-null-check)来通知Product类的用户description可以为空。

由于记录字段是不可变的，因此description字段不能更改。因此，要检索description值，我们有一些选择。一种是使用[orElse()和orElseGet()](https://www.baeldung.com/java-optional-or-else-vs-or-else-get)打开它或返回默认值。另一种方法是盲目使用get()，如果没有值，它会抛出NoSuchElementException。第三种方法是使用[orElseThrow()](https://www.baeldung.com/java-optional-throw-exception)，如果里面没有任何内容，则抛出错误。

**在任何可能的处理方式中，Optional都没有意义，因为在任何情况下，我们要么返回null，要么抛出错误**。让description成为一个可空的String会更简单。

## 5. Optional作为返回类型

在本节中，我们将探讨如何有效地使用Optional作为返回类型。让我们考虑一个场景，其中我们有一个代表用户信息的用户记录：

```java
public record User(String username, String email, String phoneNumber) {
    public Optional<String> getOptionalPhoneNumber() {
        return Optional.ofNullable(phoneNumber);
    }
}
```

在此示例中，我们有一个用户记录，其中包含username、email和phoneNumber字段。getOptionalPhoneNumber()方法返回一个Optional<String\>，表示电话号码可能存在也可能不存在：

```java
@Test
public void givenRecordCreationWithNullOptional_thenReturnOptional() {
    User user = new User("john_doe", "john@example.com", null);
    Optional<String> optionalPhoneNumber = user.getOptionalPhoneNumber();

    Assertions.assertEquals(Optional.empty(), optionalPhoneNumber);
}
```

通过返回Optional，我们可以决定如何响应-是否提示用户输入电话号码、显示默认消息、或者继续：

```java
optionalPhoneNumber.ifPresentOrElse(
    phone -> // handle if phone present
    () -> // handle if phone is absent
);
```

以这种方式使用Optional作为返回类型有助于通过明确解决值缺失问题来提高代码的安全性。**它提供了一种避免NullPointerException的方法，并鼓励更好地处理可能缺失的信息**。

## 6. 总结

在本文中，我们了解了Java记录的定义并了解了透明度和不变性的重要性。

我们还研究了Optional类的预期用途，更重要的是，我们讨论了Optional不适合用作记录参数。相反，Optional可以更有效地用作方法中的返回类型。