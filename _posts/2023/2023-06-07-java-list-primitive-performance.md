---
layout: post
title:  Java中原始List的性能比较
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在本教程中，我们将比较Java中一些流行的[原始列表库](https://www.baeldung.com/java-list-primitive-int)的性能。

为此，我们将测试每个库的add()、get()和contains()方法。

## 2. 性能比较

现在，**让我们找出哪个库提供了一个快速工作的原始集合API**。

为此，让我们比较Trove、Fastutil和Colt中的[List](https://www.baeldung.com/java-arraylist)类似物。我们将使用[JMH](https://www.baeldung.com/java-microbenchmark-harness)(Java Microbenchmark Harness)工具来编写我们的性能测试。

### 2.1 JMH参数

我们将使用以下参数运行基准测试：

```java
@BenchmarkMode(Mode.SingleShotTime)
@OutputTimeUnit(TimeUnit.MILLISECONDS)
@Measurement(batchSize = 100000, iterations = 10)
@Warmup(batchSize = 100000, iterations = 10)
@State(Scope.Thread)
public class PrimitivesListPerformance {
}
```

在这里，**我们要测量每个基准测试方法的执行时间**。此外，我们希望以毫秒为单位显示结果。

@State注解表明类中声明的变量不会成为运行基准测试的一部分。但是，我们可以在我们的基准测试方法中使用它们。

此外，让我们定义并初始化我们的原始类型列表：

```java
public static class PrimitivesListPerformance {
    private List<Integer> arrayList = new ArrayList<>(Arrays.asList(0, 1, 2, 3, 4, 5, 6, 7, 8, 9));
    private TIntArrayList tList = new TIntArrayList(new int[]{0, 1, 2, 3, 4, 5, 6, 7, 8, 9});
    private cern.colt.list.IntArrayList coltList = new cern.colt.list.IntArrayList(new int[]{0, 1, 2, 3, 4, 5, 6, 7, 8, 9});
    private IntArrayList fastUtilList = new IntArrayList(new int[]{0, 1, 2, 3, 4, 5, 6, 7, 8, 9});

    private int getValue = 4;
}
```

## 3. add()

首先，让我们测试将元素添加到原始列表中。我们还将为ArrayList添加一个作为我们的控件。

### 3.1 基准测试

第一个微基准测试是针对[ArrayList](https://www.baeldung.com/java-arraylist)的add()方法：

```java
@Benchmark
public boolean addArrayList() {
    return arrayList.add(getValue);
}
```

同样，对于Trove的TIntArrayList.add()：

```java
@Benchmark
public boolean addTroveIntList() {
    return tList.add(getValue);
}
```

同样，Colt的IntArrayList.add()看起来像：

```java
@Benchmark
public void addColtIntList() {
    coltList.add(getValue);
}
```

并且，对于Fastutil库，IntArrayList.add()方法基准测试将是：

```java
@Benchmark
public boolean addFastUtilIntList() {
    return fastUtilList.add(getValue);
}
```

### 3.2 测试结果

现在，我们运行并比较结果：

```text
Benchmark           Mode  Cnt  Score   Error  Units
addArrayList          ss   10  4.527 ± 4.866  ms/op
addColtIntList        ss   10  1.823 ± 4.360  ms/op
addFastUtilIntList    ss   10  2.097 ± 2.329  ms/op
addTroveIntList       ss   10  3.069 ± 4.026  ms/op
```

从结果中，我们可以清楚地看到ArrayList的add()是最慢的选项。

这是合乎逻辑的，正如我们在[原始列表库](https://www.baeldung.com/java-list-primitive-int)文章中解释的那样，ArrayList将使用装箱/自动装箱将int值存储在集合中。因此，我们在这里有明显的放缓。

另一方面，Colt和Fastutil的add()方法是最快的。

在幕后，所有三个库都将值存储在int[]中。那么为什么我们的add()方法有不同的运行时间呢？

答案是当默认容量已满时他们如何增加int[]：

-    **Colt只有在它变满时才会增长其内部int[]**
-   相比之下，**Trove和Fastutil在扩展int[]容器的同时会使用一些额外的计算**

这就是Colt在我们的测试结果中获胜的原因。

## 4. get()

现在，让我们添加get()操作微基准测试。

### 4.1 基准测试

首先，对于ArrayList的get()操作：

```java
@Benchmark
public int getArrayList() {
    return arrayList.get(getValue);
}
```

同样，对于Trove的TIntArrayList，我们将有：

```java
@Benchmark
public int getTroveIntList() {
    return tList.get(getValue);
}
```

并且，对于Colt的cern.colt.list.IntArrayList，get()方法将是：

```java
@Benchmark
public int getColtIntList() {
    return coltList.get(getValue);
}
```

最后，对于Fastutil的IntArrayList，我们将测试getInt()操作：

```java
@Benchmark
public int getFastUtilIntList() {
    return fastUtilList.getInt(getValue);
}
```

### 4.2 测试结果

之后，我们运行基准测试并查看结果：

```text
Benchmark           Mode  Cnt  Score   Error  Units
getArrayList        ss     20  5.539 ± 0.552  ms/op
getColtIntList      ss     20  4.598 ± 0.825  ms/op
getFastUtilIntList  ss     20  4.585 ± 0.489  ms/op
getTroveIntList     ss     20  4.715 ± 0.751  ms/op
```

虽然分数差异不大，但我们可以注意到getArrayList()的工作速度较慢。

对于其余的库，我们有几乎相同的get()方法实现。**他们将立即从int[]中检索值，无需任何进一步的工作**。这就是为什么Colt、Fastutil和Trove对于get()操作具有相似的性能。

## 5. contains()

最后，让我们为每种类型的列表测试contains()方法。

### 5.1 基准测试

让我们为ArrayList的contains()方法添加第一个微基准测试：

```java
@Benchmark
public boolean containsArrayList() {
    return arrayList.contains(getValue);
}
```

同样，对于Trove的TIntArrayList，contains()基准测试将是：

```java
@Benchmark
public boolean containsTroveIntList() {
    return tList.contains(getValue);
}
```

同样，Colt的cern.colt.list.IntArrayList.contains()的测试是：

```java
@Benchmark
public boolean containsColtIntList() {
    return coltList.contains(getValue);
}
```

最后，对于Fastutil的IntArrayList，contains()方法测试看起来像：

```java
@Benchmark
public boolean containsFastUtilIntList() {
    return fastUtilList.contains(getValue);
}
```

### 5.2 测试结果

最后，我们运行测试并比较结果：

```text
Benchmark                  Mode  Cnt   Score    Error  Units
containsArrayList          ss     20   2.083  ± 1.585  ms/op
containsColtIntList        ss     20   1.623  ± 0.960  ms/op
containsFastUtilIntList    ss     20   1.406  ± 0.400  ms/op
containsTroveIntList       ss     20   1.512  ± 0.307  ms/op
```

**和往常一样，containsArrayList方法的性能最差**。相比之下，与Java的核心解决方案相比，Trove，Colt和Fastutil具有更好的性能。

这一次，由开发人员选择哪个库。所有三个库的结果非常接近，可以认为它们是相同的。

## 6.  总结

在本文中，我们通过JVM基准测试研究了原始列表的实际运行时性能。此外，我们将测试结果与JDK的ArrayList进行了比较。

另外，请记住，我们在此处提供的数字只是JMH基准测试结果-始终在给定系统和运行时的范围内进行测试。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-list-3)上获得。