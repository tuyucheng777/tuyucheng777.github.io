---
layout: post
title:  修复Mockito中的模糊方法调用错误
category: mock
copyright: mock
excerpt: Mockito
---

## 1. 概述

在本教程中，我们将了解如何避免在[Mockito](https://site.mockito.org/)框架的特定上下文中出现歧义的方法调用。

在Java中，方法重载允许一个类拥有多个名称相同但参数不同的方法。当编译器无法根据提供的参数确定要调用的具体方法时，就会发生模糊的方法调用。

## 2. 介绍Mockito的ArgumentMatchers

Mockito是一个用于单元测试Java应用程序的Mock框架，可以在[Maven Central](https://mvnrepository.com/artifact/org.mockito/mockito-core)中找到该库的最新版本，让我们将依赖项添加到pom.xml中：

```xml
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>5.11.0</version>
    <scope>test</scope>
</dependency>
```

**[ArgumentMatchers](https://www.baeldung.com/mockito-argument-matchers)是Mockito框架的一部分：借助它们，我们可以指定当参数匹配给定条件时Mock方法的行为**。

## 3. 重载方法定义

首先，让我们定义一个以Integer作为参数并始终返回1作为结果的方法：

```java
Integer myMethod(Integer i) {
    return 1;
}
```

为了演示，我们希望重载方法使用自定义类型。因此，我们定义这个虚拟类：

```java
class MyOwnType {}
```

我们现在可以添加一个重载的myMethod()，它接收MyOwnType对象作为参数并始终返回taketoday作为结果：

```java
String myMethod(MyOwnType myOwnType) {
    return "taketoday";
}
```

**直观地讲，如果我们将空参数传递给myMethod()，编译器将不知道应该使用哪个版本**。此外，我们可以注意到该方法的返回类型对此问题没有影响。

## 4. isNull()调用不明确

让我们天真地尝试使用基本isNull() ArgumentMatcher模拟对myMethod()的调用，并使用空参数：

```java
@Test
void givenMockedMyMethod_whenMyMethod_ThenMockedResult(@Mock MyClass myClass) {
    when(myClass.myMethod(isNull())).thenReturn(1);
}
```

假设我们将myMethod()定义为MyClass的类，我们通过测试的方法参数很好地[注入了一个Mock的MyClass对象](https://www.baeldung.com/junit-5-mock-captor-method-parameter-injection#injecting-mock-through-method-parameters)。我们还可以注意到，我们还没有向测试添加任何断言。让我们运行此代码：

```text
java.lang.Error: Unresolved compilation problem: 
The method myMethod(Integer) is ambiguous for the type MyClass
```

我们可以看到，编译器无法决定使用哪个版本的myMethod()，因此会抛出错误。我们要强调的是，编译器的决定仅基于方法参数。由于我们在指令中编写了thenReturn(1)，因此作为读者，我们可以猜测其意图是使用返回Integer的myMethod()版本。但是，编译器不会在其决策过程中使用指令的这一部分。

**为了解决这个问题，我们需要使用重载的isNull() ArgumentMatcher，以类作为参数**。例如，为了告诉编译器它应该使用以Integer作为参数的版本，我们可以这样写：

```java
@Test
void givenMockedMyMethod_whenMyMethod_ThenMockedResult(@Mock MyClass myClass) {
    when(myClass.myMethod(isNull(Integer.class))).thenReturn(1);
    assertEquals(1, myClass.myMethod((Integer) null));
}
```

我们添加了一个断言来完成测试，现在它成功运行了。同样，我们可以修改测试以使用该方法的其他版本：

```java
@Test
void givenCorrectlyMockedNullMatcher_whenMyMethod_ThenMockedResult(@Mock MyClass myClass) {
    when(myClass.myMethod(isNull(MyOwnType.class))).thenReturn("taketoday");
    assertEquals("taketoday", myClass.myMethod((MyOwnType) null));
}
```

最后，我们要注意，在断言中，我们也需要在对myMethod()的调用中给出null类型。否则，由于同样的原因，这将抛出错误！

## 5. any()的模糊调用

以同样的方式，我们可以尝试使用any() ArgumentMatcher模拟接收任何参数的myMethod()调用：

```java
@Test
void givenMockedMyMethod_whenMyMethod_ThenMockedResult(@Mock MyClass myClass) {
    when(myClass.myMethod(any())).thenReturn(1);
}
```

再次运行此代码会导致模糊方法调用错误，我们在上一个案例中所做的所有注释在这里仍然有效。特别是，编译器甚至在查看thenReturn()方法的参数之前就失败了。

解决方案也类似：**我们需要使用any() ArgumentMatcher的版本，明确说明预期参数的类型**：

```java
@Test
void givenMockedMyMethod_whenMyMethod_ThenMockedResult(@Mock MyClass myClass) {
    when(myClass.myMethod(anyInt())).thenReturn(1);
    assertEquals(1, myClass.myMethod(2));
}
```

大多数基础Java类型已经为此目的定义了Mockito方法，在我们的例子中，anyInt()方法将接收任何Integer参数。另一方面，myMethod()的另一个版本接收我们自定义的MyOwnType类型的参数。因此，我们需要使用any() ArgumentMatcher的重载版本，该版本将对象的类型作为参数：

```java
@Test
void givenCorrectlyMockedNullMatcher_whenMyMethod_ThenMockedResult(@Mock MyClass myClass) {
    when(myClass.myMethod(any(MyOwnType.class))).thenReturn("taketoday");
    assertEquals("taketoday", myClass.myMethod((MyOwnType) null));
}
```

测试现在运行良好：我们成功消除了歧义的方法调用错误！

## 6. 总结

在本文中，我们了解了为什么使用Mockito框架时会遇到模糊方法调用错误。此外，我们还展示了解决问题的方法。

在现实项目中，当我们使用带有大量参数的重载方法时，最有可能出现此类问题，并且我们决定使用约束较少的isNull()或any() ArgumentMatcher，因为某些参数的值与我们的测试无关。在简单的情况下，大多数现代IDE甚至可以在我们需要运行测试之前指出问题。