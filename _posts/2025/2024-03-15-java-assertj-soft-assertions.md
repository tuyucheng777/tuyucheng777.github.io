---
layout: post
title: 使用AssertJ进行软断言
category: assertion
copyright: assertion
excerpt: AssertJ
---

## 1. 简介

在本教程中，我们将研究[AssertJ](https://www.baeldung.com/introduction-to-assertj)的软断言功能，回顾其动机，并讨论其他测试框架中的类似解决方案。

## 2. 动机

首先，我们应该理解为什么软断言会存在。为此，让我们探讨以下示例：

```java
@Test
void test_HardAssertions() {
    RequestMapper requestMapper = new RequestMapper();

    DomainModel result = requestMapper.map(new Request().setType("COMMON"));

    Assertions.assertThat(result.getId()).isNull();
    Assertions.assertThat(result.getType()).isEqualTo(1);
    Assertions.assertThat(result.getStatus()).isEqualTo("DRAFT");
}
```

这段代码非常简单。我们有一个映射器，它将Request实体映射到某个DomainModel实例。我们想测试这个映射器的行为。当然，我们认为只有当所有断言都通过时，映射器才能正常工作。

现在，假设我们的映射器存在缺陷，并且错误地映射了id和status。在这种情况下，如果我们启动了这个测试，我们就会得到一个AssertionFailedError错误：

```text
org.opentest4j.AssertionFailedError: 
expected: null
but was: "73a3f292-8131-4aa9-8d55-f0dba77adfdb"
```

一切都很好，除了一件事-虽然id的映射确实是错误的，但status的映射也是错误的。测试没有告诉我们status映射不正确，它只抱怨id。发生这种情况是因为，默认情况下，当我们将断言与AssertJ一起使用时，我们将它们用作硬断言，这意味着测试中第一个未通过的断言将立即触发AssertionError。

乍一看，这似乎没什么大不了的，因为我们只需启动两次测试-第一次发现我们的ID映射是错误的，最后发现我们的status映射是错误的。但是，如果我们的映射器更复杂，并且它映射具有数十个字段的实体(这在实际项目中是可能的)，那么通过不断重新运行测试来找出所有问题将花费我们大量的时间。具体来说，为了解决这种不便，AssertJ为我们提供了软断言。

## 3. AssertJ中的软断言

软断言通过做一件非常简单的事情来解决这个问题-收集所有断言期间遇到的所有错误并生成单个报告，让我们通过重写上面的测试来看看软断言如何在AssertJ中帮助我们：

```java
@Test
void test_softAssertions() {
    RequestMapper requestMapper = new RequestMapper();

    DomainModel result = requestMapper.map(new Request().setType("COMMON"));

    SoftAssertions.assertSoftly(softAssertions -> {
        softAssertions.assertThat(result.getId()).isNull();
        softAssertions.assertThat(result.getType()).isEqualTo(1);
        softAssertions.assertThat(result.getStatus()).isEqualTo("DRAFT");
    });
}
```

在这里，我们要求AssertJ宽松地执行一系列断言，这意味着上面的[Lambda表达式](https://www.baeldung.com/java-8-functional-interfaces)中的所有断言无论如何都会执行，并且如果其中一些断言生成错误-这些错误将被打包成一个报告，如下所示：

```text
org.assertj.core.error.AssertJMultipleFailuresError: 
Multiple Failures (3 failures)
-- failure 1 --
expected: null
 but was: "66f8625c-b5e4-4705-9a49-94db3b347f72"
at SoftAssertionsUnitTest.lambda$test_softAssertions$0(SoftAssertionsUnitTest.java:19)
-- failure 2 --
expected: 1
 but was: 0
at SoftAssertionsUnitTest.lambda$test_softAssertions$0(SoftAssertionsUnitTest.java:20)
-- failure 3 --
expected: "DRAFT"
 but was: "NEW"
at SoftAssertionsUnitTest.lambda$test_softAssertions$0(SoftAssertionsUnitTest.java:21)
```

在调试过程中，它非常有用，可以缩短捕获所有错误所花费的时间。此外，值得一提的是，在AssertJ中还有另一种使用软断言编写测试的方法：

```java
@Test
void test_softAssertionsViaInstanceCreation() {
    RequestMapper requestMapper = new RequestMapper();

    DomainModel result = requestMapper.map(new Request().setType("COMMON"));

    SoftAssertions softAssertions = new SoftAssertions();
    softAssertions.assertThat(result.getId()).isNull();
    softAssertions.assertThat(result.getType()).isEqualTo(1);
    softAssertions.assertThat(result.getStatus()).isEqualTo("DRAFT");
    softAssertions.assertAll();
}
```

在这种情况下，我们只是直接创建SoftAssertions的一个实例，与前面的示例相反，在前面的示例中，仍会创建SoftAssertions实例，但是在框架之下。

至于这两个变体之间的差异，从功能角度来看，它们是完全相同的，所以我们可以自由选择我们想要的任何方法。

## 4. 其他测试框架中的软断言

由于软断言这一功能非常有用，因此许多测试框架已经采用了它。棘手的是，在不同的框架中，此功能可能具有不同的名称，甚至可能根本没有名称。例如，我们在JUnit 5中有一个[assertAll()](https://www.baeldung.com/junit5-assertall-vs-multiple-assertions)，它以类似的方式工作，但有自己的风格。[TestNG也具有软断言这一功能](https://www.baeldung.com/java-testng-continue-test-post-failure)，它与AssertJ中的功能非常相似。

因此，重点是大多数著名的测试框架都具有软断言作为功能，尽管它们可能没有这个名字。

## 5. 总结

让我们最终在一张表中总结一下硬断言和软断言之间的所有区别和相似之处：

|                     | 硬断言 | 软断言 |
|:-------------------:|:---:| :----: |
|       是默认断言模式       | true  | false |
| 大多数框架都支持(包括AssertJ) | true  |  true  |
|      展现快速失败行为       | true  | false |

## 6. 总结

在本文中，我们探讨了软断言及其动机。与硬断言相反，软断言允许我们继续执行测试，即使我们的某些断言失败，在这种情况下框架会生成详细报告。这使得软断言成为一个有用的功能，因为它们使调试更容易、更快。

最后，软断言并不是AssertJ独有的特性。它们也存在于其他流行的框架中，可能名称不同或没有名称。