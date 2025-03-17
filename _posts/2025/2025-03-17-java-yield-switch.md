---
layout: post
title:  Java中的Yield关键字指南
category: java
copyright: java
excerpt: Java
---

## 1. 概述

我们可能经常使用Switch语句将一个值转换为另一个值，在早期版本的Java中，这要求我们将Switch嵌入到单独的函数中并对每个case使用return语句，或者要求我们从每个case中分配一个临时变量以供稍后在函数中使用。

自[Java 14](https://www.baeldung.com/java-switch#2-the-yield-keyword)以来，Switch表达式中的Yield关键字为我们提供了一种更好的方法。

## 2. yield关键字

Yield关键字让我们通过返回一个成为[Switch表达式](https://www.baeldung.com/java-switch#switch-expressions)的值的值来退出Switch表达式。

这意味着我们可以将Switch表达式的值分配给变量。

最后，通过在Switch表达式中使用yield，我们可以隐式检查是否覆盖了我们的case，这使得我们的代码更加健壮。

让我们看一些例子。

### 2.1 箭头操作符中的yield

首先，假设我们有以下枚举和Switch语句：

```java
public enum Number {
    ONE, TWO, THREE, FOUR;
}

String message;
switch (number) {
    case ONE:
        message = "Got a 1";
        break;
    case TWO:
        message = "Got a 2";
        break;
    default:
        message = "More than 2";
}
```

让我们将其转换为Switch表达式并使用yield关键字和箭头运算符：

```java
String message = switch (number) {
    case ONE -> {
        yield "Got a 1";
    }
    case TWO -> {
        yield "Got a 2";
    }
    default -> {
        yield "More than 2";
    }
};
```

Yield根据number的值设置Switch表达式的值。

### 2.2 使用冒号分隔符的yield

我们还可以使用带有冒号分隔符的yield创建一个switch表达式：

```java
String message = switch (number) {
    case ONE:
        yield "Got a 1";
    case TWO:
        yield "Got a 2";
    default:
        yield "More than 2";
};
```

此代码的行为与上一节相同，但箭头运算符更清晰，也更不容易忘记yield(或break)语句。

我们应该注意，在同一个Switch表达式中不能混合冒号和箭头分隔符。

## 3. 详尽性

使用Switch表达式和yield的另一个好处是，如果我们缺少case覆盖，我们将看到编译错误。让我们从箭头运算符Switch表达式中删除默认case来检查：

```java
String message = switch (number) {
    case ONE -> {
        yield "Got a 1";
    }
    case TWO -> {
        yield "Got a 2";
    }
};
```

上面的代码给了我们一个数字错误：“the switch expression does not cover all possible input values”。

我们可以重新添加默认情况，或者我们可以专门涵盖number的其余可能值：

```java
String message = switch (number) {
    case ONE -> {
        yield "Got a 1";
    }
    case TWO -> {
        yield "Got a 2";
    }
    case THREE, FOUR -> {
        yield "More than 2";
    }
};
```

Switch表达式迫使我们的case覆盖面变得详尽无遗。

## 4. 总结

在本文中，我们探讨了Java中的Yield关键字、其用法及其一些好处。