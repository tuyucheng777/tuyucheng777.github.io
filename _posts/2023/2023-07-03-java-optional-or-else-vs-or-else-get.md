---
layout: post
title:  Java Optional：orElse()与orElseGet()
category: java
copyright: java
excerpt: Java Optional
---

## 1. 概述

Optional的 API通常有两个可能引起混淆的方法：orElse()和orElseGet()。

在本快速教程中，我们将介绍这两者之间的区别并探讨何时使用它们。

## 2. 签名

首先，让我们从基础开始，看看他们的签名：

```java
public T orElse(T other)

public T orElseGet(Supplier<? extends T> other)
```

显然，orElse()接收类型T的任何参数，而orElseGet()接收返回T类型的对象的Supplier类型的函数接口。

基于他们的[Javadocs](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html#orElse(T))：

-   orElse()：如果存在则返回值，否则返回other
-   orElseGet()：如果存在则返回值，否则调用other并返回其调用结果

## 3. 差异

这些简化的定义很容易让人感到困惑，所以让我们深入挖掘一下，看看一些实际的使用场景。

### 3.1 orElse()

假设我们已经正确配置了[Logger](https://www.baeldung.com/java-logging-intro)，让我们从编写一段简单的代码开始：

```java
String name = Optional.of("tuyucheng")
    .orElse(getRandomName());
```

请注意，getRandomName()是一个从名称列表中返回随机name的方法：

```java
public String getRandomName() {
    LOG.info("getRandomName() method - start");
    
    Random random = new Random();
    int index = random.nextInt(5);
    
    LOG.info("getRandomName() method - end");
    return names.get(index);
}
```

在执行我们的代码时，我们会发现控制台中打印了以下消息：

```text
getRandomName() method - start
getRandomName() method - end
```

变量name将在代码执行结束时保留“tuyucheng”。

有了它，**我们可以很容易地推断出orElse()的参数被评估了，即使有一个非空的Optional**。

### 3.2 orElseGet()

现在让我们尝试使用orElseGet()编写类似的代码：

```java
String name = Optional.of("tuyucheng")
    .orElseGet(() -> getRandomName());
```

上面的代码不会调用getRandomName()方法。

**请记住(来自Javadoc)作为参数传递的Supplier方法仅在Optional值不存在时执行**。

因此，在我们的案例中使用orElseGet()将为我们节省计算随机名称的时间。

## 4. 衡量性能影响

现在，为了也了解性能差异，让我们使用[JMH](https://www.baeldung.com/java-microbenchmark-harness)并观察一些实际数字：

```java
@Benchmark
@BenchmarkMode(Mode.AverageTime)
public String orElseBenchmark() {
    return Optional.of("tuyucheng").orElse(getRandomName());
}
```

和orElseGet()：

```java
@Benchmark
@BenchmarkMode(Mode.AverageTime)
public String orElseGetBenchmark() {
    return Optional.of("tuyucheng").orElseGet(() -> getRandomName());
}
```

在执行我们的基准测试方法时，我们得到：

```text
Benchmark           Mode  Cnt      Score       Error  Units
orElseBenchmark     avgt   20  60934.425 ± 15115.599  ns/op
orElseGetBenchmark  avgt   20      3.798 ±     0.030  ns/op
```

正如我们所见，即使对于这样一个简单的用例场景，性能影响也可能很大。

上面的数字可能略有不同；但是，**对于我们的特定示例，orElseGet()的性能明显优于orElse()**。

毕竟，orElse()涉及每次运行的getRandomName()方法的计算。

## 5. 什么重要？

除了性能方面，其他值得考虑的因素包括：

-   如果该方法将执行一些额外的逻辑怎么办？例如进行一些数据库插入或更新

-   即使我们将对象分配给orElse()参数，我们仍然无缘无故地创建“other”对象：

    ```java
    String name = Optional.of("tuyucheng").orElse("Other")
    ```

这就是为什么根据我们的需要在orElse()和orElseGet()之间做出谨慎的决定对我们来说很重要。**默认情况下，每次都使用orElseGet()更有意义，除非默认对象已经构建并且可以直接访问**。

## 6. 总结

在本文中，我们了解了Optional orElse()和OrElseGet()方法之间的细微差别，我们还讨论了这些简单的概念有时如何具有更深的含义。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-optional)上获得。