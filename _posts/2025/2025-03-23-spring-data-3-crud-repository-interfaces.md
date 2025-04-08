---
layout: post
title:  Spring Data 3中的新CRUD Repository接口
category: springdata
copyright: springdata
excerpt: Spring Data JPA
---

## 1. 概述

在本教程中，我们将了解Spring Data 3中引入的新Repository接口。

Spring Data 3引入了基于List的CRUD Repository接口，可用于替换现有的返回Iterable的CRUD Repository接口。此外，分页和排序接口默认不从原始CRUD Repository继承，而是将该选项留给用户。我们将看到这些接口与现有接口有何不同以及如何使用它们。

## 2. 项目设置

让我们从设置项目开始，我们将创建一个Spring Boot应用程序，其中包含一个将使用新接口的简单实体和Repository。

### 2.1 依赖

首先，将所需的依赖添加到我们的项目中，我们将添加[Spring Boot Starter Data](https://central.sonatype.com/search?q=spring-boot-starter-data-jpa&namespace=org.springframework.boot)依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <version>3.1.5</version>
</dependency>
```

除此之外，我们还将配置数据库。可以使用任何SQL数据库，在本教程中，我们将使用[内存H2数据库](https://www.baeldung.com/spring-boot-h2-database)。让我们为其添加一个依赖：

```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
</dependency>
```

### 2.2 实体

让我们创建一个将在Repository中使用的[实体](https://www.baeldung.com/jpa-entities)Book：

```java
@Entity
public class Book {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String title;
    private String author;
    private String isbn;

    // Constructor, getters and setters
}
```

现在我们有了实体，让我们看看如何使用新的接口与数据库交互。

## 3. 基于列表的CRUD Repository

**Spring Data 3引入了一组新的CRUD Repository接口，这些接口返回实体列表**。这些接口与现有接口类似，只是它们返回List而不是Iterable。这使我们能够使用List接口中的高级方法，例如get()和indexOf()。

### 3.1 基于列表的Repository接口

Spring Data 3中添加了三个新接口。

ListCrudRepository接口提供了用于基本CRUD操作的方法，例如save()、delete()和findById()。ListCrudRepository与其基于Iterable的对应项之间的主要区别在于，**返回的值现在是列表，这使我们能够对返回的数据执行更高级的操作**。

ListQuerydslPredicateExecutor接口提供了使用Querydsl谓词执行查询的方法。Querydsl是一个框架，允许我们在Java中构建类型安全的类似SQL的查询。**使用ListQuerydslPredicateExecutor，我们可以执行Querydsl查询并以列表形式返回结果**。

**ListQueryByExampleExecutor接口提供了使用示例实体执行查询的方法，并以列表形式返回结果**。示例实体是包含我们想要用来搜索其他实体的值的实体，例如，如果我们有一个标题为Spring Data的Book实体，我们可以创建一个标题为Spring Data的示例实体并使用它来搜索具有相同标题的其他书籍。

### 3.2 ListCrudRepository

让我们详细了解ListCrudRepository接口，我们将创建一个使用此接口与数据库交互的Repository：

```java
@Repository
public interface BookListRepository extends ListCrudRepository<Book, Long> {
    
    List<Book> findBooksByAuthor(String author);
}
```

**上面的Repository扩展了ListCrudRepository接口，该接口为我们提供了基本的CRUD方法**。我们可以使用现有的Repository方法在数据库中保存、删除和查找书籍。

除了这些方法之外，我们还定义了findBooksByAuthor()方法来按作者查找书籍。

### 3.3 使用基于列表的Repository接口

让我们看看如何使用这个Repository与数据库交互：

```java
@SpringBootTest
public class BookListRepositoryIntegrationTest {

    @Autowired
    private BookListRepository bookListRepository;

    @Test
    public void givenDbContainsBooks_whenFindBooksByAuthor_thenReturnBooksByAuthor() {
        Book book1 = new Book("Spring Data", "John Doe", "1234567890");
        Book book2 = new Book("Spring Data 2", "John Doe", "1234567891");
        Book book3 = new Book("Spring Data 3", "John Doe", "1234567892");
        bookListRepository.saveAll(Arrays.asList(book1, book2, book3));

        List<Book> books = bookListRepository.findBooksByAuthor("John Doe");
        assertEquals(3, books.size());
    }
}
```

我们首先创建了三本书并将它们保存到数据库中。然后，我们使用findBooksByAuthor()方法查找作者John Doe的所有书籍。最后，我们验证返回的列表是否包含我们创建的三本书。

**请注意，我们在返回的列表上调用了size()方法。如果Repository使用基于Iterable的接口，这是不可能的，因为Iterable接口不提供size()方法**。

## 4. Repository排序

为了能够使用新的基于List的接口，Spring Data 3必须对现有的排序接口进行更改。**排序Repository不再扩展旧的CRUD Repository。相反，用户可以选择扩展新的基于List的接口或基于Iterable的接口以及排序接口**。让我们看看如何使用最新的排序接口。

### 4.1 PagingAndSortingRepository

让我们详细了解一下PagingAndSortingRepository接口，我们将创建一个使用此接口与数据库交互的Repository：

```java
@Repository
public interface BookPagingAndSortingRepository extends PagingAndSortingRepository<Book, Long>, ListCrudRepository<Book, Long> {
    
    List<Book> findBooksByAuthor(String author, Pageable pageable);
}
```

上述Repository扩展了ListCrudRepository接口，该接口为我们提供了基本的CRUD方法。除此之外，**我们还扩展了PagingAndSortingRepository接口，该接口为我们提供了排序和分页的方法**。

在[旧版本的Spring Data](https://www.baeldung.com/spring-data-jpa-pagination-sorting)中，我们不需要显式扩展CRUD Repository接口，仅排序和分页接口就足够了。我们定义了一个名为findBooksByAuthor()的新方法，该方法接收一个Pageable参数并返回书籍列表。在下一节中，我们将了解如何使用此方法对结果进行排序和分页。

### 4.2 使用排序Repository接口

让我们看看如何使用这个Repository与数据库交互：

```java
@SpringBootTest
public class BookPagingAndSortingRepositoryIntegrationTest {

    @Autowired
    private BookPagingAndSortingRepository bookPagingAndSortingRepository;

    @Test
    public void givenDbContainsBooks_whenfindBooksByAuthor_thenReturnBooksByAuthor() {
        Book book1 = new Book("Spring Data", "John Doe", "1234567890");
        Book book2 = new Book("Spring Data 2", "John Doe", "1234567891");
        Book book3 = new Book("Spring Data 3", "John Doe", "1234567892");
        bookPagingAndSortingRepository.saveAll(Arrays.asList(book1, book2, book3));

        Pageable pageable = PageRequest.of(0, 2, Sort.by("title").descending());
        List<Book> books = bookPagingAndSortingRepository.findBooksByAuthor("John Doe", pageable);
        assertEquals(2, books.size());
        assertEquals(book3.getId(), books.get(0).getId());
        assertEquals(book2.getId(), books.get(1).getId());
    }
}
```

和以前一样，我们首先创建了三本书并将它们保存到数据库中。然后，我们使用findBooksByAuthor()方法查找作者John Doe的所有书籍。但这次，我们将Pageable对象传递给该方法，按标题降序对结果进行排序，并仅返回前两个结果。

最后，我们验证了返回的列表包含我们创建的两本书。我们还验证了这些书按标题降序排列，以便首先返回标题为Spring Data 3的书，其次返回标题为Spring Data 2的书。

### 4.3 其他排序接口

除了PagingAndSortingRepository接口之外，以下接口也发生了变化：

- ReactiveSortingRepository不再继承自ReactiveCrudRepository
- CoroutineSortingRepository不再继承自CoroutineCrudRepository
- RxJavaSortingRepository不再继承自RxJavaCrudRepository

## 5. 总结

在本文中，我们探讨了Spring Data 3中添加的新的基于List的接口。我们研究了如何使用这些接口与数据库交互，我们还研究了对现有排序接口所做的更改，以便能够与新的基于List的接口一起使用。