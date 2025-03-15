---
layout: post
title:  对Mockito Spy对象进行多级Mock注入
category: mock
copyright: mock
excerpt: Mockito
---

## 1. 概述

在本教程中，我们将讨论众所周知的[Mockito](https://www.baeldung.com/mockito-annotations)注解@InjectMocks、@Mock、@Spy，并了解它们如何在多级注入场景中协同工作。我们将讨论重要的测试概念，并学习如何进行正确的测试配置。

## 2. 多级注入概念

多级注入是一个强大的概念，但如果误用，则可能很危险。在开始实现之前，让我们先回顾一下重要的理论概念。

### 2.1 单元测试概念

**根据定义，单元测试是涵盖一个源代码单元的测试**。在Java世界中，我们可以将单元测试视为涵盖某些特定类(Service、Repository、实用程序等)的测试。

测试类时，我们只想测试其业务逻辑，而不是其依赖项的行为。为了处理依赖项(例如模拟它们或验证它们的使用情况)，我们通常使用Mock框架-Mockito。它旨在扩展现有的测试引擎(JUnit，TestNG)，并有助于为具有多个依赖项的类正确构建单元测试。

### 2.2 @Spy概念

Spy是Mockito的重要支柱之一，有助于有效地处理依赖关系。

Mock是一个完整的存根，在调用方法时不执行任何操作，也不会到达实际对象。相反，Spy默认将所有调用委托给真实对象方法。并且，当指定时，Spy方法可以作为具有其所有功能的Mock运行。

**由于Spy对象的默认行为与真实对象相同，因此我们需要为其设置必要的依赖项**。Mockito会尝试将依赖项隐式注入到Spy对象中。但是，我们可以在需要时显式设置依赖项。

### 2.3 多级注入风险

Mockito中的多级注入是指被测试的类需要一个Spy，而这个Spy又依赖于具体的Mock进行注入，从而形成一个嵌套的注入结构。

在某些情况下，我们可能需要进行多级Mock。例如，我们正在测试某个ServiceA，它依赖于某个复杂的MapperA，而MapperA又依赖于不同的ServiceB。通常，将MapperA设为Spy并将ServiceB作为Mock注入会更容易。但是，这种方法破坏了单元测试概念。**当我们需要在测试中涵盖多个服务时，我们应该坚持完整的集成测试**。

**如果我们经常需要进行多级注入，这可能表明测试方法不正确或代码设计复杂，需要重构**。

## 3. 设置示例场景

在继续测试用例之前，让我们定义示例代码来展示如何使用Mockito来测试我们的代码库。

我们将使用图书馆概念，其中我们有一个主要的图书实体，由多个服务处理。这里最重要的一点是类之间的依赖关系。

处理的入口点是BookStorageService，用于存储有关取走/赠送书籍的信息并通过BookControlService验证书籍状态：

```java
public class BookStorageService {
    private BookControlService bookControlService;
    private final List<Book> availableBooks;
}
```

另一方面，BookControlService依赖于另外两个类，StatisticService和RepairService，它们应该计算已处理的书籍数量并检查书籍是否需要修复：

```java
public class BookControlService {
    private StatisticService statisticService;
    private RepairService repairService;
}
```

## 4. 深入了解@InjectMocks注解

当我们考虑将一个Mockito管理的类注入另一个类时，@InjectMocks注解看起来是最直观的机制。但是，它的功能有限。[文档](https://javadoc.io/static/org.mockito/mockito-core/5.10.0/org/mockito/InjectMocks.html)强调，**Mockito不应被视为依赖项注入框架，它不是为处理对象网络的复杂注入而设计的**。

此外，Mockito不会报告任何注入失败。换句话说，当Mockito无法将Mock注入字段时，该字段将保持为空。因此，如果测试类设置不正确，我们最终会得到多个NullPointerException(NPE)。

**@InjectMocks在不同的配置下表现不同，并且并非每种设置都会以所需的方式工作**，让我们详细回顾一下注解使用的特点。

### 4.1 @InjectMocks和@Spy无效的配置

在一个类中使用多个@InjectMocks注解可能是直观的：

```java
public class MultipleInjectMockDoestWorkTest {
    @InjectMocks
    private BookStorageService bookStorageService;
    @Spy
    @InjectMocks
    private BookControlService bookControlService;
    @Mock
    private StatisticService statisticService;
}
```

这样的配置目的是将statisticService Mock注入到bookControlService Spy中，将bookControlService注入到bookStorageService中。**然而，这样的配置在较新的Mockito版本中不起作用并导致NPE**。

在底层，框架的当前版本(5.10.0)将所有带注解的对象收集到两个集合中。第一个集合用于使用@InjectMocks标注的Mock相关字段(bookStorageService和bookControlService)。

第二个集合是所有实体候选注入点，它们都是Mock和符合条件的Spy。**但是，同时标记为[@Spy和@InjectMock的字段](https://github.com/mockito/mockito/issues/2459)永远不会被视为注入候选**。因此，Mockito不会知道应该将bookControlService注入到bookStorageService中。

上述配置的另一个问题是概念上的。使用@InjectMocks注解，我们针对两个类(BookStorageService和BookControlService)进行测试，这违反了单元测试方法。

### 4.2 @InjectMocks和@Spy有效配置

同时，@Spy和@InjectMocks一起使用没有任何限制，只要类中只有一个@InjectMocks注解：

```java
@Spy
@InjectMocks
private BookControlService bookControlService;
@Mock
private StatisticService statisticService;
@Spy
private RepairService repairService;
```

通过此配置，我们拥有一个正确构建且可测试的层次结构：

```java
@Test
void whenOneInjectMockWithSpy_thenHierarchySuccessfullyInitialized(){
    Book book = new Book("Some name", "Some author", 355, ZonedDateTime.now());
    bookControlService.returnBook(book);

    Assertions.assertNull(book.getReturnDate());
    Mockito.verify(statisticService).calculateAdded();
    Mockito.verify(repairService).shouldRepair(book);
}
```

## 5. 通过@InjectMocks和手动Spy进行多级注入

**解决多级注入的选项之一是在Mockito初始化之前手动实例化Spy对象**，正如我们已经讨论过的，Mockito无法将所有依赖项注入到使用@Spy和@InjectMocks标注的字段中。但是，框架可以向仅使用@InjectMocks标注的对象注入依赖项，即使该对象是Spy。

我们可以在类级别使用@ExtendWith(MockitoExtension.class)并在字段中初始化Spy：

```java
@InjectMocks
private BookControlService bookControlService = Mockito.spy(BookControlService.class);
```

或者我们可以使用MockitoAnnotations.openMocks(this)并在@BeforeEach方法中初始化Spy：

```java
@BeforeEach
public void openMocks() {
    bookControlService = Mockito.spy(BookControlService.class);
    closeable = MockitoAnnotations.openMocks(this);
}
```

**在这两种情况下，都应该在Mockito初始化之前创建一个Spy**。

通过上述设置，Mockito处理手动创建的Spy上的@InjectMocks并注入所有需要的Mock：

```java
@InjectMocks
private BookStorageService bookStorageService;
@InjectMocks
private BookControlService bookControlService;
@Mock
private StatisticService statisticService;
@Mock
private RepairService repairService;
```

测试成功执行：

```java
@Test
void whenSpyIsManuallyCreated_thenInjectMocksWorks() {
    Book book = new Book("Some name", "Some author", 355);
    bookStorageService.returnBook(book);

    Assertions.assertEquals(1, bookStorageService.getAvailableBooks().size());
    Mockito.verify(bookControlService).returnBook(book);
    Mockito.verify(statisticService).calculateAdded();
    Mockito.verify(repairService).shouldRepair(book);
}
```

## 6. 通过反射实现多级注入

处理复杂测试设置的另一种可能方法是手动创建所需的间Mock对象，然后将其注入到被测对象中。**利用反射机制，我们可以用所需的Mock更新Mockito创建的对象**。

在以下示例中，我们手动配置了所有内容，而不是使用@InjectMocks标注BookControlService。为了确保Mockito创建的Mock在Mock初始化期间可用，必须先初始化Mockito上下文。否则，无论在哪里使用Mock，都可能会出现NullPointerException。

一旦BookControlService Spy配置了所有Mock，我们就通过反射将其注入到BookStorageService：

```java
@InjectMocks
private BookStorageService bookStorageService;
@Mock
private StatisticService statisticService;
@Mock
private RepairService repairService;
private BookControlService bookControlService;

@BeforeEach
public void openMocks() throws Exception {
    bookControlService = Mockito.spy(new BookControlService(statisticService, repairService));
    injectSpyToTestedMock(bookStorageService, bookControlService);
}

private void injectSpyToTestedMock(BookStorageService bookStorageService, BookControlService bookControlService) throws NoSuchFieldException, IllegalAccessException { 
    Field bookControlServiceField = BookStorageService.class.getDeclaredField("bookControlService"); 
    bookControlServiceField.setAccessible(true); 
    bookControlServiceField.set(bookStorageService, bookControlService); 
}
```

通过这样的配置，我们可以验证repairService和bookControlService的行为。

## 7. 总结

在本文中，我们回顾了重要的单元测试概念，并学习了如何使用@InjectMocks、@Spy和@Mock注解来执行复杂的多级注入。我们还了解了如何手动配置Spy以及如何将其注入到测试对象中。