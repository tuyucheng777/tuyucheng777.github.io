---
layout: post
title:  在Java中计算加权平均值
category: algorithms
copyright: algorithms
excerpt: 加权平均值
---

## 1. 简介

在本文中，我们将探讨几种不同的方法来解决同一个问题-计算一组值的加权平均值。

## 2. 什么是加权平均值？

我们通过将所有数字相加，然后除以数字的数量来计算一组数字的标准平均值。例如，数字1、3、5、7、9的平均值将是(1 + 3 + 5 + 7 + 9) / 5，等于5。

**当我们计算加权平均值时，我们会得到一组具有权重的数字**：

| 数字| 权重 |
| -------- |----|
|1        | 10 |
|3        | 20 |
|5        | 30 |
|7        | 50 |
|9        | 40 |

在这种情况下，我们需要考虑权重，**新的计算方法是将每个数字与其权重的乘积相加，然后除以所有权重的总和**。例如，这里的平均值将是((1 * 10) + (3 * 20) + (5 * 30) + (7 * 50) + (9 * 40)) / (10 + 20 + 30 + 50 + 40)，等于6.2。

## 3. 设置

为了这些例子，我们将进行一些初始设置，**最重要的是我们需要一个类型来表示我们的加权值**：
```java
private static class Values {
    int value;
    int weight;

    public Values(int value, int weight) {
        this.value = value;
        this.weight = weight;
    }
}
```

在我们的示例代码中，我们还将有一组初始值和平均值的预期结果：
```java
private List<Values> values = Arrays.asList(
    new Values(1, 10),
    new Values(3, 20),
    new Values(5, 30),
    new Values(7, 50),
    new Values(9, 40)
);

private Double expected = 6.2;
```

## 4. 两次计算

**计算这个的最明显方法正如我们上面所见，我们可以遍历数字列表并分别求出除法所需的值**：
```java
double top = values.stream()
        .mapToDouble(v -> v.value * v.weight)
        .sum();
double bottom = values.stream()
        .mapToDouble(v -> v.weight)
        .sum();
```

完成此操作后，我们的计算现在只是将两个数字相除：
```java
double result = top / bottom;
```

**我们可以进一步简化这一过程，改用传统的for循环，并在执行过程中进行两次求和**；缺点是结果不能是不可变的值：
```java
double top = 0;
double bottom = 0;

for (Values v : values) {
    top += (v.value * v.weight);
    bottom += v.weight;
}
```

## 5. 扩大列表

我们可以用不同的方式思考加权平均值的计算，**我们可以扩展每个加权值，而不是计算乘积之和**。例如，我们可以扩展列表以包含10个“1”的副本、20个“2”的副本，等等。此时，我们可以对扩展后的列表直接求平均值：
```java
double result = values.stream()
    .flatMap(v -> Collections.nCopies(v.weight, v.value).stream())
    .mapToInt(v -> v)
    .average()
    .getAsDouble();
```

