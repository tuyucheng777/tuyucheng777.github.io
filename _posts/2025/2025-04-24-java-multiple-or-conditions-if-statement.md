---
layout: post
title:  Java中If语句中多个“或”条件的格式
category: java-new
copyright: java-new
excerpt: Java 21
---

## 1. 概述

在使用if语句时，我们可能需要使用[逻辑运算符](https://www.baeldung.com/java-operators#logical-operators)(例如AND或OR)来处理多个条件，这可能不是一个简洁的设计，并且会影响代码的可读性和[认知复杂性](https://www.baeldung.com/java-cognitive-complexity)。

在本教程中，我们将看到在if语句中格式化多个值条件的替代方法。

## 2. 我们可以避免if语句吗？

假设我们有一个电商平台，并针对特定月份出生的人设置了折扣，我们来看一段代码：

```java
if (month == 10 || month == 11) {
    // doSomething()
} else if(month == 4 || month == 5) {
    // doSomething2()
} else {
    // doSomething3()
}
```

这可能会导致一些代码难以阅读，此外，即使我们有良好的测试覆盖率，代码也可能很难维护，因为它迫使我们将执行特定操作的不同条件放在一起。

### 2.1 使用干净的代码

我们可以应用模式来[替换许多if语句](https://www.baeldung.com/java-replace-if-statements)，例如，我们可以将if的多个条件的逻辑移到类或枚举中。在运行时，我们将根据客户端输入在接口之间进行切换。同样，我们可以看看[策略模式](https://www.baeldung.com/java-strategy-pattern)。

这并非严格意义上的格式问题，通常需要重新思考逻辑，尽管如此，我们仍然可以考虑改进设计。

### 2.2 改进方法语法

**但是，只要代码可读性好且易于维护，使用if/else逻辑也没什么问题**。例如，让我们考虑以下代码片段：

```java
if (month == 8 || month == 9) {
    return doSomething();
} else {
    return doSomethingElse();
}
```

第一步，我们可以避免使用else部分：

```java
if (month == 8 || month == 9) {
    return doSomething();
}

return doSomethingElse();
```

另外，还可以改进其他一些代码，例如，用java.time包的Enum替换月份数字：

```java
if (month == OCTOBER || month == NOVEMBER || month == DECEMBER) {
    return doSomething();
}
// ...
```

这些都是简单而有效的代码改进，因此，在应用复杂模式之前，我们首先应该看看是否可以提高代码的可读性。

我们还将了解如何使用函数式编程，在Java中，这从版本8开始通过Lambda表达式语法实现。

## 3. 测试图例

参照电商折扣示例，我们将创建测试并检查折扣月份中的值，例如，从10月到12月，否则我们将断言false，我们将设置允许范围内或范围外的随机月份：

```java
Month monthIn() {
    return Month.of(rand.ints(10, 13)
        .findFirst()
        .orElse(10));
}

Month monthNotIn() {
    return Month.of(rand.ints(1, 10)
        .findFirst()
        .orElse(1));
}
```

可能有多个if条件，但为了简单起见，我们假设只有一个if/else语句。

## 4. 使用Switch

**除了使用if逻辑之外，还可以使用[switch](https://www.baeldung.com/java-switch)命令**，让我们看看如何在示例中使用它：

```java
boolean switchMonth(Month month) {
    switch (month) {
        case OCTOBER:
        case NOVEMBER:
        case DECEMBER:
            return true;
        default:
            return false;
    }
}
```

请注意，如果需要，它将向下移动并检查所有有效月份。此外，我们可以使用Java 12中的新Switch语法来改进这一点：

```java
return switch (month) {
    case OCTOBER, NOVEMBER, DECEMBER -> true;
    default -> false;
};
```

最后，我们可以做一些测试来验证值是否在范围内：

```java
assertTrue(switchMonth(monthIn()));
assertFalse(switchMonth(monthNotIn()));
```

## 5. 使用集合

**我们可以使用集合来对满足if条件的内容进行分组，并检查某个值是否属于它**：

```java
Set<Month> months = Set.of(OCTOBER, NOVEMBER, DECEMBER);
```

让我们添加一些逻辑来查看集合是否包含特定值：

```java
boolean contains(Month month) {
    if (months.contains(month)) {
        return true;
    }
    return false;
}
```

同样，我们可以添加一些单元测试：

```java
assertTrue(contains(monthIn()));
assertFalse(contains(monthNotIn()));
```

## 6. 使用函数式编程

**我们可以使用[函数式编程](https://www.baeldung.com/java-functional-programming)将if/else逻辑转换为函数**，按照这种方法，我们将能够更准确地使用方法语法。

### 6.1 简单谓词

我们仍然使用contains()方法，不过，这次我们使用[Predicate](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/function/Predicate.html)将其变为Lambda表达式：

```java
Predicate<Month> collectionPredicate = this::contains;
```

现在我们可以确定Predicate是不可变的，没有中间变量，其结果是可预测的，并且可以在其他情况下重复使用。

让我们使用test()方法来检查一下：

```java
assertTrue(collectionPredicate.test(monthIn()));
assertFalse(collectionPredicate.test(monthNotIn()));
```

### 6.2 谓词链

我们可以链接多个Predicate，在or条件中添加我们的逻辑：

```java
Predicate<Month> orPredicate() {
    Predicate<Month> predicate = x -> x == OCTOBER;
    Predicate<Month> predicate1 = x -> x == NOVEMBER;
    Predicate<Month> predicate2 = x -> x == DECEMBER;

    return predicate.or(predicate1).or(predicate2);
}
```

然后我们可以将其代入if中：

```java
boolean predicateWithIf(Month month) {
    if (orPredicate().test(month)) {
        return true;
    }
    return false;
}
```

让我们通过测试来检查一下这是否有效：

```java
assertTrue(predicateWithIf(monthIn()));
assertFalse(predicateWithIf(monthNotIn()));
```

### 6.3 流中的谓词

类似地，我们可以在[流过滤器](https://www.baeldung.com/java-stream-filter-lambda)中使用Predicate。同样，过滤器中的Lambda表达式将取代并增强if逻辑，if最终会消失，这是函数式编程的优势，同时仍然保持了良好的性能和可读性。

让我们在解析月份的输入列表时对此进行测试：

```java
List<Month> monthList = List.of(monthIn(), monthIn(), monthNotIn());

monthList.stream()
    .filter(this::contains)
    .forEach(m -> assertThat(m, is(in(months))));
```

我们也可以使用predicateWithIf()代替contains()，如果Lambda支持方法签名，则没有任何限制。例如，它可以是静态方法。

## 7. 总结

在本教程中，我们学习了如何提高if语句中多个条件的可读性，我们还了解了如何使用Switch语句。此外，我们还介绍了如何使用集合来检查其是否包含某个值。最后，我们了解了如何使用Lambda表达式来实现函数式编程，Predicate和Stream更不容易出错，并且可以增强代码的可读性和维护性。
