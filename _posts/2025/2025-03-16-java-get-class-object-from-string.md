---
layout: post
title:  在Java中从字符串获取Class类型
category: java-reflect
copyright: java-reflect
excerpt: Java反射
---

## 1. 概述

[Class](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Class.html)类在[Java反射](https://www.baeldung.com/java-reflection)中起着重要作用，因为它是所有反射操作的入口点。

在本快速教程中，我们将探讨如何从字符串中的类名获取Class对象。

## 2. 问题介绍

首先我们来创建一个简单的类作为例子：

```java
package cn.tuyucheng.taketoday.getclassfromstr;

public class MyNiceClass {
    public String greeting(){
        return "Hi there, I wish you all the best!";
    }
}
```

如上面的代码所示，**MyNiceClass类是在包cn.tuyucheng.taketoday.getclassfromstr中创建的**。此外，该类只有一个方法greeting()，它返回一个String。

我们的目标是根据类名获取MyNiceClass类的Class对象。此外，我们还想根据Class对象创建一个MyNiceClass实例，以验证该Class对象是否是我们要找的对象。

为简单起见，我们将使用单元测试断言来验证我们的解决方案是否按预期工作。

接下来让我们看看它的实际效果。

## 3. 使用forName()方法获取类对象

Class类提供了[forName()](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Class.html#forName(java.lang.String))方法，可以根据类名作为字符串获取Class对象，接下来，我们看看如何调用该方法获取MyNiceClass的Class对象：

```java
Class cls = Class.forName("cn.tuyucheng.taketoday.getclassfromstr.MyNiceClass");
assertNotNull(cls);
```

接下来，让我们从Class对象cls创建一个MyNiceClass实例。如果我们的Java版本早于9，我们可以使用cls.newInstance()方法获取一个实例。但是，**[自Java 9以来](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Class.html#newInstance())，此方法已被弃用**，对于较新的Java版本，我们可以使用cls.getDeclaredConstructor().newInstance()从Class对象获取一个新实例：

```java
MyNiceClass myNiceObject = (MyNiceClass) cls.getDeclaredConstructor().newInstance();
assertNotNull(myNiceObject);
assertEquals("Hi there, I wish you all the best!", myNiceObject.greeting());
```

测试运行后通过。因此，我们根据类名得到了所需的Class对象。

值得一提的是，**要获取Class对象，我们必须将限定的类名而不是简单名称传递给forName()方法**。例如，我们应该将字符串“cn.tuyucheng.taketoday.getclassfromstr.MyNiceClass”传递给forName()方法。否则，forName()方法会抛出ClassNotFoundException：

```java
assertThrows(ClassNotFoundException.class, () -> Class.forName("MyNiceClass"));
```

## 4. 关于异常处理的一些说明

我们已经了解了如何从类名获取MyNiceClass的Class对象。为简单起见，我们在测试中省略了[异常处理](https://www.baeldung.com/java-exceptions)。现在，让我们看看在使用Class.forName()和cls.getDeclaredConstructor().newInstance()方法时应该处理哪些异常。

首先，Class.forName()会抛出ClassNotFoundException，我们在将MyNiceClass的简单名称传递给它时提到过它。**ClassNotFoundException是一个[受检异常](https://www.baeldung.com/java-checked-unchecked-exceptions#checked)**，因此，我们必须在调用Class.forName()方法时处理它。

接下来我们看一下cls.getDeclaredConstructor()和newInstance()，**getDeclaredConstructor()方法会抛出NoSuchMethodException异常。此外，newInstance()还会抛出InstantiationException、IllegalAccessException、IllegalArgumentException和InvocationTargetException异常**。这五个异常都是受检异常；所以，如果要使用这两个方法，就需要对它们进行处理。

值得一提的是，本节中我们讨论的所有异常都是ReflectiveOperationException的子类型，也就是说，**如果我们不想单独处理这些异常，我们可以处理ReflectiveOperationException**，例如：

```java
void someNiceMethod() throws ReflectiveOperationException {
    Class cls = Class.forName("cn.tuyucheng.taketoday.getclassfromstr.MyNiceClass");
    MyNiceClass myNiceObject = (MyNiceClass) cls.getDeclaredConstructor().newInstance();
    // ...
}
```

或者，我们可以使用try-catch块：

```java
try {
    Class cls = Class.forName("cn.tuyucheng.taketoday.getclassfromstr.MyNiceClass");
    MyNiceClass myNiceObject = (MyNiceClass) cls.getDeclaredConstructor().newInstance();
    // ...
} catch (ReflectiveOperationException exception) {
    // handle the exception
}
```

## 5. 总结

在这篇简短的文章中，我们学习了如何使用Class.forName()方法从给定的类名字符串获取Class对象。需要注意的是，我们应该将限定名称传递给Class.forName()方法。