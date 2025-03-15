---
layout: post
title:  在Spring Boot测试中使用@Autowired和@InjectMocks
category: spring-test
copyright: spring-test
excerpt: Spring Test
---

## 1. 概述

在本教程中，我们将探讨在Spring Boot测试中注入依赖项时Spring Boot的[@Autowired](https://www.baeldung.com/spring-autowire)和Mockito的[@InjectMocks](https://www.baeldung.com/mockito-annotations#injectmocks-annotation)的用法，我们将讨论需要使用它们的用例并查看示例。

## 2. 理解测试注解

在开始代码示例之前，让我们快速了解一些测试注解的基础知识。

首先，Mockito中最常用的[@Mock](https://www.baeldung.com/mockito-annotations#mock-annotation)注解用于创建依赖项的Mock实例以供测试，**它通常与@InjectMocks结合使用，将标有@Mock的Mock注入到被测试的目标对象中**。

除了Mockito的注解之外，Spring Boot的注解[@MockBean](https://www.baeldung.com/java-spring-mockito-mock-mockbean#spring-boots-mockbean-annotation)可以帮助创建Mock的Spring bean，然后，上下文中的其他Bean可以使用Mock的Bean。**此外，如果Spring上下文自行创建了无需Mock即可使用的Bean，我们可以使用@Autowired注解来注入它们**。

## 3. 示例设置

在我们的代码示例中，我们将创建一个具有两个依赖项的服务，然后我们将探索使用上述注解来测试该服务。

### 3.1 依赖项

让我们首先添加所需的依赖项，包括[Spring Boot Starter Web](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web) 和[Spring Boot Starter Test](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-test)依赖项：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>3.3.2</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <version>3.3.2</version>
    <scope>test</scope>
</dependency>
```

除此之外，我们还将添加Mock服务所需的[Mockito Core](https://mvnrepository.com/artifact/org.mockito/mockito-core)依赖：

```xml
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>5.12.0</version>
</dependency>
```

### 3.2 DTO

接下来，让我们创建一个将在服务中使用的[DTO](https://www.baeldung.com/java-dto-pattern)：

```java
public class Book {
    private String id;
    private String name;
    private String author;
    
    // constructor, setters/getters
}
```

### 3.3 Service

接下来看看我们的Service，首先，我们定义一个负责数据库交互的服务：

```java
@Service
public class DatabaseService {
    public Book findById(String id) {
        // querying a Database and getting a book
        return new Book("id","Name", "Author");
    }
}
```

我们不会深入讨论数据库交互，因为它们与示例无关。我们使用[@Service](https://www.baeldung.com/spring-service-annotation-placement)注解将类声明为Service构造型的Spring Bean。

接下来我们来介绍一下依赖上述服务的一个服务：

```java
@Service
public class BookService {
    private DatabaseService databaseService;
    private ObjectMapper objectMapper;

    BookService(DatabaseService databaseService, ObjectMapper objectMapper) {
        this.databaseService = databaseService;
        this.objectMapper = objectMapper;
    }

    String getBook(String id) throws JsonProcessingException {
        Book book = databaseService.findById(id);
        return objectMapper.writeValueAsString(book);
    }
}
```

这里我们有一个小服务，它有一个getBook()方法，该方法利用DatabaseService从数据库中获取一本书。然后它使用Jackson的[ObjectMapper](https://www.baeldung.com/jackson-object-mapper-tutorial) API将Book对象转换并返回为JSON字符串。

因此，该服务有两个依赖：DatabaseService和ObjectMapper。

## 4. 测试

现在我们的服务已经设置好了，让我们看看如何使用我们之前定义的注解来测试BookService。

### 4.1 使用@Mock和@InjectMocks

第一个选项是使用@Mock模拟服务的两个依赖项，并使用@InjectMocks将它们注入到服务中。让我们为其创建一个测试类：

```java
@ExtendWith(MockitoExtension.class)
class BookServiceMockAndInjectMocksUnitTest {
    @Mock
    private DatabaseService databaseService;

    @Mock
    private ObjectMapper objectMapper;

    @InjectMocks
    private BookService bookService;

    @Test
    void givenBookService_whenGettingBook_thenBookIsCorrect() throws JsonProcessingException {
        Book book1 = new Book("1234", "Inferno", "Dan Brown");
        when(databaseService.findById(eq("1234"))).thenReturn(book1);

        when(objectMapper.writeValueAsString(any())).thenReturn(new ObjectMapper().writeValueAsString(book1));

        String bookString1 = bookService.getBook("1234");
        Assertions.assertTrue(bookString1.contains("Dan Brown"));
    }
}
```

首先，我们用@ExtendWith(MockitoExtension.class)标注测试类，MockitoExtension扩展允许我们在测试中Mock和注入对象。

接下来，我们声明DatabaseService和ObjectMapper字段并用@Mock标注它们，这会为它们两者创建Mock对象。在声明要测试的BookService实例时，**我们添加了@InjectMocks注解，这会注入服务所需的任何依赖，这些依赖之前已用@Mocks声明过**。

最后，在我们的测试中，我们模拟Mock对象的行为并测试我们服务的getBook()方法。

**使用此方法时，必须Mock服务的所有依赖项**。例如，如果我们不Mock ObjectMapper，则在测试方法中调用它时会导致NullPointerException。

### 4.2 使用@Autowired和@MockBean

在上述方法中，我们Mock了两个依赖项。但是，可能需要Mock某些依赖项而不Mock其他依赖项。假设我们不需要Mock ObjectMapper的行为，而仅Mock DatabaseService。

由于我们在测试中加载了Spring上下文，因此我们可以结合使用@Autowired和@MockBean注解来执行此操作：

```java
@SpringBootTest
class BookServiceAutowiredAndInjectMocksUnitTest {
    @MockBean
    private DatabaseService databaseService;

    @Autowired
    private BookService bookService;

    @Test
    void givenBookService_whenGettingBook_thenBookIsCorrect() throws JsonProcessingException {
        Book book1 = new Book("1234", "Inferno", "Dan Brown");
        when(databaseService.findById(eq("1234"))).thenReturn(book1);

        String bookString1 = bookService.getBook("1234");
        Assertions.assertTrue(bookString1.contains("Dan Brown"));
    }
}
```

首先，要使用Spring上下文中的Bean，我们需要使用@SpringBootTest标注我们的测试类。接下来，我们使用@MockBean标注DatabaseService。然后我们使用@Autowired从应用程序上下文中获取BookService实例。

**当BookService Bean被注入时，实际的DatabaseService Bean将被Mock的Bean替换**。相反，ObjectMapper Bean仍然与应用程序最初创建的相同。

当我们现在测试这个实例时，我们不需要Mock ObjectMapper的任何行为。

当我们需要测试嵌套Bean的行为并且不想Mock每个依赖项时，此方法很有用。

### 4.3 同时使用@Autowired和@InjectMocks

对于上述用例，我们还可以使用@InjectMocks代替@MockBean。

我们来看一下代码，看看这两种方法的区别：

```java
@Mock
private DatabaseService databaseService;

@Autowired
@InjectMocks
private BookService bookService;

@Test
void givenBookService_whenGettingBook_thenBookIsCorrect() throws JsonProcessingException {
    Book book1 = new Book("1234", "Inferno", "Dan Brown");

    MockitoAnnotations.openMocks(this);

    when(databaseService.findById(eq("1234"))).thenReturn(book1);
    String bookString1 = bookService.getBook("1234");
    Assertions.assertTrue(bookString1.contains("Dan Brown"));
}
```

在这里，我们使用@Mock而不是@MockBean来Mock DatabaseService。除了@Autowired，我们还将@InjectMocks注解添加到BookService实例。

当两个注解一起使用时，@InjectMocks不会自动注入Mock依赖项，并且会在测试开始时注入自动注入的BookService对象。

**但是，我们可以稍后在测试中通过调用MockitoAnnotations.openMocks()方法来注入DatabaseService的Mock实例**。此方法查找标有@InjectMocks的字段并将Mock对象注入其中。

在我们需要Mock DatabaseService的行为之前，我们在测试中调用它。当我们想要动态决定何时使用Mock以及何时使用实际Bean来获取依赖项时，此方法很有用。

## 5. 方法比较

现在我们已经研究了多种方法，让我们总结一下它们之间的比较：

|             方法             |                                    描述                                    |                                     用法                                    |
| :--------------------------: |:------------------------------------------------------------------------:|:-------------------------------------------------------------------------:|
|   @Mock与@InjectMocks    |    使用Mockito的@Mock注解创建依赖项的Mock实例，并使用@InjectMocks将这些Mock注入到正在测试的目标对象中     |                         适用于单元测试，我们想要Mock被测试类的所有依赖项                        |
|  @MockBean与@Autowired   |    利用Spring Boot的@MockBean注解创建一个Mock的Spring Bean并使用@Autowired注入这些Bean    |  非常适合在Spring Boot应用程序中进行集成测试，它允许Mock一些Spring Bean，同时从Spring的依赖注入中获取其他Bean |
| @InjectMocks与@Autowired | 使用Mockito的@Mock注解创建Mock实例，并使用@InjectMocks将这些Mock注入到已使用Spring自动注入的目标Bean中 | 在需要使用Mockito临时Mock某些依赖项以覆盖注入的Spring Bean的情况下，提供了灵活性。适用于在Spring应用程序中测试复杂场景 |

## 6. 总结

在本文中，我们研究了Mockito和Spring Boot注解的不同用例-@Mock、@InjectMocks、@Autowired和@MockBean，我们探讨了何时根据测试需求使用注解的不同组合。