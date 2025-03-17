---
layout: post
title:  如何在Java中将方法作为参数传递
category: java
copyright: java
excerpt: Java Function
---

## 1. 简介

在Java中，我们可以使用函数式编程概念将一个方法作为参数传递给另一个方法，具体来说是使用Lambda表达式、[方法引用](https://www.baeldung.com/java-method-references)和[函数接口](https://www.baeldung.com/java-8-functional-interfaces)。在本教程中，我们将探讨将方法作为参数传递的几种方法。

## 2. 使用接口和匿名内部类

在Java 8之前，我们依赖接口和匿名内部类将方法作为参数传递。下面是一个示例来说明这种方法：

```java
interface Operation {
    int execute(int a, int b);
}
```

我们定义一个名为Operation的接口，该接口只有一个抽象方法execute()，此方法接收两个整数作为参数并返回一个整数，**任何实现此接口的类都必须提供execute()方法的实现**。

接下来，我们创建一个名为performOperation()的方法来接收两个int参数和一个Operation实例：

```java
int performOperation(int a, int b, Operation operation) {
    return operation.execute(a, b);
}
```

在这个方法中，我们调用operation.execute(a, b)，这行代码调用作为参数传递的Operation实例的execute()方法。

然后我们调用performOperation()方法并传递三个参数：

```java
int actualResult = performOperation(5, 3, new Operation() {
    @Override
    public int execute(int a, int b) {
        return a + b;
    }
});
```

在performOperation()方法中，使用[匿名内部类](https://www.baeldung.com/java-anonymous-classes)创建Operation接口的新实例。**此类没有名称，但它动态地提供了对execute()方法的实现**。

在匿名内部类中，重写了execute()方法。**在本例中，它只是将两个整数a和b相加，然后返回两个整数的和**。

最后，让我们使用断言验证我们的实现，以确保结果符合预期：

```java
assertEquals(8, actualResult);
```

## 3. 使用Lambda表达式

在Java 8中，Lambda表达式使将方法作为参数传递更加优雅和简洁。以下是我们如何使用Lambda实现相同的功能：

```java
@FunctionalInterface
interface Operation {
    int execute(int a, int b);
}
```

**我们定义一个接口Operation，并使用@FunctionalInterface注解来指示该接口只有一个抽象方法**。

接下来，我们调用performOperation()方法并传入两个int参数和一个Operation接口的实例：

```java
int actualResult = performOperation(5, 3, (a, b) -> a + b);
```

对于第三个参数，我们传递的不是匿名内部类，而是Lambda表达式(a, b)-> a + b，它表示Operation函数接口的一个实例。

我们应该得到相同的结果：

```java
assertEquals(8, actualResult);
```

与匿名内部类相比，使用Lambda表达式简化了语法并使代码更具可读性。

## 4. 使用方法引用

Java中的方法引用提供了一种将方法作为参数传递的简化方法，**它们充当调用特定方法的Lambda表达式的简写**，让我们看看如何使用方法引用实现相同的功能。

我们定义一个名为add()的方法，它接收两个整数a和b作为参数并返回它们的和：

```java
int add(int a, int b) {
    return a + b;
}
```

此方法只是将两个整数相加并返回结果。**然后，使用语法object::methodName或ClassName::methodName传递该方法作为引用**：

```java
int actualResult = performOperation(5, 3, FunctionParameter::add);
assertEquals(8, actualResult);
```

此处，FunctionParameter::add指的是FunctionParameter类中的add()方法。**它允许我们将add()方法定义的行为作为参数传递给另一个方法，在本例中为performOperation()方法**。

此外，在performOperation()方法中，add()方法引用被视为Operation函数接口的一个实例，该接口具有单个抽象方法execute()。

## 5. 使用Function类

除了方法引用和Lambda表达式之外，Java 8还引入了java.util.function包，为常用操作提供了函数式接口。**其中，[BiFunction](https://www.baeldung.com/java-bifunction-interface)是一个函数式接口，表示具有两个输入参数和一个返回值的函数**。下面我们来探索如何使用BiFunction实现类似的功能。

首先，我们创建executeFunction()方法，该方法接收BiFunction<Integer, Integer, Integer\>作为第一个参数，这意味着它接收一个以两个Integer值作为输入并返回一个Integer的函数：

```java
int executeFunction(BiFunction<Integer, Integer, Integer> function, int a, int b) {
    return function.apply(a, b);
}
```

**apply()方法用于将函数应用于其两个参数**。接下来，我们可以使用Lambda表达式创建BiFunction的实例，并将其作为参数传递给executeFunction()方法：

```java
int actualResult = executeFunction((a, b) -> a + b, 5, 3);
```

此Lambda表达式(a, b)-> a + b表示对其两个输入求和的函数，整数5和3分别作为第二和第三个参数传递。

最后，我们使用断言来验证我们的实现是否按预期工作：

```java
assertEquals(8, actualResult);
```

## 6. 使用Callable类

我们还可以使用[Callable](https://www.baeldung.com/java-runnable-callable#2-with-callable)将方法作为参数传递，**Callable接口是java.util.concurrent包的一部分，表示返回结果并可能引发异常的任务，这在并发编程中特别有用**。

让我们探索如何使用Callable将方法作为参数传递。首先，我们创建接收Callable<Integer\>作为参数的executeCallable()方法，这意味着它接收一个返回Integer的任务：

```java
int executeCallable(Callable<Integer> task) throws Exception {
    return task.call();
}
```

call()方法用于执行任务并返回结果，**它可能会引发异常，因此我们需要适当处理它**。我们可以使用Lambda表达式或匿名内部类定义Callable任务。这里，为了简单起见，我们使用Lambda表达式：

```java
Callable<Integer> task = () -> 5 + 3;
```

这个Lambda表达式表示计算5和3之和的任务，然后我们可以调用executeCallable()方法并将Callable任务作为参数传递：

```java
int actualResult = executeCallable(task);
assertEquals(8, actualResult);
```

使用Callable将方法作为参数传递提供了一种替代方法，这在并发编程场景中特别有用。

## 7. 总结

在本文中，我们探讨了在Java中将方法作为参数传递的各种方法。对于简单的操作，Lambda表达式或方法引用通常是首选，因为它们简洁。对于复杂的操作，匿名内部类可能仍然合适。