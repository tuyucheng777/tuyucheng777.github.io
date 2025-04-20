---
layout: post
title:  Java中的可变对象与不可变对象
category: designpattern
copyright: designpattern
excerpt: 不可变对象
---

## 1. 简介

在Java中使用对象时，理解可变对象和不可变对象之间的区别至关重要，**这些概念会影响Java代码的行为和设计**。

在本教程中，让我们探讨可变和不可变对象的定义、示例、优点和注意事项。

## 2. 不可变对象

**不可变对象是指一旦创建，其状态就无法更改的对象**。不可变对象一旦实例化，其值和属性在其整个生命周期内保持不变。

让我们探索Java中内置不可变类的一些示例。

### 2.1 String类

Java中[String](https://www.baeldung.com/java-string)的不变性保证了线程安全，增强了安全性，并通过[字符串池](https://www.baeldung.com/java-string-pool)机制帮助高效利用内存。

```java
@Test
public void givenImmutableString_whenConcatString_thenNotSameAndCorrectValues() {
    String originalString = "Hello";
    String modifiedString = originalString.concat(" World");

    assertNotSame(originalString, modifiedString);

    assertEquals("Hello", originalString);
    assertEquals("Hello World", modifiedString);
}
```

在此示例中，concat()方法创建一个新的String，而原始String保持不变。

### 2.2 Integer类

在Java中，[Integer](https://www.baeldung.com/java-number-of-digits-in-int)类是不可变的，这意味着它的值一旦设置就无法更改。**但是，当你对Integer执行操作时，会创建一个新的实例来保存结果**。

```java
@Test
public void givenImmutableInteger_whenAddInteger_thenNotSameAndCorrectValue() {
    Integer immutableInt = 42;
    Integer modifiedInt = immutableInt + 8;

    assertNotSame(immutableInt, modifiedInt);

    assertEquals(42, (int) immutableInt);
    assertEquals(50, (int) modifiedInt);
}
```

这里，+操作创建了一个新的Integer对象，而原始对象保持不变。

### 2.3 不可变对象的优点

Java中的不可变对象具有多种优势，有助于提高代码的可靠性、简洁性和性能，让我们了解一下使用不可变对象的一些好处：

- **[线程安全](https://www.baeldung.com/java-thread-safety)**：不可变性本质上确保了线程安全，由于不可变对象的状态在创建后无法修改，因此它可以在多个线程之间安全地共享，而无需显式同步，这简化了并发编程并降低了竞争条件的风险。
- **可预测性和调试性**：不可变对象的恒定状态使代码更加可预测，不可变对象的值一旦创建便保持不变，从而简化了对代码行为的推理。
- **方便缓存和优化**：不可变对象可以轻松缓存和重用，不可变对象一旦创建，其状态就不会改变，从而支持高效的缓存策略。

因此，开发人员可以在Java应用程序中使用不可变对象设计更强大、可预测和高效的系统。

## 3. 创建不可变对象

为了创建一个不可变的对象，我们以一个名为ImmutablePerson的类为例，该类被声明为final以防止扩展，并且它包含private final字段，没有Setter方法，遵循不变性原则。

```java
public final class ImmutablePerson {
    private final String name;
    private final int age;

    public ImmutablePerson(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }
}
```

现在，让我们考虑一下当我们尝试修改ImmutablePerson实例的名称时会发生什么：

```java
ImmutablePerson person = new ImmutablePerson("John", 30);
person.setName("Jane");
```

尝试修改ImmutablePerson实例的名称将导致编译错误，这是因为该类被设计为不可变的，实例化后没有Setter方法允许更改其状态。

**没有Setter并且将类声明为final，确保了对象的不变性，从而提供了一种清晰而强大的方法来处理其整个生命周期中的常量状态**。

## 4. 可变对象

**Java中的可变对象是指创建后状态可以修改的实体**，这种可变性引入了可变内部数据的概念，允许在对象的生命周期内修改其值和属性。

让我们探讨几个例子来了解它们的特点。

### 4.1 StringBuilder类

Java中的[StringBuilder](https://www.baeldung.com/java-string-builder-string-buffer)类表示可变的字符序列，与不可变的String不同，StringBuilder允许动态修改其内容。

```java
@Test
public void givenMutableString_whenAppendElement_thenCorrectValue() {
    StringBuilder mutableString = new StringBuilder("Hello");
    mutableString.append(" World");

    assertEquals("Hello World", mutableString.toString());
}
```

这里，append方法直接改变了StringBuilder对象的内部状态，展示了它的可变性。

### 4.2 ArrayList类

[ArrayList](https://www.baeldung.com/java-arraylist)类是可变对象的另一个示例，它表示一个可以增大或缩小的动态数组，允许添加或删除元素。

```java
@Test
public void givenMutableList_whenAddElement_thenCorrectSize() {
    List<String> mutableList = new ArrayList<>();
    mutableList.add("Java");

    assertEquals(1, mutableList.size());
}
```

add方法通过添加元素来修改ArrayList的状态，体现了其可变性。

### 4.3 注意事项

虽然可变对象提供了灵活性，但开发人员需要注意以下几点：

- **线程安全**：可变对象可能需要额外的同步机制来确保多线程环境中的线程安全，如果没有适当的同步，并发修改可能会导致意外行为。
- **代码理解的复杂性**：修改可变对象内部状态的能力增加了代码理解的复杂性，开发人员需要谨慎对待对象状态的潜在变化，尤其是在大型代码库中。
- **状态管理挑战**：管理可变对象的内部状态需要谨慎考虑，开发人员应该跟踪和控制变更，以确保对象的完整性并防止意外修改。

尽管存在这些考虑，可变对象提供了一种动态且灵活的方法，允许开发人员根据不断变化的需求调整对象的状态。

## 5. 可变对象与不可变对象

在对比可变对象和不可变对象时，需要考虑几个因素，让我们来探讨一下这两种对象之间的根本区别：

|     标准|          可变对象|        不可变对象|
| :----------: | :------------------------: | :----------------------: |
| 可修改性|       创建后可以更改|   一旦创建，就保持不变|
| 线程安全| 可能需要同步以确保线程安全|       固有线程安全|
| 可预测性|   可能会增加理解的复杂性|      简化推理和调试|
| 性能影响|   可能会因同步而影响性能| 通常会对绩效产生积极影响|

### 5.1 在可变性和不可变性之间进行选择

可变性和不可变性的选择取决于应用程序的需求，**如果需要适应性和频繁更改，则选择可变对象。但是，如果一致性、安全性和稳定状态是优先考虑的，则选择不可变性**。

考虑多任务场景中的并发性，**不变性简化了任务之间的数据共享，而无需复杂的同步**。

此外，请评估应用程序的性能需求。虽然不可变对象通常可以提高性能，但请权衡这种提升是否比可变对象提供的灵活性更重要，尤其是在数据不频繁更改的情况下。

保持适当的平衡可确保你的代码有效地满足应用程序的需求。

## 6. 总结

总而言之，Java中可变对象和不可变对象的选择对于代码的可靠性、效率和可维护性至关重要。**不可变性提供了线程安全性、可预测性和其他优势，而可变性则提供了灵活性和动态状态变化**。

评估应用程序的需求并考虑并发性、性能和代码复杂性等因素将有助于做出适当的选择，以设计具有弹性且高效的Java应用程序。