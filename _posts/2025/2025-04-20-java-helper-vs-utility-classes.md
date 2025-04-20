---
layout: post
title:  Java辅助类与实用程序类
category: designpattern
copyright: designpattern
excerpt: 辅助类
---

## 1. 概述

在本教程中，我们将探讨Java辅助类和实用程序类之间的区别，首先，我们将了解它们的含义以及如何创建它们。

## 2. Java帮助类

辅助类提供Java程序整体运行所需的功能，**辅助类包含其他类用来执行重复性任务的方法，而这些任务并非应用程序的核心目的**。

顾名思义，**它们通过提供一些补充其他类所提供的服务的功能来帮助其他类**。

它们包含实现日常和重复任务的方法，使整个代码库模块化并可在多个类中重复使用。

辅助类可以被实例化，并且可能包含[实例变量](https://www.baeldung.com/java-static-instance-initializer-blocks)、实例和[静态](https://www.baeldung.com/java-static)方法。

我们的应用程序中可以存在多个辅助类实例，当不同的类具有共同的功能时，我们可以将这些功能组合在一起，形成一个辅助类，以便应用程序中的某些类可以访问该类。

### 2.1 如何创建Java辅助类

我们将创建一个示例帮助类来进一步理解这个概念。

要创建辅助类，我们在类名中使用默认访问修饰符，默认访问修饰符确保只有同一包内的类才能访问此类、其方法和变量：

```java
class MyHelperClass {

    public double discount;

    public MyHelperClass(double discount) {
        if (discount > 0 && discount < 1) {
            this.discount = discount;
        }
    }

    public double discountedPrice(double price) {
        return price - (price * discount);
    }

    public static int getMaxNumber(int[] numbers) {
        if (numbers.length == 0) {
            throw new IllegalArgumentException("Ensure array is not empty");
        }
        int max = numbers[0];
        for (int i = 1; i < numbers.length; i++) {
            if (numbers[i] > max) {
                max = numbers[i];
            }
        }
        return max;
    }

    public static int getMinNumber(int[] numbers) {
        if (numbers.length == 0) {
            throw new IllegalArgumentException("Ensure array is not empty");
        }
        int min = numbers[0];
        for (int i = 1; i < numbers.length; i++) {
            if (numbers[i] < min) {
                min = numbers[i];
            }
        }
        return min;
    }
}
```

定义类之后，我们可以根据需要添加任意数量的相关实例和静态方法。

辅助类可以具有[实例方法](https://www.baeldung.com/java-class-methods-vs-instance-methods)，因为它们可以被实例化。

从上面的代码片段可以看出，MyHelperClass中有一个实例和一个静态方法：

```java
@Test
void whenCreatingHelperObject_thenHelperObjectShouldBeCreated() {
    MyHelperClass myHelperClassObject = new MyHelperClass(0.10);
    assertNotNull(myHelperClassObject);
    assertEquals(90, myHelperClassObject.discountedPrice(100.00));

    int[] numberArray = {15, 23, 66, 3, 51, 79};
    assertEquals( 79, MyHelperClass.getMaxNumber(numberArray));
    assertEquals(3, MyHelperClass.getMinNumber(numberArray));
}
```

从测试中我们可以看到创建了一个辅助对象，辅助类中的静态方法可以通过类名访问。

## 3. Java实用程序类

**Java中的实用程序类是提供可在整个应用程序中访问的静态方法的类**，实用程序类中的静态方法用于执行应用程序中的常见例程。

**工具类无法实例化**，如果没有静态变量，它们有时是无状态的。我们将工具类声明为[final](https://www.baeldung.com/java-final)，并且其所有方法都必须是static的。

由于我们不希望工具类被实例化，因此引入了私有构造函数，**拥有私有构造函数意味着Java不会为工具类创建默认构造函数**，构造函数可以为空。

实用程序类的目的是提供在程序中执行某些功能的方法，而主类则专注于它解决的核心问题。

实用程序的方法可以通过类名访问。它使我们的代码在保持模块化的同时，使用起来更加灵活。

Java具有实用程序类，例如java.util.Arrays、java.lang.Math、java.util.Scanner、java.util.Collections等。

### 3.1 如何创建Java实用程序类

创建实用程序类与创建辅助类并没有太大区别，创建实用程序类时，有些操作略有不同。

要创建一个实用程序类，**我们使用public访问修饰符，并将该类声明为final**。创建实用程序类时使用的final关键字意味着该类将保持不变，它不能被继承或实例化。

让我们创建一个名为MyUtilityClass的实用程序类：

```java
public final class MyUtilityClass {

    private MyUtilityClass(){}

    public static String returnUpperCase(String stringInput) {
        return stringInput.toUpperCase();
    }

    public static String returnLowerCase(String stringInput) {
        return stringInput.toLowerCase();
    }

    public static String[] splitStringInput(String stringInput, String delimiter) {
        return stringInput.split(delimiter);
    }
}
```

要遵守的另一条规则是，实用程序类的所有方法都是静态的，并带有公共访问修饰符。

由于实用程序类中只有静态方法，因此只能通过类名访问这些方法：

```java
@Test
void whenUsingUtilityMethods_thenAccessMethodsViaClassName(){
    assertEquals(MyUtilityClass.returnUpperCase("iniubong"), "INIUBONG");
    assertEquals(MyUtilityClass.returnLowerCase("AcCrA"), "accra");
}
```

实用程序类中的returnUpperCase()和returnLowerCase()方法只能由其调用者通过类名访问，如上所示。

## 4. Java辅助类与实用程序类

Java中的辅助类和实用程序类通常具有相同的用途，**实用程序类是通用的辅助类，开发人员有时会互换使用这两个术语**。

这是因为它们通过提供处理某些非应用程序核心功能的任务的方法来补充其他类。

尽管它们非常相似，但它们之间存在一些细微的差别：

- 辅助类可以被实例化，而实用程序类不能被实例化，因为它们具有私有构造函数。
- 辅助类可以具有实例变量，也可以具有实例和静态方法。
- 实用程序类只有静态变量和方法。
- 实用程序类通常在我们的应用程序中具有全局范围，而辅助类始终具有包范围。

## 5. 总结

在本文中，我们研究了Java中的辅助类和实用程序类，本质上辅助类和实用程序类在应用程序中的使用方式非常相似。

我们详细介绍了如何创建辅助类或实用程序类。

在用Java创建健壮的应用程序时，我们应该始终记住将执行重复任务的相似但独立的方法分组到辅助类或实用程序类中。有时，软件工程师和开发人员创建实用程序类或辅助类会违背Java面向对象的编程范式。然而，它们使我们的代码库高度模块化且可重用。