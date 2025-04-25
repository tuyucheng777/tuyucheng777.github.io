---
layout: post
title:  使用FindBugs和PMD介绍代码质量规则
category: staticanalysis
copyright: staticanalysis
excerpt: FindBugs
---

## 1. 概述

在本文中，我们将重点介绍FindBugs、PMD和CheckStyle等代码分析工具中的一些重要规则。

## 2. 圈复杂度

### 2.1 什么是圈复杂度？

代码复杂度很重要，但衡量起来却很困难，PMD在其[代码大小规则部分](http://pmd.sourceforge.net/pmd-4.3.0/rules/codesize.html)提供了一套可靠的规则，旨在检测与方法大小和结构复杂度相关的违规行为。

CheckStyle以其根据编码标准和格式规则分析代码的能力而闻名，此外，它还可以通过计算一些复杂性[指标](https://checkstyle.sourceforge.io/checks/metrics/index.html)来检测类/方法设计中的问题。

这两种工具中最相关的复杂性测量之一是CC(圈复杂度)。

CC值可以通过测量程序的独立执行路径的数量来计算。

例如，以下方法将产生3的圈复杂度：

```java
public void callInsurance(Vehicle vehicle) {
    if (vehicle.isValid()) {
        if (vehicle instanceof Car) {
            callCarInsurance();
        } else {
            delegateInsurance();
        }
    }
}
```

CC考虑了条件语句和多部分布尔表达式的嵌套。

一般来说，CC值高于11的代码被认为非常复杂，并且难以测试和维护。

静态分析工具使用的一些常见值如下所示：

- 1-4：低复杂度-易于测试
- 5-7：中等复杂度-可以容忍
- 8-10：高复杂性-应考虑重构以简化测试
- 11+：非常高的复杂性-很难测试

复杂度级别也会影响代码的可测试性，**CC越高，执行相关测试的难度就越高**。实际上，圈复杂度值恰恰显示了达到100%分支覆盖率所需的测试用例数量。

与callInsurance()方法相关的流程图是：

![](/assets/images/2025/staticanalysis/codequalitymetrics01.png)

可能的执行路径有：

- 0 => 3
- 0 => 1 => 3
- 0 => 2 => 3

从数学上讲，CC可以使用以下简单公式计算：

```text
CC = E - N + 2P
```

- E：边的总数
- N：节点总数
- P：出口点总数

### 2.2 如何降低圈复杂度？

为了编写不太复杂的代码，开发人员可能会根据情况倾向于使用不同的方法：

- 避免使用设计模式编写冗长的Switch语句，例如，构建器和策略模式可能是处理代码大小和复杂性问题的良好选择。
- 通过模块化代码结构和实现[单一职责原则](https://en.wikipedia.org/wiki/Single_responsibility_principle)来编写可重用和可扩展的方法。
- **遵循其他PMD[代码大小](http://pmd.sourceforge.net/pmd-4.3.0/rules/codesize.html)规则可能会对CC产生直接影响**，例如过多的方法长度规则、单个类中的字段过多、单个方法中的参数列表过多...等等。

你还可以考虑遵循有关代码大小和复杂性的原则和模式，例如[KISS(保持简单和愚蠢)原则](https://en.wikipedia.org/wiki/KISS_principle)和[DRY(不要重复自己)](https://en.wikipedia.org/wiki/Don't_repeat_yourself)。

## 3. 异常处理规则

与异常相关的缺陷可能很常见，但其中一些缺陷被严重低估，应该予以纠正，以避免生产代码出现严重功能障碍。

PMD和 FindBugs都提供了一些关于异常的规则，以下是我们挑选出的在Java程序中处理异常时可能被认为至关重要的规则。

### 3.1 不要在Finally中抛出异常

你可能已经知道，Java中的finally{}块通常用于关闭文件和释放资源，将其用于其他目的可能会被视为[代码异味](https://en.wikipedia.org/wiki/Code_smell)。

一个典型的容易出错的例程是在finally{}块内抛出异常：

```java
String content = null;
try {
    String lowerCaseString = content.toLowerCase();
} finally {
    throw new IOException();
}
```

该方法应该抛出一个NullPointerException，但令人惊讶的是它抛出了IOException，这可能会误导调用方法处理错误的异常。

### 3.2 在finally块中返回

在finally{}块中使用return语句可能会让人感到困惑，这条规则之所以如此重要，是因为每当代码抛出异常时，它都会被return语句丢弃。

例如，以下代码运行时没有任何错误：

```java
String content = null;
try {
    String lowerCaseString = content.toLowerCase();
} finally {
    return;
}
```

NullPointerException未被捕获，但仍被finally块中的return语句丢弃。

### 3.3 发生异常时无法关闭流

关闭流是我们使用finally块的主要原因之一，但它并不像看起来那么简单。

下面的代码尝试在finally块中关闭两个流：

```java
OutputStream outStream = null;
OutputStream outStream2 = null;
try {
    outStream = new FileOutputStream("test1.txt");
    outStream2  = new FileOutputStream("test2.txt");
    outStream.write(bytes);
    outStream2.write(bytes);
} catch (IOException e) {
    e.printStackTrace();
} finally {
    try {
        outStream.close();
        outStream2.close();
    } catch (IOException e) {
        // Handling IOException
    }
}
```

如果outStream.close()指令抛出IOException，则将跳过outStream2.close()。

一个快速的解决方法是使用单独的try/catch块来关闭第二个流：

```java
finally {
    try {
        outStream.close();
    } catch (IOException e) {
        // Handling IOException
    }
    try {
        outStream2.close();
    } catch (IOException e) {
        // Handling IOException
    }
}
```

如果你想要一种避免连续try/catch块的好方法，请检查Apache commons中的[IOUtils.closeQuiety](https://commons.apache.org/proper/commons-io/javadocs/api-release/org/apache/commons/io/IOUtils.html)方法，它可以轻松处理流关闭而不会引发IOException。

## 4. 不良做法

### 4.1 类定义compareto()并使用Object.equals()

每当你实现compareTo()方法时，不要忘记对equals()方法执行相同的操作，否则，此代码返回的结果可能会令人困惑：

```java
Car car = new Car();
Car car2 = new Car();
if(car.equals(car2)) {
    logger.info("They're equal");
} else {
    logger.info("They're not equal");
}
if(car.compareTo(car2) == 0) {
    logger.info("They're equal");
} else {
    logger.info("They're not equal");
}
```

结果：

```text
They're not equal
They're equal
```

为了消除混淆，建议确保在实现Comparable时永远不会调用Object.equals()，相反，你应该尝试使用如下方式覆盖它：

```java
boolean equals(Object o) { 
    return compareTo(o) == 0; 
}
```

### 4.2 可能的空指针引用

NullPointerException(NPE)被认为是Java编程中最常遇到的异常，而FindBugs会抱怨空指针取消引用以避免抛出它。

以下是抛出NPE的最基本示例：

```java
Car car = null;
car.doSomething();
```

避免NPE的最简单方法是执行空检查：

```java
Car car = null;
if (car != null) {
    car.doSomething();
}
```

空检查可以避免NPE，但是如果广泛使用，肯定会影响代码的可读性。

因此，这里有一些用于避免没有空检查的NPE的技术：

- 编码时避免使用关键字null：这个规则很简单，在初始化变量或返回值时避免使用关键字null。
- 使用[@NotNull](http://docs.oracle.com/javaee/7/api/javax/validation/constraints/NotNull.html)和[@Nullable](http://findbugs.sourceforge.net/api/edu/umd/cs/findbugs/annotations/Nullable.html)注解。
- 使用java.util.Optional。
- 实现[空对象模式](https://en.wikipedia.org/wiki/Null_Object_pattern)。

## 5. 总结

在本文中，我们对静态分析工具检测到的一些关键缺陷进行了全面的介绍，并提供了适当解决检测到的问题的基本指导。

你可以通过访问以下链接浏览每个规则的完整规则集：[FindBugs](http://findbugs.sourceforge.net/bugDescriptions.html)、[PMD](http://pmd.sourceforge.net/pmd-4.3.0/rules/index.html)。