这显然效率会降低，但也可能更清晰、更容易理解。我们还可以更轻松地对最终的数字集进行其他操作-例如，这样求[中位数](https://www.baeldung.com/java-stream-integers-median-using-heap)就更容易理解了。

## 6. 归约列表

我们已经知道，对乘积和权重求和比尝试展开值更有效。但是，如果我们想一次性完成此操作而不使用可变值怎么办？**我们可以使用Stream中的[reduce()](https://www.baeldung.com/java-stream-reduce)功能来实现这一点，具体来说，我们将使用它来执行加法运算，并将累计总数收集到一个对象中**。

我们想要做的第一件事就是建立一个类来收集运行总数：
```java
class WeightedAverage {
    final double top;
    final double bottom;

    public WeightedAverage(double top, double bottom) {
        this.top = top;
        this.bottom = bottom;
    }

    double average() {
        return top / bottom;
    }
}
```

我们还添加了一个average()函数，用于执行最终计算。现在，我们可以执行归约操作了：
```java
double result = values.stream()
    .reduce(new WeightedAverage(0, 0),
        (acc, next) -> new WeightedAverage(
            acc.top + (next.value * next.weight),
            acc.bottom + next.weight),
        (left, right) -> new WeightedAverage(
            left.top + right.top,
            left.bottom + right.bottom))
    .average();
```

这看起来很复杂，所以让我们把它分解成几个部分。

reduce()的第一个参数是我们的标识，这是值为0的加权平均值。

下一个参数是一个Lambda，它接收一个WeightedAverage实例并将下一个值添加到该实例中。我们会注意到，这里的和的计算方式与之前执行的方式相同。

最后一个参数是用于组合两个WeightedAverage实例的Lambda，这对于使用reduce()的某些情况是必需的，例如在并行流上执行此操作时。

Reduce()调用的结果是一个WeightedAverage实例，我们可以使用它来计算结果。

## 7. 自定义收集器

我们的reduce()版本确实简洁，但比其他尝试更难理解。我们最终将两个Lambda传递到函数中，并且仍然需要执行后处理步骤来计算平均值。

**我们可以探索的最后一个解决方案是编写一个自定义[收集器](https://www.baeldung.com/java-8-collectors)来封装这项工作，这将直接产生我们的结果，并且使用起来会更加简单**。

在编写收集器之前，让我们先看看需要实现的接口：
```java
public interface Collector<T, A, R> {
    Supplier<A> supplier();
    BiConsumer<A, T> accumulator();
    BinaryOperator<A> combiner();
    Function<A, R> finisher();
    Set<Characteristics> characteristics();
}
```

这里有很多事情要做，但我们会在构建收集器的过程中逐步解决。我们还将看到这些额外的复杂性如何让我们在[并行流](https://www.baeldung.com/java-when-to-use-parallel-stream)上使用完全相同的收集器，而不仅仅是在顺序流上使用。

首先要注意的是泛型类型：

- T：这是输入类型，我们的收集器始终需要与它能够收集的值的类型绑定
- R：这是结果类型，我们的收集器始终需要指定其将生成的类型
- A：这是聚合类型，这通常是收集器的内部类型，但对于某些函数签名而言是必需的

**这意味着我们需要定义一个聚合类型，它只是一种在运行时收集运行结果的类型**，我们不能直接在收集器中实现这一点，因为我们需要能够支持并行流，而并行流中可能同时运行着数量未知的并行流。因此，我们定义一个单独的类型来存储每个并行流的结果：
```java
class RunningTotals {
    double top;
    double bottom;

    public RunningTotals() {
        this.top = 0;
        this.bottom = 0;
    }
}
```

这是一种可变类型，但由于它的使用将被限制在一个并行流中，所以没关系。

现在，我们可以实现收集器方法。我们会注意到，其中大多数方法都返回Lambda。同样，这是为了支持并行流，底层流框架会根据需要调用这些方法的组合。

**第一个方法是supplier()**，这将构造一个新的、值为0的RunningTotals实例：
```java
@Override
public Supplier<RunningTotals> supplier() {
    return RunningTotals::new;
}
```

接下来，**我们有accumulator()**，它需要一个RunningTotals实例和下一个要处理的Values实例并将它们组合起来，从而更新我们的RunningTotals实例：
```java
@Override
public BiConsumer<RunningTotals, Values> accumulator() {
    return (current, next) -> {
        current.top += (next.value * next.weight);
        current.bottom += next.weight;
    };
}
```

下一个方法是Combiner()，它需要两个RunningTotals实例(来自不同的并行流)并将它们合并为一个：
```java
@Override
public BinaryOperator<RunningTotals> combiner() {
    return (left, right) -> {
        left.top += right.top;
        left.bottom += right.bottom;

        return left;
    };
}
```

在这种情况下，我们修改了其中一个输入并直接返回。这非常安全，但如果更方便的话，我们也可以返回一个新的实例。

仅当JVM决定将流处理拆分为多个并行流时才会使用此功能，这取决于几个因素，但是，我们应该实现它以防万一。

**我们需要实现的最后一个Lambda方法是finisher()**，该方法获取所有值累积并合并所有并行流后剩下的最终RunningTotals实例，并返回最终结果：
```java
@Override
public Function<RunningTotals, Double> finisher() {
    return rt -> rt.top / rt.bottom;
}
```

**我们的Collector还需要一个Characteristics()方法，该方法返回一组描述如何使用收集器的特征**，Collectors.Characteristics枚举由3个值组成：

- CONCURRENT：从并行线程中调用同一个聚合实例时，accumulator()函数是安全的，如果指定了CONCURRENT，则永远不会使用Combiner()函数，但使用Aggregation()函数时必须格外小心。
- UNORDERED：收集器可以安全地以任何顺序处理来自底层流的元素，如果未指定，则在可能的情况下，将以正确的顺序提供值。
- IDENTITY_FINISH：finisher()函数直接返回其输入，如果指定了此项，则收集过程可能会缩短此调用并直接返回值。

**在我们的例子中，我们有一个UNORDERED收集器，但需要省略另外两个**：
```java
@Override
public Set<Characteristics> characteristics() {
    return Collections.singleton(Characteristics.UNORDERED);
}
```

现在可以使用我们的收集器了：
```java
double result = values.stream().collect(new WeightedAverage());
```

**虽然编写收集器比以前复杂得多，但使用起来却容易得多**。我们还可以毫不费力地利用并行流之类的功能，这意味着，如果我们需要重用它，这将为我们提供一个更易于使用且更强大的解决方案。

## 8. 总结

到这里，我们已经了解了几种计算一组值的加权平均值的方法，从简单的循环遍历这些值，到编写一个完整的Collector实例，以便在需要执行此计算时可以重复使用。