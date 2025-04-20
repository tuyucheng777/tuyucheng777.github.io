---
layout: post
title:  Java中的清洁代码
category: designpattern
copyright: designpattern
excerpt: 清洁代码
---

## 1. 概述

在本教程中，我们将学习整洁代码的原则。我们还将了解为什么整洁代码如此重要，以及如何在Java中实现这一点。此外，我们还会看看是否有可用的工具可以帮助我们。

## 2. 什么是清洁代码？

因此，在深入探讨“整洁代码”的细节之前，我们先来理解一下“整洁代码”的含义。说实话，这个问题没有一个统一的答案。在编程中，有些关注点是相互关联的，因此会形成一些通用的原则。但是，每种编程语言和范式都有其自身的细微差别，这要求我们采用合适的做法。

广义上讲，**整洁代码可以概括为任何开发人员都能轻松阅读和修改的代码**，虽然这听起来可能过于简化了概念，但我们稍后会在本教程中看到它是如何构建的。每当我们听到“整洁代码”这个词时，或许都会提到Martin Fowler，以下是他在某处对整洁代码的描述：

> 即使是傻瓜也能写出计算机能理解的代码，优秀的程序员能写出人类能理解的代码。

## 3. 为什么我们应该关心干净的代码？

编写简洁的代码既关乎个人习惯，也关乎技能。作为一名开发者，我们随着时间的流逝，经验和知识不断积累。但是，我们必须问自己，为什么要投入精力开发简洁的代码？我们知道，其他人可能会觉得我们的代码更容易阅读，但这真的足够激励我们吗？让我们来一探究竟！

整洁编码原则帮助我们实现许多与软件开发相关的理想目标，让我们来仔细研究一下，以便更好地理解：

- 可维护的代码库：我们开发的任何软件都有其生命周期，在此期间需要进行更改和常规维护，**简洁的代码有助于开发易于更改和维护的软件**。
- 更轻松的故障排除：软件可能会因各种内部或外部因素而出现意外行为，通常需要快速修复和可用性，**采用清晰编码原则开发的软件更容易解决问题**。
- 更快的上手体验：软件在其生命周期中会经历许多开发人员的创建、更新和维护，并且开发人员会在不同的时间点加入，**这需要更快的上手体验以保持高生产力**，而简洁的代码有助于实现这一目标。

## 4. 整洁代码的特点

遵循整洁编码原则编写的代码库具有一些与众不同的特征，让我们来看看其中一些特征：

- 专注：**一段代码应该用于解决特定问题**，它不应该做任何与解决特定问题严格无关的事情。这适用于代码库中的所有抽象级别，例如方法、类、包或模块。
- 简单：这是迄今为止最重要却又经常被忽视的清洁代码特性，**软件设计和实现必须尽可能简单**，才能帮助我们实现预期结果。代码库的复杂性不断增加，使其容易出错，难以阅读和维护。
- 可测试：简洁的代码虽然简单，但必须能够解决当前的问题。**它必须直观且易于测试代码库**，最好以自动化的方式进行。这有助于建立代码库的基线行为，并使更改代码库变得更容易，而不会破坏任何功能。

这些有助于我们实现上一节讨论的目标，相比事后重构，在开发之初就牢记这些特性是有益的，这将降低软件生命周期的总体拥有成本。

## 5. Java清洁代码

既然我们已经了解了足够的背景知识，让我们看看如何在Java中融入简洁代码原则。Java提供了许多最佳实践来帮助我们编写简洁的代码，我们将把它们分为不同的类别，并通过代码示例来了解如何编写简洁的代码。

### 5.1 项目结构

