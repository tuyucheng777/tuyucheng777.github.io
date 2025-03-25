---
layout: post
title:  Atlassian Fugue简介
category: libraries
copyright: libraries
excerpt: Fugue
---

## 1. 简介

[Fugue](https://bitbucket.org/atlassian/fugue)是Atlassian的Java库；它是支持函数式编程的实用程序集合。

在这篇文章中，我们将重点关注并探索最重要的Fugue API。

## 2. Fugue入门

要在我们的项目中开始使用Fugue，我们需要添加以下依赖：

```xml
<dependency>
    <groupId>io.atlassian.fugue</groupId>
    <artifactId>fugue</artifactId>
    <version>4.5.1</version>
</dependency>
```

我们可以在Maven Central上找到最新版本的[Fugue](https://mvnrepository.com/artifact/io.atlassian.fugue/fugue)。

## 3. Option 

让我们从Option类开始我们的旅程，它是Fugue对java.util.Optional的增强。

从名称上我们可以猜出，**Option是一个表示可能缺失的值的容器**。

换句话说，Option要么是某种类型的某个值，要么是None：

```java
Option<Object> none = Option.none();
assertFalse(none.isDefined());

Option<String> some = Option.some("value");
assertTrue(some.isDefined());
assertEquals("value", some.get());

Option<Integer> maybe = Option.option(someInputValue);
```

### 3.1 map操作

标准函数式编程API之一是map()方法，它允许将提供的函数应用于底层元素。

如果存在，该方法将提供的函数应用于Option的值：

```java
Option<String> some = Option.some("value") 
    .map(String::toUpperCase);
assertEquals("VALUE", some.get());
```

### 3.2 Option和Null值

除了命名差异之外，Atlassian确实为Option做出了一些不同于Optional的设计选择；现在让我们看看它们。

**我们不能直接创建一个持有空值的非空Option**：

```java
Option.some(null);
```

以上抛出异常。

**但是，我们可以通过使用map()操作得到一个**：

```java
Option<Object> some = Option.some("value")
    .map(x -> null);
assertNull(some.get());
```

这在仅使用java.util.Optional时是不可能的。

### 3.3 Option是Iterable

Option可以被视为最多包含一个元素的集合，因此实现Iterable接口是有意义的。

这大大提高了处理集合/流时的互操作性。

现在，例如，可以与另一个集合拼接：

```java
Option<String> some = Option.some("value");
Iterable<String> strings = Iterables
    .concat(some, Arrays.asList("a", "b", "c"));
```

### 3.4 将Option转换为Stream

由于Option是Iterable，因此它也可以很容易地转换为Stream。

转换后，如果Option存在，则Stream实例将只有1个元素，否则为0：

```java
assertEquals(0, Option.none().toStream().count());
assertEquals(1, Option.some("value").toStream().count());
```

### 3.5 java.util.Optional互操作性

如果我们需要一个标准的Optional实现，我们可以使用toOptional()方法轻松获得它：

```java
Optional<Object> optional = Option.none()
    .toOptional();
assertTrue(Option.fromOptional(optional)
    .isEmpty());
```

### 3.6 Options实用程序类

最后，Fugue在恰当命名的Options类中提供了一些用于处理Option的实用方法。

它具有诸如filterNone之类的方法，用于从集合中删除空Options，以及flatten用于将Options集合转换为封闭对象集合，过滤掉空Options。

此外，它还具有几种lift方法的变体，可以将Function<A,B\>提升为Function<Option<A\>, Option<B>\>：

```java
Function<Integer, Integer> f = (Integer x) -> x > 0 ? x + 1 : null;
Function<Option<Integer>, Option<Integer>> lifted = Options.lift(f);

assertEquals(2, (long) lifted.apply(Option.some(1)).get());
assertTrue(lifted.apply(Option.none()).isEmpty());
```

当我们想要将一个不知道Option的函数传递给某些使用Option的方法时，这很有用。

请注意，就像map方法一样，**lift不会将null映射到None**：

```java
assertEquals(null, lifted.apply(Option.some(0)).get());
```

## 4. Either

正如我们所见，Option类允许我们以函数式方式处理值缺失的情况。

但是，有时我们需要返回比“无值”更多的信息；例如，我们可能想要返回一个合法值或一个错误对象。

Either类涵盖了该用例。

**Either的实例可以是Right或Left，但不能同时是两者**。

按照惯例，Right是成功计算的结果，而Left是例外情况。

### 4.1 构造Either

我们可以通过调用它的两个静态工厂方法之一来获得一个Either实例。

如果我们想要一个包含Right值的Either，我们调用right：

```java
Either<Integer, String> right = Either.right("value");
```

否则，我们调用left：

```java
Either<Integer, String> left = Either.left(-1);
```

在这里，我们的计算可以返回一个字符串或一个整数。

### 4.2 使用Either

当我们有一个Either实例时，我们可以检查它是左还是右并相应地采取行动：

```java
if (either.isRight()) {
    // ...
}
```

更有趣的是，我们可以使用函数式风格链接操作：

```java
either
    .map(String::toUpperCase)
    .getOrNull();
```

### 4.3 投影

Either与其他monadic工具(如Option、Try)的主要区别在于它通常是无偏的。简单地说，如果我们调用map()方法，Either不知道是使用Left侧还是Right侧。

这时，投影就派上用场了。

**左投影和右投影是Either的镜面视图，分别关注左值或右值**：

```java
either.left()
    .map(x -> decodeSQLErrorCode(x));
```

在上面的代码片段中，如果Either为Left，则decodeSQLErrorCode()将应用于基础元素。如果Either是Right，它就不会。使用Right投影时，反之亦然。

### 4.4 实用方法

与Options一样，Fugue也为Eithers提供了一个充满实用程序的类，它的名称就是Eithers。

它包含用于过滤、转换和迭代Either集合的方法。

## 5. Try异常处理

我们用另一种名为Try的变体来结束对Fugue中非此即彼数据类型的介绍。

Try类似于Either，但不同之处在于它专用于处理异常。

与Option相似，但与Either不同，Try通过单一类型进行参数化，因为“其他”类型固定为Exception(而对于Option，它隐式为Void)。

因此，Try可以是Success也可以是Failure：

```java
assertTrue(Try.failure(new Exception("Fail!")).isFailure());
assertTrue(Try.successful("OK").isSuccess());
```

### 5.1 实例化Try 

通常，我们不会明确地创建一个Try作为成功或失败；相反，我们会通过方法调用来创建一个。

Checked.of调用给定的函数并返回一个封装其返回值或任何抛出的异常的Try：

```java
assertTrue(Checked.of(() -> "ok").isSuccess());
assertTrue(Checked.of(() -> { throw new Exception("ko"); }).isFailure());
```

另一种方法Checked.lift接收一个可能抛出异常的函数，并将其提升为一个返回Try的函数：

```java
Checked.Function<String, Object, Exception> throwException = (String x) -> {
    throw new Exception(x);
};
        
assertTrue(Checked.lift(throwException).apply("ko").isFailure());
```

### 5.2 使用Try 

一旦我们有了Try，我们最终可能想用它做的最常见的三件事是：

1.  提取其值
2.  将某些操作链接到成功的值
3.  使用函数处理异常

此外，很明显，丢弃Try或将其传递给其他方法，上述三种方法并不是我们唯一的选择，但所有其他内置方法只是对这三种方法的一种方便。

### 5.3 提取Try值

要提取值，我们使用getOrElse方法：

```java
assertEquals(42, failedTry.getOrElse(() -> 42));
```

如果存在则返回成功值，否则返回某个计算值。

没有getOrThrow或类似的方法，但由于getOrElse没有捕获任何异常，我们可以轻松地编写它：

```java
someTry.getOrElse(() -> {
    throw new NoSuchElementException("Nothing to get");
});
```

### 5.4 成功后链接调用

在函数式风格中，我们可以将函数应用于成功值(如果存在)，而无需先显式提取它。

这是我们在Option、Either和大多数其他容器和集合中找到的典型map方法：

```java
Try<Integer> aTry = Try.successful(42).map(x -> x + 1);
```

它返回一个Try以便我们可以链接进一步的操作。

当然，我们也有flatMap变种：

```java
Try.successful(42).flatMap(x -> Try.successful(x + 1));
```

### 5.5 从异常中恢复

我们有类似的映射操作，将Try与异常使用(如果存在)，而不是它的成功值。

但是，这些方法的不同之处在于它们的含义是从异常中恢复，即在默认情况下产生成功的Try。

因此，我们可以使用recover产生一个新值：

```java
Try<Object> recover = Try
    .failure(new Exception("boo!"))
    .recover((Exception e) -> e.getMessage() + " recovered.");

assertTrue(recover.isSuccess());
assertEquals("boo! recovered.", recover.getOrElse(() -> null));
```

如我们所见，恢复函数将异常作为其唯一参数。

如果恢复函数本身抛出，则结果是另一个失败的Try：

```java
Try<Object> failure = Try.failure(new Exception("boo!")).recover(x -> {
    throw new RuntimeException(x);
});

assertTrue(failure.isFailure());
```

与flatMap类似的是restoreWith：

```java
Try<Object> recover = Try
    .failure(new Exception("boo!"))
    .recoverWith((Exception e) -> Try.successful("recovered again!"));

assertTrue(recover.isSuccess());
assertEquals("recovered again!", recover.getOrElse(() -> null));
```

## 6. 其他实用程序

在结束之前，让我们快速浏览一下Fugue中的其他一些实用程序。

### 6.1 Pair

Pair是一种非常简单且用途广泛的数据结构，由两个同等重要的组件组成，Fugue称之为left和right：

```java
Pair<Integer, String> pair = Pair.pair(1, "a");
        
assertEquals(1, (int) pair.left());
assertEquals("a", pair.right());
```

除了映射和应用函子模式之外，Fugue没有提供许多针对Pair的内置方法。

然而，Pair在整个库中使用，并且它们可以随时供用户程序使用。

### 6.2 Unit

Unit是一个具有单个值的枚举，表示“无值”。

它是void返回类型和Void类的替代品，取消了null：

```java
Unit doSomething() {
    System.out.println("Hello! Side effect");
    return Unit();
}
```

然而，令人惊讶的是，**Option不理解Unit，而是将其视为某种值而不是无值**。

### 6.3 静态工具

我们有几个类充满了静态实用方法，我们不需要编写和测试它们。

Functions类提供以各种方式使用和转换函数的方法：组合、应用、柯里化、使用Option的部分函数、弱记忆等。

Suppliers类为Supplier提供了一个类似但更有限的实用程序集合，即没有参数的函数。

最后，Iterables和Iterators包含大量静态方法，用于操作这两个广泛使用的标准Java接口。

## 7. 总结

在本文中，我们概述了Atlassian的Fugue库。

我们没有接触像Monoid和Semigroups这样的代数密集型类，因为它们不适合写在通才文章中。

但是，你可以在[Fugue Javadoc](https://docs.atlassian.com/fugue/4.5.1/fugue/apidocs/index.html)和[源代码](https://bitbucket.org/atlassian/fugue/src)中阅读有关它们的更多信息。

我们也没有涉及任何可选模块，例如提供与Guava和Scala的集成。