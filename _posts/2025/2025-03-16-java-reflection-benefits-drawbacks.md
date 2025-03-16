---
layout: post
title:  Java反射是不好的做法吗？
category: java-reflect
copyright: java-reflect
excerpt: Java反射
---

## 1. 概述

**长期以来，使用反射API在Java社区中引发了广泛的争论，有时被视为一种不好的做法**。虽然它被流行的Java框架和库广泛使用，但它的潜在缺点阻碍了它在常规服务器端应用程序中的频繁使用。

在本教程中，我们将深入探讨反射可能给我们的代码库带来的好处和缺点。此外，**我们将探讨何时适合或不适合使用反射，最终帮助我们确定它是否是一种不好的做法**。

## 2. 理解Java反射

**在计算机科学中，反射编程或反射是指进程检查、自省和修改其结构和行为的能力**。当编程语言完全支持反射时，它允许在运行时检查和修改代码库中类和对象的结构和行为，从而允许源代码重写自身的某些方面。

根据这个定义，Java完全支持反射。除了Java，其他支持反射编程的常见编程语言还有C#、Python和JavaScript。

许多流行的Java框架(如[Spring](https://www.baeldung.com/spring-tutorial)和[Hibernate](https://www.baeldung.com/learn-jpa-hibernate))都依赖它来提供高级功能，如依赖注入、面向切面编程和数据库映射。除了通过框架或库间接使用反射外，我们还可以借助[java.lang.reflect](https://www.baeldung.com/java-reflection)包或[Reflections](https://www.baeldung.com/reflections-library)库直接使用它。

## 3. Java反射的优点

**如果使用得当，Java反射可以成为一种强大而多功能的功能**。在本节中，我们将探讨反射的一些主要优势以及如何在某些情况下有效地使用它。

### 3.1 动态配置

**反射API支持动态编程，增强应用程序的灵活性和适应性**。当我们遇到直到运行时才知道所需类或模块的情况时，这一方面非常有用。

此外，通过利用反射的动态功能，开发人员可以构建可实时轻松重新配置的系统，而无需进行大量的代码更改。

**例如，Spring框架使用反射来创建和配置Bean**。它扫描类路径组件并根据注解和XML配置动态实例化和配置Bean，允许开发人员在不更改源代码的情况下添加或修改Bean。

### 3.2 可扩展性

**使用反射的另一个重要优势是可扩展性，这使我们能够在运行时合并新功能或模块，而无需更改应用程序的核心代码**。

为了说明这一点，假设我们正在使用一个第三方库，该库定义了一个基类并包含多个子类型以实现[多态反序列化](https://www.baeldung.com/java-jackson-polymorphic-deserialization)，我们希望通过引入扩展相同基类的自定义子类型来扩展功能。对于这种特定用例，反射API非常有用，因为我们可以利用它在运行时动态注册这些自定义子类型，并轻松地将它们与第三方库集成。**因此，我们可以使库适应我们的特定需求，而无需更改其代码库**。

### 3.3 代码分析

**反射的另一个用例是代码分析，它允许我们动态检查代码**。这特别有用，因为它可以提高软件开发的质量。

例如，用于架构单元测试的Java库[ArchUnit](https://www.baeldung.com/java-archunit-intro)就利用了反射和字节码分析。库无法通过反射API获取的信息将在字节码级别获取。这样，库就可以动态分析代码，我们就可以执行架构规则和约束，从而确保软件项目的完整性和高质量。

## 4. Java反射的缺点

正如我们在上一节中看到的，反射是一种功能强大的功能，具有多种应用。**然而，它有一系列的缺点，我们在决定在项目中使用它之前需要考虑这些缺点**。在本节中，我们将深入探讨此功能的一些主要缺点。

### 4.1 性能开销

**Java反射会动态解析类型，并可能限制某些JVM优化**。因此，反射操作的性能比非反射操作要慢。因此，在处理性能敏感的应用程序时，我们应该考虑避免在代码中经常调用的部分使用反射。

为了演示这一点，我们将创建一个非常简单的Person类并对其执行一些反射和非反射操作：

```java
public class Person {

    private String firstName;
    private String lastName;
    private Integer age;

    public Person(String firstName, String lastName, Integer age) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.age = age;
    }

    // standard getters and setters
}
```

现在，我们可以创建一个基准测试，以便查看调用我们类的Getter的时间差异：

```java
public class MethodInvocationBenchmark {

    @Benchmark
    @Fork(value = 1, warmups = 1)
    @OutputTimeUnit(TimeUnit.NANOSECONDS)
    @BenchmarkMode(Mode.AverageTime)
    public void directCall(Blackhole blackhole) {
        Person person = new Person("John", "Doe", 50);

        blackhole.consume(person.getFirstName());
        blackhole.consume(person.getLastName());
        blackhole.consume(person.getAge());
    }

    @Benchmark
    @Fork(value = 1, warmups = 1)
    @OutputTimeUnit(TimeUnit.NANOSECONDS)
    @BenchmarkMode(Mode.AverageTime)
    public void reflectiveCall(Blackhole blackhole) throws InvocationTargetException, NoSuchMethodException, IllegalAccessException {
        Person person = new Person("John", "Doe", 50);

        Method getFirstNameMethod = Person.class.getMethod("getFirstName");
        blackhole.consume(getFirstNameMethod.invoke(person));

        Method getLastNameMethod = Person.class.getMethod("getLastName");
        blackhole.consume(getLastNameMethod.invoke(person));

        Method getAgeMethod = Person.class.getMethod("getAge");
        blackhole.consume(getAgeMethod.invoke(person));
    }
}
```

让我们检查一下运行方法调用基准测试的结果：

```text
Benchmark                                 Mode  Cnt    Score   Error  Units
MethodInvocationBenchmark.directCall      avgt    5    8.428 ± 0.365  ns/op
MethodInvocationBenchmark.reflectiveCall  avgt    5  102.785 ± 2.493  ns/op
```

现在，让我们创建另一个基准测试来测试反射初始化与直接调用构造函数相比的性能：

```java
public class InitializationBenchmark {

    @Benchmark
    @Fork(value = 1, warmups = 1)
    @OutputTimeUnit(TimeUnit.NANOSECONDS)
    @BenchmarkMode(Mode.AverageTime)
    public void directInit(Blackhole blackhole) {
        blackhole.consume(new Person("John", "Doe", 50));
    }

    @Benchmark
    @Fork(value = 1, warmups = 1)
    @OutputTimeUnit(TimeUnit.NANOSECONDS)
    @BenchmarkMode(Mode.AverageTime)
    public void reflectiveInit(Blackhole blackhole) throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
        Constructor<Person> constructor = Person.class.getDeclaredConstructor(String.class, String.class, Integer.class);
        blackhole.consume(constructor.newInstance("John", "Doe", 50));
    }
}
```

让我们检查构造函数调用的结果：

```text
Benchmark                                 Mode  Cnt    Score   Error  Units
InitializationBenchmark.directInit        avgt    5    5.290 ± 0.395  ns/op
InitializationBenchmark.reflectiveInit    avgt    5   23.331 ± 0.141  ns/op
```

在查看了两个基准测试的结果后，我们可以合理地推断，在Java中使用反射对于调用方法或初始化对象等用例来说可能会慢得多。

我们的文章[使用Java进行微基准测试](https://www.baeldung.com/java-microbenchmark-harness)包含更多有关我们用于比较执行时间的信息。

### 4.2 内部结构暴露

**反射允许在非反射代码中可能受到限制的操作**，一个很好的例子是访问和操作类的私有字段和方法的能力。通过这样做，我们违反了封装，这是面向对象编程的基本原则。

作为示例，让我们创建一个仅具有一个私有字段的虚拟类，而不创建任何Getter或Setter：

```java
public class MyClass {
    
    private String veryPrivateField;
    
    public MyClass() {
        this.veryPrivateField = "Secret Information";
    }
}
```

现在，让我们尝试在单元测试中访问这个私有字段：

```java
@Test
public void givenPrivateField_whenUsingReflection_thenIsAccessible() throws IllegalAccessException, NoSuchFieldException {
      MyClass myClassInstance = new MyClass();

      Field privateField = MyClass.class.getDeclaredField("veryPrivateField");
      privateField.setAccessible(true);

      String accessedField = privateField.get(myClassInstance).toString();
      assertEquals(accessedField, "Secret Information");
}
```

### 4.3 编译时安全性的丧失

**反射的另一个缺点是编译时安全性的丧失**。在典型的Java开发中，编译器会执行严格的类型检查，并确保我们正确使用类、方法和字段。然而，反射会绕过这些检查，因此，某些错误直到运行时才可发现。因此，这可能会导致难以检测的错误，并可能损害我们代码库的可靠性。

### 4.4 代码可维护性下降

**使用反射会显著降低代码的可维护性**，严重依赖反射的代码往往比非反射代码的可读性更差。可读性降低会导致维护困难，因为开发人员更难理解代码的意图和功能。

**另一个挑战是工具支持有限**，并非所有开发工具和IDE都完全支持反射，因此，这会减慢开发速度并使其更容易出错，因为开发人员必须依靠手动检查来发现问题。

### 4.5 安全问题

Java反射涉及访问和操作程序的内部元素，这可能会引起安全问题。在受限环境中，**允许反射访问可能会带来风险，因为恶意代码可能会尝试利用反射来获取对敏感资源的未经授权的访问或执行违反安全策略的操作**。

## 5. Java 9对反射的影响

**Java 9中引入的[模块](https://www.baeldung.com/java-9-modularity)对模块封装代码的方式带来了重大变化，在Java 9之前，使用反射很容易破坏封装**。

默认情况下，模块不再公开其内部内容。但是，Java 9提供了一些机制来选择性地授予模块之间反射访问的权限。这使我们能够在必要时打开特定的包，确保与旧代码或第三方库的兼容性。

## 6. 何时应该使用Java反射？

在探索了反射的优点和缺点之后，我们可以确定何时适合或不适合使用此强大功能的一些用例。

**在动态行为至关重要的情况下，使用反射API非常有用**。正如我们已经看到的，许多知名的框架和库(如Spring和Hibernate)都依赖它来实现关键功能。在这些情况下，反射使这些框架能够为开发人员提供灵活性和定制性。此外，当我们自己创建库或框架时，反射可以使其他开发人员扩展和定制他们与我们代码的交互，使其成为合适的选择。

**此外，反射可以成为扩展我们无法修改的代码的一种选择**。因此，当我们使用第三方库或遗留代码并需要集成新功能或调整现有功能而不改变原始代码库时，它可以成为一个强大的工具。它允许我们访问和操作原本无法访问的元素，使其成为此类场景的实用选择。

但是，在考虑使用反射时务必谨慎。**在对安全性要求较高的应用程序中，应谨慎使用反射代码**。反射允许访问程序的内部元素，而这些元素可能会被恶意代码利用。此外，在处理性能至关重要的应用程序时，特别是在经常调用的代码部分中，反射的性能开销可能成为一个问题。此外，如果编译时类型检查对我们的项目至关重要，我们应该考虑避免使用反射代码，因为它缺乏编译时安全性。

## 7. 总结

正如我们在这篇文章中了解到的，Java中的反射应该被视为一种需要谨慎使用的强大工具，而不是被贴上坏习惯的标签。与任何功能类似，过度使用反射确实可以被视为一种坏习惯。然而，如果谨慎使用并且只在真正必要时使用，反射可以成为一种宝贵的资产。