**虽然Java不强制任何项目结构，但遵循一致的模式来组织我们的源文件、测试、配置、数据和其他代码构件总是有益的**。Maven是一款流行的Java构建工具，它规定了一种特定的[项目结构](http://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html)。虽然我们可能不会使用Maven，但遵循惯例总是好的。

让我们看看Maven建议我们创建的一些文件夹：

- src/main/java：用于源文件
- src/main/resources：用于资源文件，例如properties
- src/test/java：用于测试源文件
- src/test/resources：用于测试资源文件，例如properties

与此类似，还有[其他流行的项目结构](https://www.baeldung.com/bazel-build-tool)，例如建议用于Java的Bazel，我们应该根据我们的需求和受众选择一种。

### 5.2 命名约定

**遵循命名约定对于提高代码的可读性和可维护性大有帮助**，Spring的创建者Rod Johnson强调了Spring中命名约定的重要性：

> “...如果你知道某个东西的作用，你就有很大机会猜出它的Spring类或接口的名称...”

Java[规定](https://www.oracle.com/technetwork/java/codeconventions-135099.html)了一套在命名任何对象时必须遵循的规则，一个格式良好的名称不仅有助于阅读代码，还能传达出很多关于代码意图的信息。我们来举几个例子：

- 类：在面向对象的概念中，类是对象的蓝图，通常代表现实世界中的对象。因此，使用名词来命名能够充分描述它们的类是有意义的：

  ```java
  public class Customer {
  }
  ```

- 变量：Java中的变量捕获从类创建的对象的状态，变量的名称应该清楚地描述变量的用途：

  ```java
  public class Customer {
      private String customerName;
  }
  ```

- 方法：Java中的方法始终是类的一部分，因此通常表示对由该类创建的对象的状态执行的操作。因此，[使用动词来命名方法](https://www.baeldung.com/java-pojo-class#javabeans)很有用：

  ```java
  public class Customer {
      private String customerName;
      public String getCustomerName() {
          return this.customerName;
      }
  }
  ```

虽然我们只讨论了如何在Java中命名标识符，但请注意，还有其他一些最佳实践，例如驼峰式大小写，为了提高可读性，我们应该遵循这些最佳实践。此外，还有更多与接口、枚举和常量命名相关的约定。

### 5.3 源文件结构

源文件可以包含不同的元素，虽然**Java编译器会强制执行某些结构**，但很大一部分结构是可变的。但是，遵循源文件中元素的特定顺序可以显著提高代码的可读性。有很多流行的代码风格指南可供参考，例如[Google](https://google.github.io/styleguide/javaguide.html)和[Spring](https://github.com/spring-projects/spring-framework/wiki/Code-Style)的指南。

让我们看看源文件中元素的典型顺序应该是什么样的：

- 包声明
- 导入语句
  - 所有静态导入
  - 所有非静态导入
- 只有一个顶级类
  - 类变量
  - 实例变量
  - 构造函数
  - 方法

除上述之外，**方法可以根据其功能或范围进行分组**。没有一个好的惯例，想法应该一旦确定，**然后始终如一地遵循**。

让我们看一个格式良好的源文件：

```java
# /src/main/java/cn/tuyucheng/taketoday/application/entity/Customer.java
package cn.tuyucheng.taketoday.application.entity;

import java.util.Date;

public class Customer {
    private String customerName;
    private Date joiningDate;
    public Customer(String customerName) {
        this.customerName = customerName;
        this.joiningDate = new Date();
    }

    public String getCustomerName() { 
        return this.customerName; 
    }

    public Date getJoiningDate() {
        return this.joiningDate;
    }
}
```

### 5.4 空行

我们都知道，与大段文字相比，短段落更容易阅读和理解。阅读代码也是如此，位置合理且一致的空格和空行可以增强代码的可读性。

这里的想法是在代码中引入逻辑分组，这有助于在阅读代码时组织思维过程。这里没有单一的规则，而是一套通用的指导方针，以及以可读性为核心的内在意图：

- 静态块、字段、构造函数和内部类开始前的两个空行
- 多行方法签名后有一个空行
- 用一个空格将保留关键字(如if、for、catch)与左括号分开
- 保留关键字(例如else、catch)与右括号之间用一个空格隔开

这里的列表并不详尽，但应该能给我们指明方向。

### 5.5 缩进

虽然这很琐碎，但几乎所有开发人员都会确信，**良好的缩进代码更易于阅读和理解**。Java中没有统一的代码缩进约定，关键在于要么采用一种流行的约定，要么定义一个私有的约定，并在整个组织内统一遵循。

让我们看看一些重要的缩进标准：

- 典型的最佳实践是使用4个空格作为缩进单位，请注意，有些指南建议使用制表符而不是空格。虽然这里没有绝对的最佳实践，但关键在于保持一致性！
- 通常，行长应该有一个上限，但由于如今开发人员使用的屏幕更大，这个上限可以设置得高于传统的80。
- 最后，由于许多表达式无法放在一行中，因此我们必须一致地断开它们：
  - 在逗号后中断方法调用
  - 在运算符之前中断表达式
  - 缩进换行以提高可读性

让我们看一个例子：

```java
List<String> customerIds = customer.stream()
    .map(customer -> customer.getCustomerId())
    .collect(Collectors.toCollection(ArrayList::new));
```

### 5.6 方法参数

参数对于方法按规范工作至关重要，但是，**冗长的参数列表可能会使人们难以阅读和理解代码**。那么，我们应该在哪里划清界限呢？让我们了解一些可能对我们有帮助的最佳实践：

- 尝试限制方法接收的参数数量，3个参数是一个不错的选择
- 如果方法需要的参数超过推荐参数，请考虑[重构](https://www.baeldung.com/cs/refactoring)该方法，通常较长的参数列表也表明该方法可能正在执行多项操作
- 我们可以考虑将参数捆绑到自定义类型中，但必须小心，不要将不相关的参数转储到单个类型中
- 最后，虽然我们应该用这个建议来判断代码的可读性，但我们不能对此吹毛求疵

让我们来看一个例子：

```java
public boolean setCustomerAddress(String firstName, String lastName, String streetAddress, 
  String city, String zipCode, String state, String country, String phoneNumber) {
}

// This can be refactored as below to increase readability

public boolean setCustomerAddress(Address address) {
}
```

### 5.7 硬编码

代码中的硬编码值通常会导致多种副作用，例如，**它会导致重复，从而使更改更加困难**。如果值需要动态更改，通常会导致不良行为。在大多数情况下，可以通过以下方式之一重构硬编码值：

- 考虑用Java中定义的常量或枚举替换
- 或者，用在类级别或单独的类文件中定义的常量替换
- 如果可能，请用可以从配置或环境中选择的值替换

让我们看一个例子：

```java
private int storeClosureDay = 7;

// This can be refactored to use a constant from Java

private int storeClosureDay = DayOfWeek.SUNDAY.getValue()
```

再次强调，这方面并没有严格的指导原则需要遵循。但我们必须意识到，有些人以后需要阅读和维护这段代码，我们应该选择一个适合自己的约定，并坚持下去。

### 5.8 代码注释

阅读代码时，**[代码注释](https://www.baeldung.com/cs/clean-code-comments)有助于理解重要的方面**。同时，应注意不要在注释中包含显而易见的内容，这会使注释变得臃肿，难以阅读相关部分。

Java允许两种类型的注释：实现注释和文档注释，它们有不同的用途和格式，让我们更好地理解它们：

- 文档/JavaDoc注释
  - 这里的受众是代码库的用户
  - 这里的细节通常与实现无关，更多地关注规范
  - 通常独立于代码库使用
- 实现/块注释
  - 这里的受众是从事代码库工作的开发人员
  - 这里的细节是具体实现的
  - 通常与代码库一起使用

那么，我们应该如何最佳地使用它们，才能让它们变得有用并且符合上下文呢？

- 注释应该只是对代码的补充，如果我们无法理解没有注释的代码，也许我们需要重构它
- 我们应该很少使用块注释，可能是为了描述非平凡的设计决策
- 我们应该对大多数类、接口、公共和受保护的方法使用JavaDoc注释
- 所有注释都应格式正确，并有适当的缩进以提高可读性

让我们看一个有意义的文档注释的例子：

```java
/**
* This method is intended to add a new address for the customer.
* However do note that it only allows a single address per zip
* code. Hence, this will override any previous address with the
* same postal code.
*
* @param address an address to be added for an existing customer
*/
/*
* This method makes use of the custom implementation of equals 
* method to avoid duplication of an address with same zip code.
*/
public addCustomerAddress(Address address) {
}
```

### 5.9 日志记录

任何曾经接触过生产代码进行调试的人，都曾渴望看到更多日志。**日志在开发过程中，尤其是在维护过程中，其重要性怎么强调也不为过**。

Java中有很多用于日志记录的库和框架，包括SLF4J和Logback。虽然它们使代码库中的日志记录变得非常简单，但必须谨慎考虑日志记录的最佳实践，不合理的日志记录可能会成为维护的噩梦，而不是任何帮助。让我们来看看其中的一些最佳实践：

- 避免过多的日志记录，思考哪些信息可能有助于故障排除
- 明智地选择日志级别，我们可能希望在生产环境中有选择地启用日志级别
- 日志消息中的上下文数据要非常清晰且具有描述性
- 使用外部工具跟踪、聚合、过滤日志消息，以便更快地进行分析

让我们看一个具有正确级别的描述性日志记录的示例：

```java
logger.info(String.format("A new customer has been created with customer Id: %s", id));
```

## 6. 就这些吗？

虽然上一节重点介绍了几种代码格式约定，但这些并不是我们应该了解和关注的全部，一段可读且可维护的代码可以从长期积累的大量其他最佳实践中受益。

随着时间的推移，我们可能已经将它们视为有趣的首字母缩略词，**它们本质上是将这些经验总结为一条或一组可以帮助我们编写更好代码的原则**。但是，请注意，我们不应该仅仅因为它们存在就全部遵循。大多数情况下，它们带来的好处与代码库的大小和复杂性成正比。在采用任何原则之前，我们必须先访问我们的代码库。更重要的是，我们必须与它们保持一致。

### 6.1 SOLID

SOLID是一个助记缩写，它源于编写可理解和可维护的软件所提出的[五项原则](https://www.baeldung.com/solid-principles)：

- 单一职责原则：我们定义的**每个接口、类或方法都应该有一个明确的目标**。本质上，它应该只做一件事，并且做好它。这可以有效地减少方法和类的体积，同时也便于测试。
- 开闭原则：理想情况下，我们编写的代码应该**对扩展开放，但对修改关闭**。这实际上意味着，类应该以无需修改的方式编写。但是，它应该允许通过继承或组合进行更改。
- [里氏替换原则](https://www.baeldung.com/cs/liskov-substitution-principle)：该原则指出，**每个子类或派生类都应该可以替换其父类或基类**。这有助于减少代码库中的耦合度，从而提高可重用性。
- 接口隔离原则：实现接口是为类提供特定行为的一种方式，但是，**类不必实现它不需要的方法**。这要求我们定义更小、更专注的接口。
- 依赖倒置原则：根据此原则，**类应该只依赖于抽象，而不依赖于其具体实现**。这实际上意味着类不应该负责为其依赖项创建实例，相反，这些依赖项应该被注入到类中。

### 6.2 DRY和KISS

[DRY](https://www.baeldung.com/cs/dry-software-design-principle)代表“不要重复自己”，该原则指出，**一段代码不应在整个软件中重复**。该原则背后的原理是减少重复并提高可重用性，但是，请注意，我们在执行这一原则时应谨慎，不要过于字面化，适当的重复实际上可以提高代码的可读性和可维护性。

[KISS](https://www.baeldung.com/cs/kiss-software-design-principle)代表“保持简单，精简”，这条原则指出，**我们应该尽量保持代码简洁**，这样，代码才能更容易理解和维护。遵循前面提到的一些原则，如果我们保持类和方法的专注和精简，就能写出更简洁的代码。

### 6.3 测试驱动开发

TDD代表“测试驱动开发”，这是一种编程实践，要求我们[仅在自动化测试失败时才编写代码](https://www.baeldung.com/java-test-driven-list)。因此，我们必须**从自动化测试的设计开发开始**。在Java中，有多个框架可以编写自动化单元测试，例如JUnit和TestNG。

这种做法的好处是巨大的，它能让软件始终按预期运行。由于我们总是从测试开始，我们会逐步地、小块地添加可用的代码。此外，我们只在新的或任何旧的测试失败时才添加代码，这意味着它也提高了可重用性。

## 7. 帮助工具

编写简洁的代码不仅仅是原则和实践的问题，更是一种个人习惯。随着学习和适应，我们往往会成长为更优秀的开发人员。然而，为了在大型团队中保持一致性，我们也必须采取一些强制措施，**代码审查一直是保持一致性并通过建设性反馈帮助开发人员成长的绝佳工具**。

然而，我们不必在代码审查期间手动验证所有这些原则和最佳实践。[Java OffHeap](https://www.javaoffheap.com/2019/10/episode-47-microsoft-flexing-its-java-muscle-javafx-is-alive-and-well-and-would-you-approve-my-low-quality-pr.html)的Freddy Guime谈到了自动化部分质量检查的价值，以便始终保持代码质量达到一定的阈值。

Java生态系统中有一些工具，它们可以分担代码审阅人员的部分职责，让我们看看这些工具有哪些：

- 代码格式化程序：大多数流行的Java代码编辑器(包括Eclipse和IntelliJ)都支持自动代码格式化，我们可以使用默认的格式化规则，也可以自定义它们，或者用自定义的格式化规则替换它们，这解决了许多结构化代码规范的问题。
- 静态分析工具：Java有多种静态代码分析工具，包括[SonarQube](https://www.baeldung.com/sonar-qube)、[Checkstyle](https://www.baeldung.com/checkstyle-java)、[PMD](https://www.baeldung.com/pmd)和[SpotBugs](https://spotbugs.github.io/)。它们拥有丰富的规则集，可以直接使用，也可以针对特定项目进行定制。它们非常擅长检测各种[代码异味](https://www.baeldung.com/cs/code-smells)，例如违反命名约定和资源泄漏。

## 8. 总结

在本教程中，我们探讨了整洁代码原则的重要性以及整洁代码所展现的特征。我们还学习了如何在Java开发实践中运用这些原则，我们还讨论了其他有助于保持代码可读性和可维护性的最佳实践。最后，我们讨论了一些可用于帮助我们实现这一目标的工具。

总而言之，重要的是要注意，所有这些原则和实践都是为了让我们的代码更简洁。这是一个比较主观的术语，因此必须结合具体情况来评估。

虽然有众多规则可供采用，但我们必须考虑到自身的成熟度、文化和需求。我们可能需要定制规则，甚至可能需要制定一套全新的规则。但无论如何，重要的是在整个组织内保持一致，才能从中获益。