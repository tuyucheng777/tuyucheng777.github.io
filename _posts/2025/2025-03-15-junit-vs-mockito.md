---
layout: post
title:  JUnit和Mockito之间的区别
category: unittest
copyright: unittest
excerpt: JUnit
---

## 1. 简介

软件测试是软件开发生命周期中的关键阶段。它帮助我们评估、识别和改进[软件的质量](https://www.baeldung.com/cs/software-quality)。

测试框架通过预定义的工具帮助我们简化这一过程。在本文中，我们将讨论JUnit、Mockito、它们如何帮助我们以及这两个框架之间的差异。

## 2. 什么是JUnit？

JUnit是最广泛使用的单元测试框架之一，[JUnit 5](https://www.baeldung.com/junit-5)是其最新一代。此版本专注于Java 8及以上版本，并支持各种测试风格。

JUnit 5使用断言、注解和测试运行器运行测试用例，该框架主要用于[单元测试](https://www.baeldung.com/java-unit-testing-best-practices)。它主要关注方法和类，与项目的其他元素隔离。

与以前的版本不同，JUnit 5由三个不同的子项目组成：

- JUnit Platform：在JVM上启动测试框架的关键
- JUnit Jupiter：引入了新的编程模型(编写测试的要求)和扩展模型(Extension API)来编写新一代测试
- JUnit Vintage：确保与使用JUnit 3和[Junit 4](https://www.baeldung.com/junit-4-rules)编写的测试兼容

对于运行时，JUnit 5需要Java 8(或更高版本)。但是，已使用以前版本的JDK编译的代码仍可进行测试。

通用的JUnit测试类和方法如下所示：

```java
public class JunitVsMockitoUnitTest {
    @Test
    public void whenUsingJunit_thenObjectCanBeInstantiated() {
        InstantiableClassForJunit testableClass = new InstantiableClassForJunit();

        assertEquals("tested unit", testableClass.testableComponent());
    }
}
```

在上面的例子中，我们创建了一个简单的InstantiableClassForJunit类，它只有一个返回String的方法。此外，查看Github中的类，我们会看到，对于assertEquals()方法，我们导入了Assertions类。这是一个明显的例子，我们使用的是JUnit 5(Jupiter)，而不是旧版本。

## 3. 什么是Mockito？

[Mockito](https://www.baeldung.com/mockito-series)是一个基于Java的框架，由Szczepan Faber和他的朋友开发。它提出了一种不同的方法(与使用expect-run-verify的传统模拟库不同)，我们在执行后询问有关交互的问题。

这种方法还意味着Mockito Mock通常不需要事先进行昂贵的设置。此外，它有一个精简的API，可以快速启动Mock。只有一种Mock，并且只有一种创建Mock的方法。

Mockito的其他一些重要功能包括：

- Mock具体类和接口的能力
- 小注解语法糖–@Mock
- 清除指向代码行的错误消息
- 创建自定义参数匹配器的能力

以下是添加Mockito后上述代码的样子：

```java
@ExtendWith(MockitoExtension.class)
public class JunitVsMockitoUnitTest {
    @Mock 
    NonInstantiableClassForMockito mock;

    // the previous Junit method

    @Test
    public void whenUsingMockito_thenObjectNeedsToBeMocked() {
        when(mock.nonTestableComponent()).thenReturn("mocked value");

        assertEquals("mocked value", mock.nonTestableComponent());
    }
}
```

查看同一个类，我们注意到我们在类名上添加了注解(@ExtendWith)。还有其他方法可以启用Mockito，但我们在此不做详细介绍。

接下来，我们使用@Mock注解Mock了所需类的实例。最后，在测试方法中，我们使用了when().thenReturn()构造。

最好记住在assert方法之前使用此构造，这让Mockito知道，当调用Mock类的特定方法时，它应该返回提到的值。在我们的例子中，这是“mocked value”，而不是它通常会返回的“some result”。

## 4. JUnit和Mockito之间的区别

### 4.1.测试用例结构

JUnit使用注解来创建其结构，例如，我们在方法上方使用@Test来表示该方法是测试方法，或者使用@ParametrizedTest来表示测试方法将使用不同的参数多次运行。我们使用@BeforeEach、@AfterEach、@BeforeAll和@AfterAll来表示我们希望该方法在一个或所有测试方法之前或之后执行。

另一方面，Mockito可以帮助我们处理这些带注解的方法。Mockito提供了创建Mock对象、配置其行为(它们返回的内容)以及验证发生的某些交互(是否调用该方法、调用了多少次、使用了哪种类型的参数等)的方法。

### 4.2 测试范围

正如我们之前提到的，我们使用JUnit进行单元测试，这意味着我们创建逻辑来单独测试各个组件(方法)。接下来，我们使用JUnit运行测试逻辑并确认测试结果。

**另一方面，Mockito是一个框架，可帮助我们生成某些类的对象(Mock)并在测试期间控制其行为**。Mockito更多地关注测试期间的交互，而不是实际进行测试本身。我们可以使用Mockito在测试期间Mock外部依赖关系。

例如，我们应该从端点接收答案。使用Mockito，我们可以模拟该端点，并在测试期间调用它时决定其行为。这样，我们就不必再实例化该对象了。此外，有时我们甚至无法实例化该对象，除非重构它。

### 4.3 测试替身

JUnit的设计侧重于对象和测试替身的具体实现，后者指的是伪造和存根(而非Mock)的实现。测试替身模拟真实的依赖关系，但对于我们试图实现的目标而言，其行为有限。

另一方面，Mockito适用于动态对象(使用反射创建的Mock)。当我们使用这些Mock时，我们可以个性化和控制它们的行为以满足我们的测试需求。

### 4.4 测试覆盖率

[JaCoCo](https://www.baeldung.com/jacoco)是一个与JUnit配合使用的测试覆盖框架，而不是Mockito。这是两个框架之间差异的另一个明显例子。

Mockito是与JUnit配合使用的框架，JaCoCo(或其他代码覆盖库)只能与测试框架一起使用。

### 4.5 对象Mock

使用Mockito，我们可以指定Mock对象的期望和行为，从而加快所需测试场景的创建。

JUnit更侧重于使用断言进行单个组件测试，它没有内置的Mock功能。

## 5. 总结

在本文中，我们了解了Java生态系统中两个最流行的测试框架。我们了解到JUnit是主要的测试促进者，它帮助我们创建结构和适当的环境。

Mockito是一个补充JUnit的框架，它通过Mock元素来帮助我们测试单个组件，否则这些元素实例化起来会非常复杂，甚至根本无法创建。此外，Mockito帮助我们控制这些元素的输出。最后，我们还可以检查所需的行为是否发生以及发生了多少次。