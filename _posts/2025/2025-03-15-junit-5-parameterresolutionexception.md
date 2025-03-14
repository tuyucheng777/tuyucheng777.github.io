---
layout: post
title:  解决JUnit 5中的ParameterResolutionException
category: unittest
copyright: unittest
excerpt: JUnit
---

## 1. 概述

JUnit 5引入了一些强大的功能，包括对[参数化测试](https://www.baeldung.com/parameterized-tests-junit-5)的支持。编写参数化测试可以节省大量时间，并且在许多情况下，只需简单的注解组合即可启用它们。

但是，**由于JUnit在后台管理测试执行的许多方面，因此不正确的配置可能会导致难以调试的异常**。

其中一个异常是ParameterResolutionException：

```text
org.junit.jupiter.api.extension.ParameterResolutionException: No ParameterResolver registered for parameter ...
```

在本教程中，我们将探讨此异常的原因以及如何解决它。

## 2. JUnit 5的ParameterResolver 

要了解此异常的原因，我们首先需要了解消息告诉我们缺少什么：ParameterResolver。

在JUnit 5中，引入了[ParameterResolver](https://www.baeldung.com/junit-5-parameters#parameterresolver)接口，允许开发人员扩展JUnit的基本功能并编写接收任意类型参数的测试，让我们看一个简单的ParameterResolver实现：

```java
public class FooParameterResolver implements ParameterResolver {
    @Override
    public boolean supportsParameter(ParameterContext parameterContext, ExtensionContext extensionContext) throws ParameterResolutionException {
        // Parameter support logic
    }

    @Override
    public Object resolveParameter(ParameterContext parameterContext, ExtensionContext extensionContext) throws ParameterResolutionException {
        // Parameter resolution logic
    }
}
```

我们可以看到该类有两个主要方法：

- supportsParameter()：确定是否支持参数类型
- resolveParameter()：返回测试执行的参数

由于在没有ParameterResolver实现的情况下会抛出ParameterResolutionException，因此我们暂时不会太关注实现细节，让我们首先讨论一下导致该异常的一些潜在原因。

## 3. ParameterResolutionException

ParameterResolutionException可能难以调试，特别是对于那些不太熟悉参数化测试的人来说。

首先，让我们定义一个简单的Book类，我们将为其编写单元测试：

```java
public class Book {
    private String title;
    private String author;
    // Standard getters and setters
}
```

在我们的示例中，我们将为Book编写一些单元测试来验证不同的title值，让我们从两个非常简单的测试开始：

```java
@Test
void givenWutheringHeights_whenCheckingTitleLength_thenTitleIsPopulated() {
    Book wuthering = new Book("Wuthering Heights", "Charlotte Bronte");
    assertThat(wuthering.getTitle().length()).isGreaterThan(0);
}

@Test
void givenJaneEyre_whenCheckingTitleLength_thenTitleIsPopulated() {
    Book jane = new Book("Jane Eyre", "Charlotte Bronte");
    assertThat(wuthering.getTitle().length()).isGreaterThan(0);
}
```

很容易看出这两个测试基本上在做同样的事情：设置图书title并检查长度。我们可以将它们合并为一个参数化测试来简化测试。让我们讨论一下这种重构可能出错的一些方式。

### 3.1 向@Test方法传递参数

采取一种非常快捷的方法，我们可能认为将参数传递给@Test注解方法就足够了：

```java
@Test
void givenTitleAndAuthor_whenCreatingBook_thenFieldsArePopulated(String title, String author) {
    Book book = new Book(title, author);
    assertThat(book.getTitle().length()).isGreaterThan(0);
    assertThat(book.getAuthor().length()).isGreaterThan(0);
}
```

代码编译并运行，但进一步思考一下，我们应该质疑这些参数来自哪里。运行此示例，我们看到一个异常：

```text
org.junit.jupiter.api.extension.ParameterResolutionException: No ParameterResolver registered for parameter [java.lang.String arg0] in method ...
```

**JUnit无法知道要传递给测试方法什么参数**。

让我们继续重构我们的单元测试并研究ParameterResolutionException的另一个原因。

### 3.2 竞争注解

正如前面提到的，我们可以使用ParameterResolver提供缺少的参数，但让我们从[值源](https://www.baeldung.com/parameterized-tests-junit-5#sources)开始更简单。由于只有两个值title和author-我们可以使用[CsvSource](https://www.baeldung.com/parameterized-tests-junit-5#4-csv-literals)将这些值提供给我们的测试。

此外，我们缺少一个关键注解：@ParameterizedTest，此注解告知JUnit我们的测试已参数化并已将测试值注入其中。

让我们快速尝试重构：

```java
@ParameterizedTest
@CsvSource({"Wuthering Heights, Charlotte Bronte", "Jane Eyre, Charlotte Bronte"})
@Test
void givenTitleAndAuthor_whenCreatingBook_thenFieldsArePopulated(String title, String author) {
    Book book = new Book(title, author);
    assertThat(book.getTitle().length()).isGreaterThan(0);
    assertThat(book.getAuthor().length()).isGreaterThan(0);
}
```

这似乎是合理的。然而，当我们运行单元测试时，我们看到了一些有趣的事情：两次测试通过，第三次测试失败。仔细观察，我们还会看到一个警告：

```text
WARNING: Possible configuration error: method [...] resulted in multiple TestDescriptors [org.junit.jupiter.engine.descriptor.TestMethodTestDescriptor, org.junit.jupiter.engine.descriptor.TestTemplateTestDescriptor].
This is typically the result of annotating a method with multiple competing annotations such as @Test, @RepeatedTest, @ParameterizedTest, @TestFactory, etc.
```

**通过添加竞争测试注解，我们无意中创建了多个TestDescriptor**，这意味着JUnit仍在运行我们测试的原始@Test版本以及我们新的参数化测试。

**只需删除@Test注解即可解决此问题**。

### 3.3 使用ParameterResolver

之前，我们讨论了ParameterResolver实现的一个简单示例 。现在我们有了一个可以运行的测试，让我们介绍一下BookParameterResolver：

```java
public class BookParameterResolver implements ParameterResolver {
    @Override
    public boolean supportsParameter(ParameterContext parameterContext, ExtensionContext extensionContext)
            throws ParameterResolutionException {
        return parameterContext.getParameter().getType() == Book.class;
    }

    @Override
    public Object resolveParameter(ParameterContext parameterContext, ExtensionContext extensionContext) throws ParameterResolutionException {
        return parameterContext.getParameter().getType() == Book.class
                ? new Book("Wuthering Heights", "Charlotte Bronte")
                : null;
    }
}
```

这是一个简单的例子，它只返回一个Book实例用于测试。现在我们有了一个ParameterResolver来为我们提供测试值，我们应该能够回到第一个示例中的测试。同样，我们可以尝试：

```java
@Test
void givenTitleAndAuthor_whenCreatingBook_thenFieldsArePopulated(String title, String author) {
    Book book = new Book(title, author);
    assertThat(book.getTitle().length()).isGreaterThan(0);
    assertThat(book.getAuthor().length()).isGreaterThan(0);
}
```

但正如我们在运行此测试时看到的那样，相同的异常仍然存在。但原因略有不同-**现在我们有了ParameterResolver，我们仍然需要告诉JUnit如何使用它**。

幸运的是，这很简单，只需**向包含我们的测试方法的外部类添加@ExtendWith注解即可**：

```java
@ExtendWith(BookParameterResolver.class)
public class BookUnitTest {
    @Test
    void givenTitleAndAuthor_whenCreatingBook_thenFieldsArePopulated(String title, String author) {
        // Test contents...
    }
    // Other unit tests
}
```

再次运行该程序，我们看到测试执行成功。

## 4. 总结

在本文中，我们讨论了JUnit 5的ParameterResolutionException以及缺失或竞争的配置如何导致此异常。