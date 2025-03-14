---
layout: post
title:  TestNG中BeforeTest和BeforeMethod之间的区别
category: unittest
copyright: unittest
excerpt: TestNG
---

## 1. 概述

使用Java中的[TestNG](https://www.baeldung.com/testng)时，高效管理测试设置和拆卸对于创建干净、独立且可维护的测试至关重要。**两个常用注解@BeforeTest和@BeforeMethod通过在测试生命周期的不同阶段运行设置代码来实现此目的**。

了解这些注解之间的区别并知道何时使用每个注解可以极大地提高我们的测试用例的组织和效率。

在本教程中，我们将详细探讨这些注解，研究它们的不同之处，并讨论每种注解最合适的场景。

## 2. Maven依赖

在深入研究注解之前，让我们确保已将[TestNG](https://mvnrepository.com/artifact/org.testng/testng)添加为Maven项目的pom.xml文件中的依赖项：

```xml
<dependency>
    <groupId>org.testng</groupId>
    <artifactId>testng</artifactId>
    <version>7.10.2</version>
    <scope>test</scope>
</dependency>
```

## 3. 什么是@BeforeTest？

**@BeforeTest注解在我们的测试类中任何带有@Test的测试方法之前执行一次方法**。

这对于设置共享资源非常理想，例如初始化应用程序上下文、操作对象或在同一个类中存在多种测试方法时设置数据库连接。

让我们看一个例子，在创建测试类之前，让我们创建一个名为Counter的类：

```java
public class Counter {
    private int totalCount;

    public Counter(int totalCount) {
        this.totalCount = totalCount;
    }

    public int getTotalCount() {
        return totalCount;
    }

    public void addCounter(int number) {
        this.totalCount = totalCount + number;
    }

    public void subtractCounter(int number) {
        this.totalCount = totalCount - number;
    }

    public void resetTotalCount() {
        this.totalCount = 0;
    }
}
```

让我们创建第一个名为BeforeTestAnnotationTest的测试类：

```java
public class BeforeTestAnnotationTest {
    private static final Logger log = LoggerFactory.getLogger(BeforeTestAnnotationTest.class);

    Counter counter;

    @BeforeTest
    public void init() {
        log.info("Initializing ...");
        counter = new Counter(0);
    }

    @Test
    public void givenCounterInitialized_whenAddingValue_thenTotalCountIncreased() {
        log.info("total counter before added: {}", counter.getTotalCount());
        counter.addCounter(2);
        log.info("total counter after added: {}", counter.getTotalCount());
    }

    @Test
    public void givenCounterInitialized_whenSubtractingValue_thenTotalCountDecreased() {
        log.info("total counter before subtracted: {}", counter.getTotalCount());
        counter.subtractCounter(1);
        log.info("total counter after subtracted: {}", counter.getTotalCount());
    }
}
```

现在，让我们运行BeforeTestAnnotationTest：

```text
Initializing ...
total counter before added: 0
total counter after added: 2
total counter before subtracted: 2
total counter after subtracted: 1
```

示例中的注解和方法协同工作的方式如下：

- 当givenCounterInitialized_whenAddingValue_thenTotalCountIncreased()运行时，它会将totalCount设置为0，如@BeforeTest所初始化的那样。
- 当givenCounterInitialized_whenSubtractingValue_thenTotalCountDecreased()运行时，它使用上一个测试givenCounterInitialized_whenAddingValue_thenTotalCountIncreased()留下的totalCount值，而不是重置为0。这解释了为什么尽管@BeforeTest最初将totalCount设置为0，但givenCounterInitialized_whenSubtractingValue_thenTotalCountDecreased()并未重新初始化totalCount。

## 4. 什么是@BeforeMethod？

**@BeforeMethod注解在测试类中的每个测试方法之前运行一个方法**，它用于在每次测试之前需要重置或重新初始化的任务，例如重置变量、清除数据或设置隔离条件。这可确保每个测试独立运行，从而避免跨测试依赖性。

在下面的示例中，我们将使用@BeforeMethod在每个测试方法之前重置Counter对象中的totalCount值。

让我们创建第二个测试类，名为BeforeMethodAnnotationTest：

```java
public class BeforeMethodAnnotationTest {
    private static final Logger log = LoggerFactory.getLogger(BeforeMethodAnnotationTest.class);

    Counter counter;

    @BeforeTest
    public void init() {
        log.info("Initializing ...");
        counter = new Counter(0);
    }

    @BeforeMethod
    public void givenCounterInitialized_whenResetTotalCount_thenTotalCountValueReset() {
        log.info("resetting total counter value ...");
        counter.resetTotalCount();
    }

    @Test
    public void givenCounterInitialized_whenAddingValue_thenTotalCountIncreased() {
        log.info("total counter before added: {}", counter.getTotalCount());
        counter.addCounter(2);
        log.info("total counter after added: {}", counter.getTotalCount());
    }

    @Test
    public void givenCounterInitialized_whenSubtractingValue_thenTotalCountDecreased() {
        log.info("total counter before subtracted: {}", counter.getTotalCount());
        counter.subtractCounter(2);
        log.info("total counter after subtracted: {}", counter.getTotalCount());
    }
}
```

现在，让我们运行BeforeMethodAnnotationTest：

```text
Initializing ...
resetting total counter value ...
total counter before added: 0
total counter after added: 2
resetting total counter value ...
total counter before subtracted: 0
total counter after subtracted: -2
```

示例中的注解和方法协同工作的方式如下：

- init()方法：此方法带有@BeforeTest注解，因此它在所有单元测试之前运行一次，将totalCount初始化为0以用于计数器。
- givenCounterInitialized_whenResetTotalCount_thenTotalCountValueReset()方法：使用@BeforeMethod注解，此方法在每个测试之前立即运行。它在每次测试执行之前将totalCount重置为0。
- 测试方法givenCounterInitialized_whenAddingValue_thenTotalCountIncreased()和givenCounterInitialized_whenSubtractingValue_thenTotalCountDecreased()：当这些测试运行时，totalCount总是从0开始，因为@BeforeMethod会在每次测试之前重置它。

## 5. 主要区别

了解@BeforeTest和@BeforeMethod之间的主要区别可以帮助我们决定何时有效地使用每个注解，下面详细介绍了它们的区别以及为什么它们在我们的测试中都很有价值：

|   方面   |                    @BeforeTest                     |         @BeforeMethod          |
| :------: | :--------------------------------------------------: | :------------------------------: |
| 执行范围 |              在任何测试方法之前运行一次              |   在类中的每个测试方法之前运行   |
|   用例   |   非常适合设置配置、启动跨多个测试共享的对象或资源   | 非常适合重置或准备每次测试的条件 |
| 典型用法 | 环境设置，初始化稍后要操作的对象，初始化数据库等资源 |  重置变量，准备特定于测试的配置  |

## 6. 何时使用每个注解

以下是如何根据用例在@BeforeTest和@BeforeMethod之间做出选择：

- 我们**在跨多个测试设置配置或共享资源时使用@BeforeTest**，例如初始化应用程序上下文、启动对象或设置共享数据库。
- 我们**在设置隔离条件或重置需要为每个测试独立的状态时使用@BeforeMethod**，例如重置对象值、重置数据库或在每次测试之前设置变量的默认值。

## 7. 总结

在TestNG中，@BeforeTest和@BeforeMethod提供了强大的工具来组织Java测试套件中的测试设置。通过了解它们的区别，我们可以将@BeforeTest用于运行一次的共享配置，将@BeforeMethod用于重置或设置特定于每个测试方法的条件。这有助于我们在Java中实现高效可靠的测试，尤其是在Spring应用程序等复杂环境中。