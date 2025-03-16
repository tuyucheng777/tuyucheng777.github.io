---
layout: post
title:  在Java中调用私有方法
category: java-reflect
copyright: java-reflect
excerpt: Java反射
---

## 1. 概述

虽然在Java中将方法设为私有，可以防止从所属类的外部调用它们，但我们可能仍需要出于某种原因调用它们。

为了实现这一点，我们需要绕过Java的访问控制。这可能有助于我们进入库的某个角落，或者让我们测试一些通常应该保持私密的代码。

在本简短教程中，我们将了解如何验证方法的功能，无论其可见性如何。我们将考虑两种不同的方法：[Java反射API](https://www.baeldung.com/java-reflection)和Spring的[ReflectionTestUtils](https://www.baeldung.com/spring-reflection-test-utils)。

## 2. 无法控制的可见性

在我们的示例中，我们使用一个对long数组进行操作的实用程序类LongArrayUtil，我们的类有两个indexOf方法：

```java
public static int indexOf(long[] array, long target) {
    return indexOf(array, target, 0, array.length);
}

private static int indexOf(long[] array, long target, int start, int end) {
    for (int i = start; i < end; i++) {
        if (array[i] == target) {
            return i;
        }
    }
    return -1;
}
```

假设这些方法的可见性无法改变，但我们想要调用私有的indexOf方法。

## 3. Java反射API

### 3.1 通过反射查找方法

虽然编译器阻止我们调用类中不可见的函数，但我们可以通过反射来调用函数。首先，我们需要访问描述我们要调用的函数的Method对象：

```java
Method indexOfMethod = LongArrayUtil.class.getDeclaredMethod("indexOf", long[].class, long.class, int.class, int.class);
```

我们必须使用getDeclaredMethod来访问非私有方法，我们在具有该函数的类型(在本例中为LongArrayUtil)上调用它，并传入参数的类型以识别正确的方法。

如果该方法不存在，函数可能会失败并抛出异常。

### 3.2 允许访问方法

现在我们需要暂时提升该方法的可见性：

```java
indexOfMethod.setAccessible(true);
```

此更改将持续到JVM停止或可访问属性被设置回false。

### 3.3 使用反射调用方法

最后，我们调用[Method对象上的invoke](https://www.baeldung.com/java-method-reflection)：

```java
int value = (int) indexOfMethod.invoke(null, someLongArray, 2L, 0, someLongArray.length);
```

现在我们已经成功访问了一个私有方法。

要调用的第一个参数是目标对象，其余参数需要与我们方法的签名匹配。在本例中，我们的方法是静态的，因此目标对象为null。对于调用实例方法，我们将传递要调用其方法的对象。

我们还应该注意，invoke返回Object，对于void函数来说，它是null，并且需要转换为正确的类型才能使用它。

## 4. Spring ReflectionTestUtils

访问类的内部是测试中常见的问题，Spring的测试库提供了一些快捷方式来帮助单元测试访问类。这通常可以解决单元测试特有的问题，即测试需要访问Spring可能在运行时实例化的私有字段。

首先，我们需要在pom.xml中添加[spring-test依赖项](https://mvnrepository.com/artifact/org.springframework/spring-test)：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>5.3.4</version>
    <scope>test</scope>
</dependency>
```

现在我们可以使用ReflectionTestUtils中的invokeMethod函数，它使用与上面相同的算法，并节省我们编写的代码：

```java
int value = ReflectionTestUtils.invokeMethod(LongArrayUtil.class, "indexOf", someLongArray, 1L, 1, someLongArray.length);
```

因为这是一个测试库，我们不希望在测试代码之外使用它。

## 5. 注意事项

使用反射来绕过方法可见性会带来一些风险，甚至可能无法实现，我们应该考虑：

- Java安全管理器是否允许在运行时进行此操作
- 我们正在调用的函数，在没有编译时检查的情况下，是否会继续存在以供我们将来调用
- 重构我们自己的代码，让事情变得更加清晰易懂

## 6. 总结

在本文中，我们研究了如何使用Java反射API和Spring的ReflectionTestUtils访问私有方法。