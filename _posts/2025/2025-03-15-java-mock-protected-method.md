---
layout: post
title:  Java中Mock受保护的方法
category: mock
copyright: mock
excerpt: Mockito
---

## 1. 概述

**在Java中Mock受保护方法类似于[Mock公共方法](https://www.baeldung.com/mockito-mock-methods)，但有一个注意事项：此方法在测试类中的可见性**。我们可以访问来自同一包类和扩展A的类A的受保护方法。因此，如果我们尝试测试来自不同包的类A，就会遇到问题。

在本教程中，我们将讨论Mock被测类的受保护方法的情况。我们将演示可以访问和不能访问该方法的两种情况，我们将使用[Mockito Spy而不是Mock](https://www.baeldung.com/mockito-spy)来实现这一点，因为我们只想对被测类的一些行为进行存根。

## 2. Mock受保护的方法

当我们可以访问受保护方法时，使用Mockito Mock受保护方法非常简单，我们可以通过两种方式访问。首先，我们将受保护的范围更改为public，或者其次，我们将测试类移动到与具有受保护方法的类相同的包中。

但有时这不是一个选择，所以另一种选择是遵循间接做法。**最常见的做法是不使用任何外部库，包括**：

- **使用[JUnit 5](https://www.baeldung.com/junit-5)和[反射](https://www.baeldung.com/java-reflection)**
- **使用扩展被测类的内部测试类**

如果我们尝试更改访问修饰符，这可能会导致不必要的行为。除非有充分的理由不这样做，否则使用最严格的访问级别是一种很好的做法。同样，如果更改测试的位置是有意义的，那么将其移动到与具有受保护方法的类相同的包中是一种简单的选择。

如果这两种方法都不适合我们的情况，那么当类中只有一个受保护方法需要存根时，使用反射的JUnit 5是一个不错的选择。**对于需要存根的具有多个受保护方法的类A，创建一个扩展A的内部类是更干净的解决方案**。

## 3. Mock可见的受保护方法

在本节中，我们将处理测试有权访问受保护方法的情况，或者我们可以进行更改以获取访问权限。如前所述，**更改可能是将访问修饰符设为public，或将测试移至与具有受保护方法的类相同的包中**。

让我们以Movies类为例，它有一个受保护的方法getTitle()来检索私有字段title的值。它还包含一个公共方法getPlaceHolder()，可供客户端使用：

```java
public class Movies {
    private final String title;

    public Movies(String title) {
        this.title = title;
    }

    public String getPlaceHolder() {
        return "Movie: " + getTitle();
    }

    protected String getTitle() {
        return title;
    }
}

```

在测试类中，首先，我们断言getPlaceholder()方法的初始值是我们期望的值。然后，我们使用Mockito Spy对受保护方法的功能进行存根，并断言getPlaceholder()返回的新值包含getTitle()的存根值：

```java
@Test
void givenProtectedMethod_whenMethodIsVisibleAndUseMockitoToStub_thenResponseIsStubbed() {
    Movies matrix = Mockito.spy(new Movies("The Matrix"));
    assertThat(matrix.getPlaceHolder()).isEqualTo("Movie: The Matrix");

    doReturn("something else").when(matrix).getTitle();

    assertThat(matrix.getTitle()).isEqualTo("something else");
    assertThat(matrix.getPlaceHolder()).isEqualTo("Movie: something else");
}
```

## 4. Mock不可见的受保护方法

接下来，让我们看看当我们无法访问受保护的方法时，如何使用Mockito对其进行Mock。我们要处理的用例是当测试类与我们想要存根的类位于不同的包中时。在这种情况下，我们有两个选择：

- JUnit 5和反射
- 使用受保护方法扩展类的内部类

### 4.1 使用JUnit和反射

**JUnit 5提供了一个类[ReflectionSupport](https://junit.org/junit5/docs/5.8.0/api/org.junit.platform.commons/org/junit/platform/commons/support/ReflectionSupport.html)，用于处理测试的常见反射用例**，例如查找/调用方法等。让我们看看它如何与我们之前的代码一起使用：

```java
@Test
void givenProtectedMethod_whenMethodIsVisibleAndUseMockitoToStub_thenResponseIsStubbed() throws NoSuchMethodException {
    Movies matrix = Mockito.spy(new Movies("The Matrix"));
    assertThat(matrix.getPlaceHolder()).isEqualTo("Movie: The Matrix");

    ReflectionSupport.invokeMethod(
            Movies.class.getDeclaredMethod("getTitle"),
            doReturn("something else").when(matrix));

    assertThat(matrix.getPlaceHolder()).isEqualTo("Movie: something else");
}
```

在这里，我们使用ReflectionSupport的invokeMethod()，在调用时将受保护方法的值设置为存根的Movie对象。

### 4.2 使用内部类

**我们可以通过创建一个内部类来解决可见性问题，该内部类扩展了被测类并使受保护的方法可见**。如果我们需要在不同的测试类中Mock同一个类的受保护方法，则内部类可以是一个独立的类。

在我们的例子中，将扩展Movies的MoviesWrapper类(来自我们之前的代码)作为测试类的内部类是有意义的：

```java
private static class MoviesWrapper extends Movies {
    public MoviesWrapper(String title) {
        super(title);
    }

    @Override
    protected String getTitle() {
        return super.getTitle();
    }
}
```

这样，我们就可以通过MoviesWrapper类访问Movies的getTitle()了。如果我们不使用内部类而是使用独立类，则方法访问修饰符可能需要变为public。

然后，测试使用MoviesWrapper类作为被测类。这样，我们就可以访问getTitle()，并且可以轻松地使用Mockito Spy对其进行存根：

```java
@Test
void givenProtectedMethod_whenMethodNotVisibleAndUseInnerTestClass_thenResponseIsStubbed() {
    MoviesWrapper matrix = Mockito.spy(new MoviesWrapper("The Matrix"));
    assertThat(matrix.getPlaceHolder()).isEqualTo("Movie: The Matrix");

    doReturn("something else").when(matrix).getTitle();

    assertThat(matrix.getPlaceHolder()).isEqualTo("Movie: something else");
}
```

## 5. 总结

在本文中，我们讨论了在Java中Mock受保护方法时可见性的困难，并演示了可能的解决方案。我们可能遇到的每个用例都有不同的选项，根据示例，我们应该每次都能选择正确的选项。