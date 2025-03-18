---
layout: post
title:  Mockito Core和Mockito Inline之间的区别
category: mock
copyright: mock
excerpt: Mockito
---

## 1. 概述

Mockito是用Java创建Mock对象的最流行的框架之一，它提供Mockito Core和Mockito Inline作为具有不同特性和用例的两个主要单元测试库。

要了解有关使用Mockito进行测试的更多信息，请查看我们全面的[Mockito系列](https://www.baeldung.com/tag/mockito/)。

## 2. Mockito Core

Mockito的基础库之一是Mockito Core，它提供了创建Mock、存根和间谍的基本功能。这个库足以满足大多数常见用例，但也有一些限制，特别是在处理最终类和静态方法时。

[此处](https://www.baeldung.com/mockito-mock-methods)查看Mockito Core的Mock示例。

## 3. Mockito Inline

Mockito Inline是Mockito Core的扩展，包括用于Mock final类、final字段、静态方法和构造函数的附加功能。当你需要Mock或存根这些类型的方法或类时，此库非常有用。在最新版本的Mockito Core中，自[Mockito Core 版本5.0.0](https://github.com/mockito/mockito/releases/tag/v5.0.0)以来，Mockito Inline成为默认的Mock生成器。

现在，让我们在代码上尝试一下。首先，我们需要在pom.xml依赖项中添加[mockito-core](https://mvnrepository.com/artifact/org.mockito/mockito-core/5.11.0)：

```xml
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>5.11.0</version>
    <scope>test</scope>
</dependency>
```

### 3.1 Mock Final类

让我们尝试Mock final类，我们需要创建一个名为FinalClass的新类：

```java
public final class FinalClass {
    public String greet() {
        return "Hello, World!";
    }
}
```

让我们看一下Mock最终类的这些代码实现：

```java
@Test
void testFinalClassMock() {
    FinalClass finalClass = mock(FinalClass.class);
    when(finalClass.greet()).thenReturn("Mocked Greeting");

    assertEquals("Mocked Greeting", finalClass.greet());
}
```

**在上面的代码示例中，我们Mock了finalClass，然后我们对名为greet()的方法进行存根，以返回字符串值“Mocked Greeting”，而不是原始值“Hello, World!”**。

### 3.2 Mock final字段

让我们尝试Mock final字段，我们需要创建一个名为ClassWithFinalField的新类：

```java
public class ClassWithFinalField {
    public final String finalField = "Original Value";

    public String getFinalField() {
        return finalField;
    }
}
```

让我们看一下Mock final字段的这些代码实现：

```java
@Test
void testFinalFieldMock() {
    ClassWithFinalField instance = mock(ClassWithFinalField.class);
    when(instance.getFinalField()).thenReturn("Mocked Value");

    assertEquals("Mocked Value", instance.getFinalField());
}
```

在上面的代码示例中，**我们Mock我们的实例，然后我们存根方法名getFinalField()以返回“Mocked Value”的字符串值而不是原始值“Original Value”**。 

### 3.3 Mock静态方法

让我们尝试Mock静态方法，我们需要创建一个名为ClassWithStaticMethod的新类：

```java
public class ClassWithStaticMethod {
    public static String staticMethod() {
        return "Original Static Value";
    }
}
```

我们来看看Mock静态方法的这些代码实现：

```java
@Test
void testStaticMethodMock() {
    try (MockedStatic<ClassWithStaticMethod> mocked = mockStatic(ClassWithStaticMethod.class)) {
        mocked.when(ClassWithStaticMethod::staticMethod).thenReturn("Mocked Static Value");

        assertEquals("Mocked Static Value", ClassWithStaticMethod.staticMethod());
    }
}
```

在上面的代码示例中，**我们Mock类名ClassWithStaticMethod，然后我们对方法名staticMethod()进行存根处理，以返回“Mocked Static Value”的字符串值，而不是原始值“Original Static Value”**。

### 3.4 Mock构造函数

让我们尝试Mock构造函数，我们需要创建一个名为ClassWithConstructor的新类：

```java
public class ClassWithConstructor {
    private String name;

    public ClassWithConstructor(String name) {
        this.name = name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return this.name;
    }
}
```

让我们看一下Mock构造函数的这些代码实现：

```java
@Test
void testConstructorMock() {
    try (MockedConstruction<ClassWithConstructor> mocked = mockConstruction(ClassWithConstructor.class,
            (mock, context) -> when(mock.getName()).thenReturn("Mocked Name"))) {

        ClassWithConstructor myClass = new ClassWithConstructor("test");
        assertEquals("Mocked Name", myClass.getName());
    }
}
```

在上面的代码示例中，**我们Mock类名ClassWithConstructor，然后Mock名为name的字段，使其具有字符串值“Mocked Name”，而不是构造函数中设置的值“test”**。

## 4. 总结

让我们总结一下迄今为止所学到的知识：

| 要Mock的元素 |                  Mockito函数                   |
|:--------:|:--------------------------------------------:|
|  Final类  |            mock(FinalClass.class)            |
| Final字段  |       mock(ClassWithFinalField.class)        |
|   静态方法   |   mockStatic(ClassWithStaticMethod.class)    |
|   构造函数   | mockConstruction(ClassWithConstructor.class) |

 

## 5. 结论

在本教程中，我们比较了Mockito Core和Mockito Inline之间的差异。简而言之，Mockito Core可以Mock、存根和监视Java代码中的常见情况，但不能执行与final类、final方法、静态方法和构造函数相关的任何操作。另一方面，Mockito Inline可以执行Mockito Core无法执行的操作，即Mock和存根final类、final字段、静态方法和构造函数。