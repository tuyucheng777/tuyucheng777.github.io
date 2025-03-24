---
layout: post
title:  Functional Java简介
category: libraries
copyright: libraries
excerpt: Functional
---

## 1. 概述

在本教程中，我们将提供[Functional Java](https://github.com/functionaljava/functionaljava)库的快速概述以及一些示例。

## 2. Functional Java库

Functional Java是一个开源库，旨在促进Java中的函数式编程。该库提供了许多[函数式编程](https://www.baeldung.com/cs/functional-programming)中常用的基本和高级编程抽象。

该库的大部分功能都围绕F接口展开，此F接口模拟了一个函数，归约。所有这些都建立在Java自己的类型系统之上。

## 3. Maven依赖

首先，我们需要在pom.xml文件中添加所需的[依赖](https://mvnrepository.com/artifact/org.functionaljava)：

```xml
<dependency>
    <groupId>org.functionaljava</groupId>
    <artifactId>functionaljava</artifactId>
    <version>4.8.1</version>
</dependency>
<dependency>
    <groupId>org.functionaljava</groupId>
    <artifactId>functionaljava-java8</artifactId>
    <version>4.8.1</version>
</dependency>
<dependency>
    <groupId>org.functionaljava</groupId>
    <artifactId>functionaljava-quickcheck</artifactId>
    <version>4.8.1</version>
</dependency>
<dependency>
    <groupId>org.functionaljava</groupId>
    <artifactId>functionaljava-java-core</artifactId>
    <version>4.8.1</version>
</dependency>
```

## 4. 定义函数

让我们首先创建一个可以在稍后的示例中使用的函数。

如果没有Functional Java，基本的乘法方法看起来会像这样：

```java
public static final Integer timesTwoRegular(Integer i) {
    return i * 2;
}
```

使用Functional Java库，我们可以更优雅地定义此功能：

```java
public static final F<Integer, Integer> timesTwo = i -> i * 2;
```

**上面，我们看到一个F接口的例子，它以一个整数作为输入，并返回该整数乘以二作为其输出**。

下面是另一个基本函数的示例，它以整数作为输入，但在这种情况下，返回布尔值来指示输入是偶数还是奇数：

```java
public static final F<Integer, Boolean> isEven = i -> i % 2 == 0;
```

## 5. 应用函数

现在我们已经有了函数，让我们将它们应用到数据集中。

Functional Java库提供了一组常用的数据类型，例如列表、集合、数组和映射。需要注意的是，**这些数据类型是不可变的**。

此外，如果需要，该库还提供方便的函数来与标准Java集合类进行转换。

在下面的示例中，我们将定义一个整数列表并将timesTwo函数应用于它。我们还将使用同一函数的内联定义调用map。当然，我们希望结果相同：

```java
public void multiplyNumbers_givenIntList_returnTrue() {
    List<Integer> fList = List.list(1, 2, 3, 4);
    List<Integer> fList1 = fList.map(timesTwo);
    List<Integer> fList2 = fList.map(i -> i * 2);

    assertTrue(fList1.equals(fList2));
}
```

我们可以看到，map返回一个大小相同的列表，其中每个元素的值都是应用函数后的输入列表的值。输入列表本身不会改变。

以下是使用isEven函数的类似示例：

```java
public void calculateEvenNumbers_givenIntList_returnTrue() {
    List<Integer> fList = List.list(3, 4, 5, 6);
    List<Boolean> evenList = fList.map(isEven);
    List<Boolean> evenListTrueResult = List.list(false, true, false, true);

    assertTrue(evenList.equals(evenListTrueResult));
}
```

**由于map方法返回一个列表，我们可以将另一个函数应用于其输出**。调用map函数的顺序会改变最终的输出：

```java
public void applyMultipleFunctions_givenIntList_returnFalse() {
    List<Integer> fList = List.list(1, 2, 3, 4);
    List<Integer> fList1 = fList.map(timesTwo).map(plusOne);
    List<Integer> fList2 = fList.map(plusOne).map(timesTwo);

    assertFalse(fList1.equals(fList2));
}
```

上述列表的输出将是：

```text
List(3,5,7,9)
List(4,6,8,10)
```

## 6. 使用函数进行过滤

函数式编程中另一个常用的操作是**获取输入并根据某些条件过滤数据**。你可能已经猜到了，这些过滤条件是以函数的形式提供的。此函数需要返回一个布尔值来指示是否需要将数据包含在输出中。

现在，让我们使用isEven函数通过filter方法从输入数组中滤除奇数：

```java
public void filterList_givenIntList_returnResult() {
    Array<Integer> array = Array.array(3, 4, 5, 6);
    Array<Integer> filteredArray = array.filter(isEven);
    Array<Integer> result = Array.array(4, 6);

    assertTrue(filteredArray.equals(result));
}
```

一个有趣的观察是，在这个例子中，我们使用了数组而不是前面例子中的列表，我们的函数运行良好。由于函数的抽象和执行方式，它们不需要知道使用什么方法来收集输入和输出。

在这个例子中，我们还使用了我们自己的isEven函数，但是Functional Java自己的Integer类也具有用于基本数字比较的标准函数。

## 7. 使用函数应用布尔逻辑

在函数式编程中，我们经常使用这样的逻辑：“只有当所有元素都满足某些条件时才执行此操作”，或者“只有当至少一个元素满足某些条件时才执行此操作”。

**Functional Java库通过exist和forall方法为我们提供了这种逻辑的快捷方式**：

```java
public void checkForLowerCase_givenStringArray_returnResult() {
    Array<String> array = Array.array("Welcome", "To", "tuyucheng");
    assertTrue(array.exists(s -> List.fromString(s).forall(Characters.isLowerCase)));

    Array<String> array2 = Array.array("Welcome", "To", "Tuyucheng");
    assertFalse(array2.exists(s -> List.fromString(s).forall(Characters.isLowerCase)));

    assertFalse(array.forall(s -> List.fromString(s).forall(Characters.isLowerCase)));
}
```

在上面的例子中，我们使用了一个字符串数组作为输入。调用fromString函数会将数组中的每个字符串转换为字符列表，对于每个列表，我们都应用了forall(Characters.isLowerCase)。

你可能已经猜到了，Characters.isLowerCase是一个函数，如果字符为小写，则返回true。因此，将forall(Characters.isLowerCase)应用于字符列表时，只有当整个列表由小写字符组成时才会返回true，这反过来又表明原始字符串全部为小写。

在前两个测试中，我们使用了exists，因为我们只想知道是否至少有一个字符串是小写的。第三个测试使用forall来验证所有字符串是否都是小写的。

## 8. 使用函数处理可选值

在代码中处理可选值通常需要== null或isNotBlank检查。Java 8现在提供了Optional类来更优雅地处理这些检查，而Functional Java库提供了类似的构造，通过其Option类优雅地处理缺失数据：

```java
public void checkOptions_givenOptions_returnResult() {
    Option<Integer> n1 = Option.some(1);
    Option<Integer> n2 = Option.some(2);
    Option<Integer> n3 = Option.none();

    F<Integer, Option<Integer>> function = i -> i % 2 == 0 ? Option.some(i + 100) : Option.none();

    Option<Integer> result1 = n1.bind(function);
    Option<Integer> result2 = n2.bind(function);
    Option<Integer> result3 = n3.bind(function);

    assertEquals(Option.none(), result1);
    assertEquals(Option.some(102), result2);
    assertEquals(Option.none(), result3);
}
```

## 9. 使用函数归约集合

最后，我们将介绍归约集合的功能，“归约集合”是“将其汇总为一个值”的一种奇特说法。

**Functional Java库将此功能称为折叠**。

需要指定一个函数来指示折叠元素的含义。例如，Integers.add函数用于表示需要相加数组或列表中的整数。

根据函数折叠时执行的操作，结果可能会有所不同，具体取决于你是从右侧还是左侧开始折叠。这就是Functional Java库提供两个版本的原因：

```java
public void foldLeft_givenArray_returnResult() {
    Array<Integer> intArray = Array.array(17, 44, 67, 2, 22, 80, 1, 27);

    int sumAll = intArray.foldLeft(Integers.add, 0);
    assertEquals(260, sumAll);

    int sumEven = intArray.filter(isEven).foldLeft(Integers.add, 0);
    assertEquals(148, sumEven);
}
```

第一个foldLeft只是将所有整数相加。而第二个将首先应用过滤器，然后相加剩余的整数。

## 10. 总结

本文只是对Functional Java库进行简单介绍。