---
layout: post
title:  使用反射检查Java类是否为抽象类
category: java-reflect
copyright: java-reflect
excerpt: Java反射
---

## 1. 概述

在本快速教程中，我们将讨论如何使用[反射](https://www.baeldung.com/java-reflection)API检查Java中某个类是否是抽象的。

## 2. 类和接口示例

为了演示这一点，我们将创建一个AbstractExample类和一个InterfaceExample接口：

```java
public abstract class AbstractExample {

    public abstract LocalDate getLocalDate();

    public abstract LocalTime getLocalTime();
}

public interface InterfaceExample {
}
```

## 3. Modifier#isAbstract方法

我们可以使用反射API中的[Modifier#isAbstract](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/reflect/Modifier.html#isAbstract(int))方法检查某个类是否是抽象的：

```java
@Test
void givenAbstractClass_whenCheckModifierIsAbstract_thenTrue() throws Exception {
    Class<AbstractExample> clazz = AbstractExample.class;
 
    Assertions.assertTrue(Modifier.isAbstract(clazz.getModifiers()));
}
```

在上面的例子中，我们首先获取要测试的类的实例。**一旦我们有了类引用，我们就可以调用Modifier#isAbstract方法**。正如我们所料，如果该类是抽象的，它返回true，否则返回false。

值得一提的是，**接口类也是抽象的**。我们可以通过一个测试方法来验证这一点：

```java
@Test
void givenInterface_whenCheckModifierIsAbstract_thenTrue() {
    Class<InterfaceExample> clazz = InterfaceExample.class;
 
    Assertions.assertTrue(Modifier.isAbstract(clazz.getModifiers()));
}
```

如果我们执行上述测试方法，它就会通过。

反射API还提供了[isInterface()](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/reflect/Modifier.html#isInterface(int))方法，如果我们想检查给定类是否是抽象类但不是接口类，我们可以结合这两种方法：

```java
@Test
void givenAbstractClass_whenCheckIsAbstractClass_thenTrue() {
    Class<AbstractExample> clazz = AbstractExample.class;
    int mod = clazz.getModifiers();
 
    Assertions.assertTrue(Modifier.isAbstract(mod) && !Modifier.isInterface(mod));
}
```

我们还来验证具体类是否返回适当的结果：

```java
@Test
void givenConcreteClass_whenCheckIsAbstractClass_thenFalse() {
    Class<Date> clazz = Date.class;
    int mod = clazz.getModifiers();
 
    Assertions.assertFalse(Modifier.isAbstract(mod) && !Modifier.isInterface(mod));
}
```

## 4. 总结

在本教程中，我们了解了如何检查一个类是否是抽象的。

 此外，我们还通过示例讨论了如何检查一个类是否是抽象类而不是接口。