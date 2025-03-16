---
layout: post
title:  Java中的Void类型
category: java-reflect
copyright: java-reflect
excerpt: Java反射
---

## 1. 概述

作为Java开发人员，我们可能在某些场合遇到过Void类型，并想知道它的用途是什么。

在这个快速教程中，我们将了解这个特殊的类，并了解何时以及如何使用它以及如何尽可能避免使用它。

## 2. 什么是Void类型

从JDK 1.1开始，Java为我们提供了Void类型，**其目的只是将void返回类型表示为类并包含Class<Void\>公共值**。它不可实例化，因为它的唯一构造函数是私有的。

因此，我们可以分配给Void变量的唯一值是null。它看起来可能有点没用，但现在我们将看到何时以及如何使用这种类型。

## 3. 用法

在某些情况下使用Void类型会很有趣。

### 3.1 反射

首先，我们可以在进行反射时使用它。实际上，**任何void方法的返回类型都将与前面提到的保存Class<Void\>值的Void.TYPE变量匹配**。

让我们想象一个简单的Calculator类：

```java
public class Calculator {
    private int result = 0;

    public int add(int number) {
        return result += number;
    }

    public int sub(int number) {
        return result -= number;
    }

    public void clear() {
        result = 0;
    }

    public void print() {
        System.out.println(result);
    }
}
```

有些方法返回int，有些则不返回任何内容。现在，**假设我们必须通过反射检索所有不返回任何结果的方法**。我们将使用Void.TYPE变量来实现这一点：

```java
@Test
void givenCalculator_whenGettingVoidMethodsByReflection_thenOnlyClearAndPrint() {
    Method[] calculatorMethods = Calculator.class.getDeclaredMethods();
    List<Method> calculatorVoidMethods = Arrays.stream(calculatorMethods)
        .filter(method -> method.getReturnType().equals(Void.TYPE))
        .collect(Collectors.toList());

    assertThat(calculatorVoidMethods)
        .allMatch(method -> Arrays.asList("clear", "print").contains(method.getName()));
}
```

我们可以看到，只检索了clear()和print()方法。

### 3.2 泛型

Void类型的另一种用法是用于泛型类，假设我们正在调用一个需要Callable参数的方法：

```java
public class Defer {
    public static <V> V defer(Callable<V> callable) throws Exception {
        return callable.call();
    }
}
```

**但是，我们想要传递的Callable不必返回任何内容。因此，我们可以传递一个Callable<Void\>**：

```java
@Test
void givenVoidCallable_whenDiffer_thenReturnNull() throws Exception {
    Callable<Void> callable = new Callable<Void>() {
        @Override
        public Void call() {
            System.out.println("Hello!");
            return null;
        }
    };

    assertThat(Defer.defer(callable)).isNull();
}
```

如上所示，**为了从返回类型为Void的方法返回，我们只需返回null**。此外，我们可以使用随机类型(例如Callable<Integer\>)并返回null或根本不返回类型(Callable)，但使用Void清楚地表明了我们的意图。

我们也可以将此方法应用于Lambda。事实上，我们的Callable可以写成Lambda。让我们想象一个需要Function的方法，但我们想使用一个不返回任何内容的Function，那么我们只需要让它返回Void：

```java
public static <T, R> R defer(Function<T, R> function, T arg) {
    return function.apply(arg);
}

@Test
void givenVoidFunction_whenDiffer_thenReturnNull() {
    Function<String, Void> function = s -> {
        System.out.println("Hello " + s + "!");
        return null;
    };

    assertThat(Defer.defer(function, "World")).isNull();
}
```

## 4. 如何避免使用它？

现在，我们已经了解了Void类型的一些用法。但是，即使第一种用法完全没问题，我们可能还是希望**尽可能避免在泛型中使用Void**。事实上，遇到表示没有结果并且只能包含null的返回类型可能会很麻烦。

现在我们来看看如何避免这些情况。首先，让我们考虑一下带有Callable参数的方法。**为了避免使用Callable<Void\>，我们可以提供另一种接收Runnable参数的方法**：

```java
public static void defer(Runnable runnable) {
    runnable.run();
}
```

因此，我们可以传递一个不返回任何值的Runnable，从而摆脱无用的return null：

```java
Runnable runnable = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello!");
    }
};

Defer.defer(runnable);
```

但是，如果Defer类不是我们可以修改的呢？那么我们可以坚持使用Callable<Void\>选项，或者**创建另一个接收Runnable的类，并将调用推迟到Defer类**：

```java
public class MyOwnDefer {
    public static void defer(Runnable runnable) throws Exception {
        Defer.defer(new Callable<Void>() {
            @Override
            public Void call() {
                runnable.run();
                return null;
            }
        });
    }
}
```

通过这样做，我们将繁琐的部分一劳永逸地封装在我们自己的方法中，以便未来的开发人员使用更简单的API。

当然，Function也可以实现同样的效果。在我们的例子中，Function不返回任何内容，因此我们可以提供另一种接收Consumer的方法：

```java
public static <T> void defer(Consumer<T> consumer, T arg) {
    consumer.accept(arg);
}
```

那么，如果我们的函数不接收任何参数怎么办？我们可以使用Runnable或创建自己的函数接口(如果这看起来更清楚的话)：

```java
public interface Action {
    void execute();
}
```

然后，我们再次重载defer()方法：

```java
public static void defer(Action action) {
    action.execute();
}
Action action = () -> System.out.println("Hello!");

Defer.defer(action);
```

## 5. 总结

在这篇短文中，我们介绍了Java Void类。我们了解了它的用途以及如何使用它，并了解了它的一些替代用法。