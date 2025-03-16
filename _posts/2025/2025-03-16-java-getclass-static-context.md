---
layout: post
title:  从静态上下文调用getClass()
category: java-reflect
copyright: java-reflect
excerpt: Java反射
---

## 1. 概述

在Java中，了解如何在[静态](https://www.baeldung.com/java-static/)和非静态上下文中调用方法至关重要，尤其是在使用诸如[getClass()](https://www.baeldung.com/java-getclass-vs-class/)之类的方法时。

我们可能遇到的一个常见问题是尝试从静态上下文调用getClass()方法，这将导致编译错误。

在本教程中，让我们探讨为什么会发生这种情况以及如何正确处理它。

## 2. 问题介绍

**getClass()方法继承自Object类，并返回调用它的对象的运行时[Class](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Class.html)**。当我们使用getClass()时，Java会为我们提供一个表示对象运行时类型的Class对象实例。

接下来我们来看一个例子：

```java
class Player {
 
    private String name;
    private int age;
 
    public Player(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public Class<?> currentClass() {
        return getClass();
    }
}
```

在这个例子中，Player是一个非常简单的类。currentClass()方法是一个实例方法，它允许我们从Player实例中获取Class<Player\>对象：

```java
Player kai = new Player("Kai", 25);
assertSame(Player.class, kai.currentClass());
```

有时，我们想从静态方法中获取当前Class对象。因此，我们可能会想出这种方法：

```java
class Player {
    // ... unrelated code omitted
    public static Class<?> getClassInStatic() {
        return getClass();
    }
}
```

但是，此代码无法编译：

```text
java: non-static method getClass() cannot be referenced from a static context
```

接下来，让我们了解为什么会发生这种情况，并探索解决此问题的不同方法。

## 3. 为什么我们不能在静态上下文中调用getClass()？

为了弄清楚这个问题的原因，让我们快速了解Java中的静态和非静态上下文。

**静态方法或变量属于类，而不是任何特定的类实例**，这意味着无需创建类的实例即可访问静态成员。

另一方面，**非静态成员(例如实例方法和变量)与类的单个实例相关联**。如果不创建该类的对象，我们就无法访问它们。

现在，让我们检查一下编译器错误发生的原因。

首先，**getClass()是一个实例方法**：

```java
public class Object {
    // ... unrelated code omitted
    public final native Class<?> getClass();
}
```

我们的getClassInStatic()是一个静态方法，我们在没有Player实例的情况下调用它。但是，在这个方法中，我们调用了getClass()。编译器会引发错误，因为**getClass()需要类的实例来确定运行时类型，但静态方法没有任何与之关联的实例**。

接下来我们看看如何解决这个问题。

## 4. 使用类字面量

在静态上下文中检索当前Class对象的最直接方法是使用类字面量(ClassName.class)：

```java
class Player {
    // ... unrelated code omitted
    public static Class<?> currentClassByClassLiteral() {
        return Player.class;
    }
}
```

我们可以看到，在currentClassByClassLiteral()方法中，**我们使用类字面量Player.class返回Player类的Class对象**。 

由于currentClassByClassLiteral()是静态方法，因此它不依赖于实例，我们可以从类中调用它：

```java
assertSame(Player.class, Player.currentClassByClassLiteral());
```

如测试所示，使用类字面量在静态上下文中获取类类型非常简单。但是，我们必须在编译时知道类名。**如果我们需要在运行时动态确定类类型，这将无济于事**。

接下来我们看看如何动态获取静态上下文中的Class对象。

## 5. 使用MethodHandles

[MethodHandle](https://www.baeldung.com/java-method-handles)是在Java 7中引入的，主要用于方法句柄和动态调用的上下文中。

[MethodHandles.Lookup](https://www.baeldung.com/java-method-handles#methodhandle-lookup)类提供各种[类似反射](https://www.baeldung.com/java-method-handles#methodhandles-reflection)的功能，**lookupClass()方法返回执行了lookup()方法的Class对象**。

接下来，让我们在Player类中创建一个静态方法，以使用这种方法返回Class对象：

```java
class Player {
    // ... unrelated codes omitted
    public static Class<?> currentClassByMethodHandles() {
        return MethodHandles.lookup().lookupClass();
    }
}
```

当我们调用此方法时，我们得到预期的Player.class对象：

```java
assertSame(Player.class, Player.currentClassByMethodHandles());
```

lookupClass()方法动态返回执行lookup()的调用类，换句话说，**lookupClass()方法返回创建MethodHandles.Lookup实例的类**，这在编写需要在运行时反射检查当前类的动态代码时非常有用。

此外，值得注意的是，由于它涉及动态查找和类似反射的机制，所以**它通常比类字面量方法慢**。

## 6. 总结

从静态上下文调用getClass()会导致Java中的编译错误，在本文中，我们讨论了此错误的根本原因。

此外，我们探索了两种解决方案来避免此问题并从静态上下文中获取预期的Class对象。使用类字面量既简单又高效，如果我们在编译时拥有类信息，这是最佳选择。但是，如果我们需要在运行时处理动态调用，MethodHandles方法是一种有用的替代方法。