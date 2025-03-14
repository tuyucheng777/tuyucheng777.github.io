---
layout: post
title:  在JUnit 5方法参数中注入@Mock和@Captor
category: mock
copyright: mock
excerpt: Mockito
---

## 1. 概述

在本教程中，我们将了解如何在单元测试方法参数中注入[@Mock](https://www.baeldung.com/mockito-annotations#mock-annotation)和[@Captor](https://www.baeldung.com/mockito-annotations#captor-annotation)注解。

我们可以在单元测试中使用@Mock来创建Mock对象。另一方面，我们可以使用@Captor来捕获和存储传递给Mock方法的参数，以供以后断言。[JUnit 5](https://www.baeldung.com/junit-5)的引入使得[将参数注入测试方法](https://www.baeldung.com/junit-5-parameters)变得非常容易，为这一新功能腾出了空间。

## 2. 示例设置

为了使此功能正常工作，我们需要使用JUnit 5。可以在[Maven Central](https://mvnrepository.com/artifact/org.junit.jupiter/junit-jupiter-engine)中找到该库的最新版本，让我们将依赖项添加到pom.xml中：

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.10.2</version>
    <scope>test</scope>
</dependency>
```

[Mockito](https://site.mockito.org/)是一个测试框架，允许我们创建动态Mock对象。**Mockito Core提供了该框架的基本功能，包含用于创建和与Mock对象交互的富有表现力的API，让我们使用它的[最新版本](https://mvnrepository.com/artifact/org.mockito/mockito-core)**：

```xml
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>5.11.0</version>
    <scope>test</scope>
</dependency>
```

最后，我们需要使用Mockito JUnit Jupiter扩展，它负责将Mockito与JUnit 5集成，因此也将[此依赖](https://mvnrepository.com/artifact/org.mockito/mockito-junit-jupiter)添加到我们的pom.xml中：

```xml
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-junit-jupiter</artifactId>
    <version>5.12.0</version>
    <scope>test</scope>
</dependency>
```

## 3. 通过方法参数注入@Mock

首先，让我们将[Mockito Extension](https://www.baeldung.com/mockito-junit-5-extension)附加到我们的单元测试类：

```java
@ExtendWith(MockitoExtension.class)
class MethodParameterInjectionUnitTest {

    // ...
}
```

注册Mockito扩展允许Mockito框架与JUnit 5测试框架集成，因此，**我们现在可以提供我们的Mock对象作为测试参数**：

```java
@Test
void whenMockInjectedViaArgumentParameters_thenSetupCorrectly(@Mock Function<String, String> mockFunction) {
    when(mockFunction.apply("take")).thenReturn("today");
    assertEquals("today", mockFunction.apply("take"));
}
```

在此示例中，当我们传递“take”作为输入时，我们的Mock[函数](https://www.baeldung.com/java-8-functional-interfaces#Functions)返回字符串“today”。[断言](https://www.baeldung.com/junit-assertions)表明Mock的行为符合我们的预期。

此外，[构造函数](https://www.baeldung.com/java-constructors)是一种方法，因此也可以将@Mock作为测试类构造函数的参数注入：

```java
@ExtendWith(MockitoExtension.class)
class ConstructorInjectionUnitTest {
    
    Function<String, String> function;

    public ConstructorInjectionUnitTest(@Mock Function<String, String> functionr) {
        this.function = function;
    }
    
    @Test
    void whenInjectedViaArgumentParameters_thenSetupCorrectly() {
        when(function.apply("take")).thenReturn("today");
        assertEquals("today", function.apply("take"));
    }
}
```

总体来说，Mock注入并不局限于基本的单元测试，我们还可以把Mock注入到其他类型的可测试方法中，比如[重复测试](https://www.baeldung.com/junit-5-repeated-test)或者[参数化测试](https://www.baeldung.com/parameterized-tests-junit-5)：

```java
@ParameterizedTest
@ValueSource(strings = {"", "take", "today"})
void whenInjectedInParameterizedTest_thenSetupCorrectly(String input, @Mock Function<String, String> mockFunction) {
    when(mockFunction.apply(input)).thenReturn("taketoday");
    assertEquals("taketoday", mockFunction.apply(input));
}
```

最后，请注意，当我们将Mock注入参数化测试时，方法参数的顺序很重要。mockFunction注入的Mock必须位于输入测试参数之后，以便参数解析器正确完成其工作。

## 4. 通过方法参数注入@Captor

[ArgumentCaptor](https://www.baeldung.com/mockito-argumentcaptor)允许检查我们在测试中无法通过其他方式访问的对象的值，**我们现在可以以非常类似的方式通过方法参数注入@Captor**：

```java
@Test
void whenArgumentCaptorInjectedViaArgumentParameters_thenSetupCorrectly(@Mock Function<String, String> mockFunction, @Captor ArgumentCaptor<String> captor) {
    mockFunction.apply("taketoday");
    verify(mockFunction).apply(captor.capture());
    assertEquals("taketoday", captor.getValue());
}
```

在这个例子中，我们将Mock函数应用于字符串“taketoday”。然后，我们使用ArgumentCaptor提取传递给函数调用的值。最后，我们验证这个值是否正确。

我们关于Mock注入的所有讨论也适用于ArgumentCaptor，具体来说，这次让我们看一个@RepeatedTest中的注入示例：

```java
@RepeatedTest(2)
void whenInjectedInRepeatedTest_thenSetupCorrectly(@Mock Function<String, String> mockFunction, @Captor ArgumentCaptor<String> captor) {
    mockFunction.apply("taketoday");
    verify(mockFunction).apply(captor.capture());
    assertEquals("taketoday", captor.getValue());
}
```

## 5. 为什么使用方法参数注入？

现在我们来看看这个新功能的优点，首先，让我们回想一下以前我们是如何声明Mock的：

```java
Mock<Function> mock = mock(Mock.class)
```

在这种情况下，编译器会发出警告，因为Mockito.mock()无法正确创建Function的[泛型类型](https://www.baeldung.com/java-generics)。得益于方法参数注入，我们能够保留泛型类型签名，并且编译器不再抱怨。

使用方法注入的另一个巨大优势是发现依赖关系，以前，我们需要检查测试代码才能了解与其他类的交互。使用方法参数注入，**方法签名显示了我们的测试系统如何与其他组件交互**。此外，测试代码更短，更专注于其目标。

## 6. 总结

在本文中，我们了解了如何通过方法参数注入@Mock和@Captor，JUnit 5中对构造函数和方法依赖注入的支持启用了此功能。总之，建议使用此新功能。乍一看，这听起来可能只是锦上添花，但它可以提高我们的代码质量和可读性。