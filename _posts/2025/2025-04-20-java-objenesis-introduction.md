---
layout: post
title:  Objenesis简介
category: libraries
copyright: libraries
excerpt: Objenesis
---

## 1. 简介

**Java中的对象创建过程通常涉及构造函数的执行**，然而，在某些情况下，我们可能需要在不强制执行构造函数逻辑的情况下创建对象。

例如，**我们可能希望在[单元测试](https://www.baeldung.com/java-unit-testing-best-practices)期间Mock依赖关系，或在框架内管理对象生命周期**，其他示例包括处理具有复杂构造函数依赖关系的遗留代码或在反序列化的特定场景中。

为了帮助我们处理此类用例，**我们可以使用[Objenesis](https://objenesis.org/)，这是一个小型Java库，它允许我们从类实例化对象，而无需调用它们的构造函数**。它对于需要动态创建对象、完全绕过构造函数执行的库和框架非常有用。

在本教程中，我们将通过实际示例探索Objenesis的工作原理、如何在我们的项目中使用它以及它的实际应用。

## 2. Java中的传统对象创建

我们知道，当我们使用new关键字创建对象时，Java总是会执行其中一个构造函数。

让我们创建User类来说明Java中这个标准对象的创建：

```java
public class User {
    private String name;

    public User() {
        System.out.println("User constructor is called!");
    }

    // getters and setters
}
```

这里，我们在User类的默认构造函数中包含了一个打印语句，**使我们能够确认在创建User类的实例时调用了构造函数**：

```java
User user = new User();
```

执行此代码后，我们可以观察到控制台上打印了消息“User constructor is called!”，从而确认在创建User对象时确实使用了构造函数。

## 3. Objenesis工作原理

Objenesis不依赖于new关键字，也不依赖类的构造函数来创建对象。相反，**它使用低级JVM机制来分配内存并实例化对象，完全绕过构造函数**。

在内部，Objenesis会根据JVM供应商和版本尝试不同的实例化策略，直到成功。

### 3.1 设置

我们将最新的[org.objenesis](https://mvnrepository.com/artifact/org.objenesis/objenesis) Maven依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>org.objenesis</groupId>
    <artifactId>objenesis</artifactId>
    <version>3.4</version>
</dependency>
```

该库为[Objenesis](https://javadoc.io/static/org.objenesis/objenesis/3.4/org/objenesis/Objenesis.html)接口提供了两种主要实现：

- [ObjenesisStd](https://javadoc.io/static/org.objenesis/objenesis/3.4/org/objenesis/ObjenesisStd.html)：标准的线程安全实现，在创建对象时尝试不同的策略(依赖于JVM的供应商和版本)
- [ObjenesisSerializer](https://javadoc.io/static/org.objenesis/objenesis/3.4/org/objenesis/ObjenesisSerializer.html)：针对序列化框架优化的专门的非线程安全实现

此外，Objenesis提供了[ObjenesisHelper](https://javadoc.io/static/org.objenesis/objenesis/3.4/org/objenesis/ObjenesisHelper.html)实用程序类，可使用上述实现简化对象实例化。

### 3.2 使用ObjenesisStd创建对象

让我们修改我们的User类来完全限制构造函数的使用：

```java
public class User implements Serializable {
    private String name;

    public User() {
        throw new RuntimeException("User constructor should not be called!");
    }

    // getters and setters
}
```

在这里，我们在默认构造函数中抛出了一个RuntimeException，以确保它永远不会在我们的测试中被调用。

然后，让我们使用ObjenesisStd类创建User类的对象，而不调用构造函数：

```java
Objenesis objenesis = new ObjenesisStd();
User user = objenesis.newInstance(User.class);
assertNotNull(user);

user.setName("Harry Potter");
assertEquals("Harry Potter", user.getName());
```

这里，我们创建了ObjenesisStd类的实例，它是Objenesis的一个线程安全实现。然后，我们使用它来实例化User对象，而无需调用其构造函数。

assertNotNull()检查确保对象已成功创建。

为了确认对象行为正常，我们使用setName()方法设置用户的name，并使用getName()方法检索它，确认其属性和行为保持不变。

### 3.3 使用ObjenesisSerializer创建对象

类似地，我们可以利用ObjenesisSerializer类通过[基于序列化](https://www.baeldung.com/java-serialization)的实例化策略来创建User对象：

```java
Objenesis objenesis = new ObjenesisSerializer();
User user = objenesis.newInstance(User.class);
assertNotNull(user);

user.setName("Harry Potter");
assertEquals("Harry Potter", user.getName());
```

Objenesis接口的这个实现使用类似于反序列化的方法来创建新对象。

底层对象创建机制依赖于序列化原则，因此，**在使用ObjenesisSerializer创建对象时，应确保类实现Serializable接口**，否则，很可能会导致异常。

### 3.4 使用ObjenesisHelper创建对象

此外，**Objenesis库提供了ObjenesisHelper实用程序类，它使用标准和序列化策略简化了对象创建**。

我们来研究一下其标准策略的使用：

```java
User user = ObjenesisHelper.newInstance(User.class);
assertNotNull(user);

user.setName("Harry Potter");
assertEquals("Harry Potter", user.getName());
```

同样的，我们可以使用序列化策略：

```java
User user = ObjenesisHelper.newSerializableInstance(User.class);
assertNotNull(user);

user.setName("Harry Potter");
assertEquals("Harry Potter", user.getName());
```

因此，**ObjenesisHelper类提供了一种方便且首选的方式来使用Objenesis，因为它封装了针对常见用例的ObjenesisStd或ObjenesisSerializer实例的显式创建**。

## 4. 高级用例

Objenesis支持许多高级用例，尤其是在标准对象创建不可行时。

**序列化框架(例如[Kryo](https://www.baeldung.com/kryo))在反序列化过程中内部使用Objenesis创建对象，而无需调用其构造函数**，这对于性能以及处理序列化数据与构造函数要求不一致的情况至关重要。

**[诸如Mockito](https://www.baeldung.com/mockito-series)和[EasyMock](https://www.baeldung.com/easymock)之类的Mock框架，通常会在内部使用Objenesis来创建Mock对象或代理，以用于测试目的**。当我们创建一个类的Mock对象时，Mock框架需要实例化一个代理对象，该代理对象可以拦截方法调用并记录交互。Objenesis提供了一种简洁的方法来创建这些代理实例，而无需触发原始类中可能很复杂或依赖关系繁重的构造函数。

通常，final类或具有私有构造函数的类很难通过反射实例化。**Objenesis绕过了这一限制，即使构造函数是私有的或类是final的，也能实现实例化**。

**一些轻量级或定制的[依赖注入](https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring#what-is-dependency-injection)框架可能会使用Objenesis来实例化Bean或服务类**，其中[构造函数注入](https://www.baeldung.com/constructor-injection-in-spring)不切实际或不可取，允许基于属性的注入而不调用构造函数。

在性能至关重要的应用程序中，我们可能希望克隆或重用对象，而无需承担完全重新初始化的成本。**Objenesis可以帮助我们创建一个新的实例，并手动填充，从而跳过昂贵的构造函数逻辑**。

## 5. 最佳实践

Objenesis提供了一种便捷的对象实例化方法，无需使用构造函数。然而，建议谨慎使用并理解其含义。

**过度使用或误用可能会导致意外行为和可维护性挑战**，让我们看看一些推荐的最佳实践：

- 优先使用构造函数：我们应该利用构造函数来创建常规对象，并且仅在必要时使用Objenesis来绕过构造函数调用
- 我们应该注意安全限制：Java安全管理器或容器化应用程序等环境可能会由于Objenesis的低级对象实例化机制而阻止它
- 利用框架与Objenesis的集成：当使用Mockito或Kryo等框架时，我们应该让它们在内部处理Objenesis，而不是直接调用它
- 未初始化对象的处理：由于Objenesis跳过了构造函数逻辑，因此我们在处理默认初始化字段时应更加小心，因为如果处理不当，它们可能会导致意外行为
- 测试实例化对象：我们必须确保使用Objenesis创建的对象经过彻底测试，以确认所有属性和行为保持完整
- 清晰地记录其用法：我们应该记录Objenesis的使用方法和相关的对象初始化策略，以提高其他开发人员的可维护性

## 6. 总结

在本文中，我们探讨了Objenesis库，它允许我们在不调用构造函数的情况下创建对象，这使其对于序列化、Mock和代理框架很有用。

我们探索了它的工作原理，在项目中进行了设置，并通过实际示例了解了它的功能。但是，尽管Objenesis功能强大，但我们应该仅在必要时使用它，以避免出现意外的副作用。此外，我们还研究了一些高级用例和最佳实践。