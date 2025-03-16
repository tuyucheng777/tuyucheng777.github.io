---
layout: post
title:  使用Java中的反射检查方法是否为静态
category: java-reflect
copyright: java-reflect
excerpt: Java反射
---

## 1. 概述

在本快速教程中，我们将讨论如何使用[反射](https://www.baeldung.com/java-reflection)API检查Java中的方法是否为[静态](https://www.baeldung.com/java-static#the-static-methods-or-class-methods)。

## 2. 示例

为了演示这一点，我们将创建StaticUtility类，它有一些静态方法：

```java
public class StaticUtility {

    public static String getAuthorName() {
        return "Umang Budhwar";
    }

    public static LocalDate getLocalDate() {
        return LocalDate.now();
    }

    public static LocalTime getLocalTime() {
        return LocalTime.now();
    }
}
```

## 3. 检查方法是否为静态

**我们可以使用[Modifier](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/reflect/Modifier.html).isStatic方法检查方法是否是静态的**：

```java
@Test
void whenCheckStaticMethod_ThenSuccess() throws Exception {
    Method method = StaticUtility.class.getMethod("getAuthorName", null);
    Assertions.assertTrue(Modifier.isStatic(method.getModifiers()));
}
```

在上面的例子中，我们首先使用Class.getMethod方法获取要测试的方法的实例。一旦我们获得了方法引用，我们所需要做的就是调用Modifier.isStatic方法。

## 4. 获取类的所有静态方法

现在我们已经知道如何检查一个方法是否是静态的，我们可以轻松列出一个类的所有静态方法：

```java
@Test
void whenCheckAllStaticMethods_thenSuccess() {
    List<Method> methodList = Arrays.asList(StaticUtility.class.getMethods())
        .stream()
        .filter(method -> Modifier.isStatic(method.getModifiers()))
        .collect(Collectors.toList());
    Assertions.assertEquals(3, methodList.size());
}
```

在上面的代码中，我们刚刚验证了StaticUtility类中的静态方法总数。

## 5. 总结

在本教程中，我们了解了如何检查方法是否为静态方法，还了解了如何获取类的所有静态方法。