---
layout: post
title:  Java中的常量：模式和反模式
category: designpattern
copyright: designpattern
excerpt: 常量
---

## 1. 简介

在本文中，我们将学习在Java中使用常量，重点关注常见模式和[反模式](https://www.baeldung.com/cs/anti-patterns)。

我们将从定义常量的一些基本约定开始，然后，我们将讨论常见的反模式，最后再看看常见的模式。

## 2. 基础知识

**常量是一种变量，其值一旦定义就不会改变**。

让我们看一下定义常量的基本知识：

```java
private static final int OUR_CONSTANT = 1;
```

我们将要讨论的一些模式将解决[public](https://www.baeldung.com/java-public-keyword)或[private](https://www.baeldung.com/java-private-keyword)[访问修饰符](https://www.baeldung.com/java-access-modifiers)的选择问题。我们将常量定义为static和[final](https://www.baeldung.com/java-final)，并赋予它们适当的类型，可以是Java原始类型、类或枚举。**名称应全部大写，单词之间用下划线分隔**，有时也称为蛇形命名法。最后，我们提供值本身。

## 3. 反模式

首先，我们先来学习一下哪些事情不该做，我们来看看在使用Java常量时可能遇到的几个常见反模式。

### 3.1 魔法数字

**[魔法数字](https://www.baeldung.com/cs/antipatterns-magic-numbers)是代码块中的数字文字**：

```java
if (number == 3.14159265359) {
    // ...
}
```

其他开发人员很难理解它们。此外，如果我们在整个代码中都使用数字，那么修改它的值会很困难，我们应该将数字定义为常量。

### 3.2 大型全局常量类

当我们启动一个项目时，创建一个名为Constants或Utils的类来定义应用程序的所有常量可能很自然。对于较小的项目来说，这样做可能没问题，但让我们考虑一下为什么这不是一个理想的解决方案。

首先，假设我们的常量类中有一百个或更多的常量，如果这个类没有人维护，无论是为了跟上文档的更新，还是为了偶尔将常量重构为逻辑分组，它的可读性都会变得非常差。我们甚至可能会得到名称略有不同的重复常量，除了最小的项目之外，这种方法很可能会在任何其他情况下带来可读性和可维护性问题。

除了维护Constants类本身的后勤工作之外，我们还通过鼓励这个全局常量类和应用程序的其他部分之间过度相互依赖而引发其他可维护性问题。

从技术角度来看，**Java编译器会将常量的值放入我们使用它们的类的引用变量中**。因此，如果我们更改常量类中的一个常量，并且只重新编译该类而不重新编译引用类，我们就会得到不一致的常量值。

### 3.3 常量接口反模式

常量接口模式是指我们定义一个接口，该接口包含某些功能的所有常量，然后让需要这些功能的类实现该接口。

让我们为计算器定义一个常量接口：

```java
public interface CalculatorConstants {
    double PI = 3.14159265359;
    double UPPER_LIMIT = 0x1.fffffffffffffP+1023;
    enum Operation {ADD, SUBTRACT, MULTIPLY, DIVIDE};
}
```

接下来，我们将实现CalculatorConstants接口：

```java
public class GeometryCalculator implements CalculatorConstants {
    public double operateOnTwoNumbers(double numberOne, double numberTwo, Operation operation) {
        // Code to do an operation
    }
}
```

反对使用常量接口的第一个理由是，它违背了接口的初衷。我们的目的是使用接口来为实现类将要提供的行为创建一个契约，当我们创建一个充满常量的接口时，我们并没有定义任何行为。

其次，使用常量接口会给我们带来由字段阴影引起的运行时问题，让我们通过在GeometryCalculator类中定义一个UPPER_LIMIT常量来看一下这种情况是如何发生的：

```java
public static final double UPPER_LIMIT = 100000000000000000000.0;
```

一旦我们在GeometryCalculator类中定义了该常量，我们就会在类的CalculatorConstants接口中隐藏该值。这样一来，我们可能会得到意想不到的结果。

反对这种反模式的另一个理由是它会导致命名空间污染，我们的CalculatorConstants现在将位于任何实现该接口的类及其子类的命名空间中。

## 4. 模式

之前，我们了解了定义常量的正确形式。接下来，我们来看看在应用程序中定义常量的其他一些良好做法。

### 4.1 一般良好做法

如果常量在逻辑上与某个类相关，我们可以直接在类中定义它们。如果我们将一组常量视为枚举类型的成员，则可以使用枚举来定义它们。

让我们在Calculator类中定义一些常量：

```java
public class Calculator {
    public static final double PI = 3.14159265359;
    private static final double UPPER_LIMIT = 0x1.fffffffffffffP+1023;
    public enum Operation {
        ADD,
        SUBTRACT,
        DIVIDE,
        MULTIPLY
    }

    public double operateOnTwoNumbers(double numberOne, double numberTwo, Operation operation) {
        if (numberOne > UPPER_LIMIT) {
            throw new IllegalArgumentException("'numberOne' is too large");
        }
        if (numberTwo > UPPER_LIMIT) {
            throw new IllegalArgumentException("'numberTwo' is too large");
        }
        double answer = 0;

        switch(operation) {
            case ADD:
                answer = numberOne + numberTwo;
                break;
            case SUBTRACT:
                answer = numberOne - numberTwo;
                break;
            case DIVIDE:
                answer = numberOne / numberTwo;
                break;
            case MULTIPLY:
                answer = numberOne * numberTwo;
                break;
        }

        return answer;
    }
}
```

在我们的示例中，为UPPER_LIMIT定义了一个常量，我们只打算在Calculator类中使用，因此我们将其设置为private。我们希望其他类能够使用PI和Operation枚举，因此我们将它们设置为public。

让我们考虑一下在Operation中使用枚举的一些优点；第一个优点是它限制了可能的值，假设我们的方法接收一个字符串作为操作值，并期望提供4个常量字符串中的一个，我们可以很容易地预见到这样一种情况：调用该方法的开发人员会发送他们自己的字符串值。**使用枚举，值仅限于我们定义的值**。我们还可以看到，枚举特别适合在[switch](https://www.baeldung.com/java-switch)语句中使用。

### 4.2 常量类

既然我们已经了解了一些通用的良好实践，让我们考虑一下常量类可能是一个好主意的情况。假设我们的应用程序包含一个需要进行各种数学计算的类包，在这种情况下，我们最好在该包中定义一个常量类，用于存放我们将在计算类中使用的常量。

让我们创建一个MathConstants类：

```java
public final class MathConstants {
    public static final double PI = 3.14159265359;
    static final double GOLDEN_RATIO = 1.6180;
    static final double GRAVITATIONAL_ACCELERATION = 9.8;
    static final double EULERS_NUMBER = 2.7182818284590452353602874713527;
    
    public enum Operation {
        ADD,
        SUBTRACT,
        DIVIDE,
        MULTIPLY
    }
    
    private MathConstants() {
    }
}
```

我们首先要注意的是，**我们的类是final的，以防止它被扩展**。此外，我们定义了一个私有构造函数，因此它不能被实例化。最后，我们可以看到我们已经应用了在文章前面讨论过的其他良好做法。我们的常量PI是公共的，因为我们预计需要在包之外访问它。我们将其他常量保留为包私有，因此我们可以在包内访问它们。我们将所有常量都设为static和final，并用蛇形命名法命名它们。操作是一组特定的值，因此我们使用枚举来定义它们。

我们可以看到，我们的特定包级常量类不同于大型全局常量类，因为它本地化到我们的包并包含与该包的类相关的常量。

## 5. 总结

在本文中，我们探讨了Java中使用常量时常见的一些模式和反模式的优缺点。首先，我们介绍了一些基本的格式规则，然后介绍了反模式。在了解了几种常见的反模式之后，我们又研究了常量中常见的一些模式。