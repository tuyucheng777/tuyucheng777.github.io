---
layout: post
title:  使用Mockito Mock嵌套方法调用
category: mock
copyright: mock
excerpt: Mockito
---

## 1. 概述

在本教程中，我们将了解如何使用Mockito存根(特别是深层存根)Mock嵌套方法调用。要了解有关使用Mockito进行测试的更多信息，请查看我们全面的[Mockito系列](https://www.baeldung.com/mockito-series)。

## 2. 解释问题

**在复杂代码(尤其是遗留代码)中，有时很难初始化单元测试所需的所有对象**。我们很容易在测试中引入许多不需要的依赖项，另一方面，Mock这些对象可能会导致[空指针异常](https://www.baeldung.com/java-exceptions#2-runtimeexceptions)。

让我们看一个代码示例，探讨这两种方法的局限性。

对于这两者，我们都需要一些类来测试。首先，让我们添加一个NewsArticle类：

```java
public class NewsArticle {
    String name;
    String link;

    public NewsArticle(String name, String link) {
        this.name = name;
        this.link = link;
    }

    // Usual getters and setters
}
```

此外，我们需要一个Reporter类：

```java
public class Reporter {
    String name;

    NewsArticle latestArticle;

    public Reporter(String name, NewsArticle latestArticle) {
        this.name = name;
        this.latestArticle = latestArticle;
    }

    // Usual getters and setters
}
```

最后，让我们创建一个NewsAgency类：

```java
public class NewsAgency {
    List<Reporter> reporters;

    public NewsAgency(List<Reporter> reporters) {
        this.reporters = reporters;
    }

    public List<String> getLatestArticlesNames(){
        List<String> results = new ArrayList<>();
        for(Reporter reporter : this.reporters){
            results.add(reporter.getLatestArticle().getName());
        }
        return results;
    }
}
```

理解它们之间的关系很重要。首先，NewsArticle由Reporter报道，而Reporter为NewsAgency工作。

NewsAgency包含一个getLatestArticlesNames()方法，该方法返回NewsAgency所有reporters撰写的最新文章的名称。此方法将被编写我们的单元测试。

让我们通过初始化所有对象来首次尝试这个单元测试。

## 3. 初始化对象

**在我们的测试中，作为第一种方法，我们将初始化所有对象**：


```java
public class NewsAgencyTest {
    @Test
    void getAllArticlesTest(){
        String title1 = "new study reveals the dimension where the single socks disappear";
        NewsArticle article1 = new NewsArticle(title1,"link1");
        Reporter reporter1 = new Reporter("Tom", article1);

        String title2 = "secret meeting of cats union against vacuum cleaners";
        NewsArticle article2 = new NewsArticle(title2,"link2");
        Reporter reporter2 = new Reporter("Maria", article2);

        List<String> expectedResults = List.of(title1, title2);

        NewsAgency newsAgency = new NewsAgency(List.of(reporter1, reporter2));
        List<String> actualResults = newsAgency.getLatestArticlesNames();
        assertEquals(expectedResults, actualResults);
    }
}
```

我们可以看到，当我们的对象变得越来越复杂时，所有对象的初始化都会变得很繁琐。**由于Mock正是为此目的，我们将使用它们来简化和避免繁琐的初始化**。

## 4. Mock对象

让我们[使用Mock](https://www.baeldung.com/mockito-mock-methods)来测试相同的方法getLatestArticlesNames()：

```java
@Test
void getAllArticlesTestWithMocks(){
    Reporter mockReporter1 = mock(Reporter.class);
    String title1 = "cow flying in London, royal guard still did not move";
    when(mockReporter1.getLatestArticle().getName()).thenReturn(title1);
    Reporter mockReporter2 = mock(Reporter.class);
    String title2 = "drunk man accidentally runs for mayor and wins";
    when(mockReporter2.getLatestArticle().getName()).thenReturn(title2);
    NewsAgency newsAgency = new NewsAgency(List.of(mockReporter1, mockReporter2));

    List<String> expectedResults = List.of(title1, title2);
    assertEquals(newsAgency.getLatestArticlesNames(), expectedResults);
}
```

如果我们尝试按原样执行此测试，我们将收到空指针异常。根本原因是对mockReporter1.getLastestArticle()的调用返回null，这是预期的行为：**Mock是对象的无效版本**。

## 5. 使用深层存根

[深层存根](https://www.baeldung.com/mockito-mocksettings#providing-a-different-default-answer)是Mock嵌套调用的简单解决方案，**深层存根可帮助我们利用Mock并仅存根测试中需要的调用**。

让我们在示例中使用它，我们将使用Mock和深度存根重写单元测试：

```java
@Test
void getAllArticlesTestWithMocksAndDeepStubs(){
    Reporter mockReporter1 = mock(Reporter.class, Mockito.RETURNS_DEEP_STUBS);
    String title1 = "cow flying in London, royal guard still did not move";
    when(mockReporter1.getLatestArticle().getName()).thenReturn(title1);
    Reporter mockReporter2 = mock(Reporter.class, Mockito.RETURNS_DEEP_STUBS);
    String title2 = "drunk man accidentally runs for mayor and wins";
    when(mockReporter2.getLatestArticle().getName()).thenReturn(title2);
    NewsAgency newsAgency = new NewsAgency(List.of(mockReporter1, mockReporter2));

    List<String> expectedResults = List.of(title1, title2);
    assertEquals(newsAgency.getLatestArticlesNames(), expectedResults);
}
```

**添加Mockito.RETURNS_DEEP_STUBS使我们能够访问所有嵌套方法和对象**。在我们的代码示例中，我们不需要Mock mockReporter1中的多层对象来访问mockReporter1.getLatestArticle().getName()。

## 6. 总结

在本文中，我们学习了如何使用深度存根来解决Mockito的嵌套方法调用问题。

我们应该记住，必须使用它们通常是违反[迪米特定律](https://www.baeldung.com/java-demeter-law)的症状，迪米特定律是面向对象编程中的一条指导原则，它有利于低耦合和避免嵌套方法调用。因此，深层存根应该留给遗留代码，在干净的现代代码中，我们应该倾向于[重构](https://www.baeldung.com/cs/refactoring)嵌套调用。