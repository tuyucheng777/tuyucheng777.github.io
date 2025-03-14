---
layout: post
title:  TestNG中断言失败后仍继续测试
category: unittest
copyright: unittest
excerpt: TestNG
---

## 1. 概述

[TestNG](https://www.baeldung.com/testng)是一种流行的Java测试框架，是JUnit的替代品。虽然这两个框架都提供了自己的范例，但它们都包含断言的概念：如果逻辑语句的计算结果为false，则停止程序执行，从而导致测试失败。TestNG中的简单断言可能如下所示：

```java
@Test 
void testNotNull() {
    assertNotNull("My String"); 
}
```

但是如果我们需要在单个测试中做出多个断言，会发生什么情况呢？**在本文中，我们将探讨TestNG的SoftAssert，这是一种同时执行多个断言的技术**。

## 2. 设置

为了练习，我们定义一个简单的Book类：

```java
public class Book {
    private String isbn;
    private String title;
    private String author;

    // Standard getters and setters...
}
```

我们还可以定义一个接口来模拟一个根据ISBN查找书籍的简单服务：

```java
interface BookService {
    Book getBook(String isbn);
}
```

然后，我们可以在单元测试中Mock此服务，稍后我们将对其进行定义。此设置让我们可以定义一个可以以现实方式测试的场景：返回可能为空或其成员变量可能为空的对象的服务，让我们开始为此编写单元测试。

## 3. 基本断言与TestNG的SoftAssert

为了说明SoftAssert的好处，我们首先使用失败的基本TestNG断言创建一个单元测试，然后将我们获得的反馈与使用SoftAssert进行的相同测试进行比较。

### 3.1 使用传统断言

首先，我们将使用assertNotNull()创建一个测试，它接收一个要测试的值和一个可选消息：

```java
@Test
void givenBook_whenCheckingFields_thenAssertNotNull() {
    Book gatsby = bookService.getBook("9780743273565");

    assertNotNull(gatsby.isbn, "ISBN");
    assertNotNull(gatsby.title, "title");
    assertNotNull(gatsby.author, "author");
}
```

然后，我们将定义BookService的Mock实现(使用[Mockito](https://www.baeldung.com/mockito-series))，返回一个Book实例：

```java
@BeforeMethod
void setup() {
    bookService = mock(BookService.class);
    Book book = new Book();
    when(bookService.getBook(any())).thenReturn(book);
}
```

运行测试，我们可以看到我们忽略了设置isbn字段：

```text
java.lang.AssertionError: ISBN expected object to not be null
```

让我们在Mock中修复这个问题并再次运行测试：

```java
@BeforeMethod void setup() {
    bookService = mock(BookService.class);
    Book book = new Book();
    book.setIsbn("9780743273565");
    when(bookService.getBook(any())).thenReturn(book);
}
```

我们现在得到一个不同的错误：

```text
java.lang.AssertionError: title expected object to not be null
```

再次，我们忘记在Mock中初始化字段，导致另一个必要的改变。

我们可以看到，**测试、更改和重新运行测试的这个循环不仅令人沮丧，而且耗时**。当然，这种影响会随着类的大小和复杂性而成倍增加。在集成测试的情况下，这个问题会进一步加剧。远程部署环境中的故障可能很难或不可能在本地重现，集成测试通常更复杂，因此执行时间更长。再加上部署测试更改所需的时间，意味着每次额外测试重新运行的循环时间成本高昂。

幸运的是，**我们可以通过使用SoftAssert来评估多个断言而无需立即停止程序执行来避免这个问题**。

### 3.2 使用SoftAssert对断言进行分组

让我们更新上面的示例以使用SoftAssert：

```java
@Test void givenBook_whenCheckingFields_thenAssertNotNull() {
    Book gatsby = bookService.getBook("9780743273565"); 
    
    SoftAssert softAssert = new SoftAssert();
    softAssert.assertNotNull(gatsby.isbn, "ISBN");
    softAssert.assertNotNull(gatsby.title, "title");
    softAssert.assertNotNull(gatsby.author, "author");
    softAssert.assertAll();
}
```

让我们详细分析一下：

- 首先，我们创建一个SoftAssert实例
- 接下来，我们做一个关键的改变：**针对SoftAssert的实例进行断言**，而不是使用TestNG的基本assertNonNull()方法
- 最后，同样重要的是要注意，**一旦我们准备好获取所有断言的结果，我们就需要在SoftAssert实例上调用assertAll()方法**

现在，如果我们使用原始Mock来运行它，而忽略了为Book设置任何成员变量值，我们将看到一条包含所有断言失败的错误消息：

```text
java.lang.AssertionError: The following asserts failed:
    ISBN expected object to not be null,
    title expected object to not be null,
    author expected object to not be null
```

这表明，当单个测试需要多个断言时，使用SoftAssert是一种很好的做法。

### 3.3 SoftAssert注意事项

虽然SoftAssert易于设置和使用，但有一个重要的考虑因素需要牢记：状态性。由于SoftAssert在内部记录每个断言的失败，因此不适合在多个测试方法之间共享。因此，**我们应确保在每个测试方法中创建一个新的SoftAssert实例**。

## 4. 总结

在本教程中，我们学习了如何使用TestNG的SoftAssert进行多重断言，以及它如何成为编写干净测试并减少调试时间的宝贵工具。我们还了解到SoftAssert是有状态的，实例不应在多个测试之间共享。