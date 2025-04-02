---
layout: post
title:  Java注解属性值限制
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

如今，很难想象没有注解的Java会是什么样子，注解是Java语言中的一个强大工具。

Java提供了一组[内置的注解](https://www.baeldung.com/java-default-annotations)，此外，还有大量来自不同库的注解。我们甚至可以定义和处理我们自己的注解，我们可以使用属性值调整这些注解，但是，这些属性值有局限性。特别地，**注解属性值必须是常量表达式**。

在本教程中，我们将了解造成这种限制的一些原因，并深入了解JVM以更好地解释它。我们还将查看一些涉及注解属性值的问题示例和解决方案。

## 2. Java注解属性的底层原理

让我们考虑一下Java类文件如何存储注解属性，Java有一个特殊的结构，称为[element_value](https://docs.oracle.com/javase/specs/jvms/se15/html/jvms-4.html#jvms-4.7.16.1)，此结构存储特定的注解属性。

结构element_value可以存储4种不同类型的值：

-   来自常量池的常量
-   类字面量
-   嵌套注解
-   值数组

因此，注解属性中的常量是[编译时常量](https://www.baeldung.com/java-compile-time-constants#1-compile-time-constants)。否则，编译器将不知道应该将什么值放入常量池并用作注解属性。

Java规范定义了生成[常量表达式](https://docs.oracle.com/javase/specs/jls/se8/html/jls-15.html#jls-15.28)的操作，如果我们将这些操作应用于编译时常量，我们将得到编译时常量。

假设我们有一个具有属性value的注解@Marker：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Marker {
    String value();
}
```

例如，这段代码编译没有错误：

```java
@Marker(Example.ATTRIBUTE_FOO + Example.ATTRIBUTE_BAR)
public class Example {
    static final String ATTRIBUTE_FOO = "foo";
    static final String ATTRIBUTE_BAR = "bar";

    // ...
}
```

在这里，我们将注解属性定义为两个字符串的拼接，拼接运算符产生常量表达式。

## 3. 使用静态初始化器

让我们考虑在静态块中初始化的常量：

```java
@Marker(Example.ATTRIBUTE_FOO)
public class Example {
    static final String[] ATTRIBUTES = {"foo", "Bar"};
    static final String ATTRIBUTE_FOO;

    static {
        ATTRIBUTE_FOO = ATTRIBUTES[0];
    }

    // ...
}
```

它在静态块中初始化字段并尝试将该字段用作注解属性，**这种方法会导致编译错误**。

首先，变量ATTRIBUTE_FOO具有static和final修饰符，但编译器无法计算该字段。应用程序在运行时计算它。

其次，**在JVM加载类之前，注解属性必须具有准确的值**。但是，当静态初始化程序运行时，该类已经加载。所以，这个限制是有道理的。

在字段初始化时会出现同样的错误，由于同样的原因，此代码不正确：

```java
@Marker(Example.ATTRIBUTE_FOO)
public class Example {
    static final String[] ATTRIBUTES = {"foo", "Bar"};
    static final String ATTRIBUTE_FOO = ATTRIBUTES[0];

    // ...
}
```

JVM如何初始化ATTRIBUTE_FOO？数组访问运算符ATTRIBUTES[0\]在类初始化程序中运行。所以，ATTRIBUTE_FOO是一个运行时常量，它不是在编译时定义的。

## 4. 数组常量作为注解属性

让我们考虑一个数组注解属性：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Marker {
    String[] value();
}
```

此代码将无法编译：

```java
@Marker(value = Example.ATTRIBUTES)
public class Example {
    static final String[] ATTRIBUTES = {"foo", "bar"};

    // ...
}
```

首先，虽然final修饰符保护引用不被更改，**但我们仍然可以修改数组元素**。

其次，**数组文字不能是运行时常量。JVM在静态初始化程序中设置每个元素**-这是我们在前面描述过的一个限制。

最后，类文件存储该数组中每个元素的值。因此，编译器计算属性数组的每个元素，并且发生在编译时。

因此，我们每次只能指定一个数组属性：

```java
@Marker(value = {"foo", "bar"})
public class Example {
    // ...
}
```

我们仍然可以使用常量作为数组属性的原始元素。

## 5. 标记接口中的注解：为什么不起作用？

因此，如果注解属性是数组，我们每次都必须重复它。但我们想避免这种复制粘贴，我们为什么不将注解设为@Inherited？我们可以将注解添加到[标记接口](https://www.baeldung.com/java-marker-interfaces#:~:text=A%20marker%20interface%20is%20an,also%20called%20a%20tagging%20interface.)：

```java
@Marker(value = {"foo", "bar"})
public interface MarkerInterface {
}
```

然后，我们可以让需要这个注解的类实现它：

```java
public class Example implements MarkerInterface {
    // ...
}
```

**这种方法行不通**，代码将编译成功，不会出现错误。但是，**Java不支持从接口继承注解**，即使注解本身具有@Inherited注解。因此，实现标记接口的类不会继承注解。

**造成这种情况的原因就是多重继承的问题**。事实上，如果多个接口具有相同的注解，Java就无法取其一。

因此，我们无法通过标记接口来避免这种复制粘贴。

## 6. 数组元素作为注解属性

假设我们有一个数组常量，并且我们将该常量用作注解属性：

```java
@Marker(Example.ATTRIBUTES[0])
public class Example {
    static final String[] ATTRIBUTES = {"Foo", "Bar"};
    // ...
}
```

此代码无法编译，注解参数必须是编译时常量。但是，正如我们之前所考虑的，**数组不是编译时常量**。

此外，**数组访问表达式不是常量表达式**。

如果我们有一个List而不是数组会怎么样？方法调用不属于常量表达式。因此，使用List类的get方法会导致相同的错误。

相反，我们应该显式地引用一个常量：

```java
@Marker(Example.ATTRIBUTE_FOO)
public class Example {
    static final String ATTRIBUTE_FOO = "Foo";
    static final String[] ATTRIBUTES = {ATTRIBUTE_FOO, "Bar"};
    // ...
}
```

这样，我们在字符串常量中指定了注解属性值，Java编译器就可以明确地找到该属性值。

## 7. 总结

在本文中，我们了解了注解参数的局限性。我们考虑了注解属性问题的一些示例，我们还在这些局限性的背景下讨论了JVM内部原理。

在所有示例中，我们对常量和注解使用相同的类。但是，所有这些限制都适用于常量来自另一个类的情况。