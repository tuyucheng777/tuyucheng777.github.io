---
layout: post
title:  实体和DTO之间的差异
category: designpattern
copyright: designpattern
excerpt: DTO
---

## 1. 概述

在软件开发领域，实体和[DTO](https://www.baeldung.com/java-dto-pattern)(数据传输对象)之间存在明显的区别，**了解它们的确切作用和区别可以帮助我们构建更高效、更易于维护的软件**。

在本文中，我们将探讨实体和DTO之间的区别，并尝试清晰地解释它们的用途，以及在软件项目中何时使用它们。在讲解每个概念的同时，我们将使用Spring Boot和[JPA](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)来绘制一个简单的用户管理应用。

## 2. 实体

**实体是表示应用程序领域中现实世界对象或概念的基本组件**，它们通常直接对应于数据库表或领域对象。因此，它们的主要目的是封装和管理这些对象的状态和行为。

### 2.1 实体示例

让我们为项目创建一些实体，表示拥有多本书的用户，我们首先创建Book实体：

```java
@Entity
@Table(name = "books")
public class Book {

    @Id
    private String name;
    private String author;

    // standard constructors / getters / setters
}
```

现在，我们需要定义我们的User实体：

```java
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private String firstName;
    private String lastName;
    private String address;

    @OneToMany(cascade=CascadeType.ALL)
    private List<Book> books;

    public String getNameOfMostOwnedBook() {
        Map<String, Long> bookOwnershipCount = books.stream()
                .collect(Collectors.groupingBy(Book::getName, Collectors.counting()));
        return bookOwnershipCount.entrySet().stream()
                .max(Map.Entry.comparingByValue())
                .map(Map.Entry::getKey)
                .orElse(null);
    }

    // standard constructors / getters / setters
}
```

### 2.2 实体特征

在我们的实体中，我们可以识别出一些独特的特征。首先，**实体通常包含对象关系映射(ORM)注解**。例如，@Entity注解将类标记为实体，从而在Java类和数据库表之间建立直接链接。

@Table注解用于指定与实体关联的数据库表的名称，此外，@Id注解将字段定义为主键，这些ORM注解简化了数据库映射的过程。

此外，**实体通常需要与其他实体建立关系**，以反映现实世界概念之间的关联，一个常见的例子是我们用来定义用户与其拥有的书籍之间的一对多关系的@OneToMany注解。

此外，实体不必仅仅充当被动数据对象，还可以包含特定领域的业务逻辑。例如，我们来考虑一个方法，例如getNameOfMostOwnedBook()，此方法位于实体内，封装了特定领域的逻辑，用于查找用户拥有最多的书籍名称，**这种方法符合面向对象编程(OOP)原则和领域驱动设计[(DDD)](https://martinfowler.com/bliki/DomainDrivenDesign.html)方法**，将特定领域的操作保留在实体内，从而促进代码组织和封装。

此外，实体可能包含其他特性，例如[验证约束](https://www.baeldung.com/java-validation)或[生命周期方法](https://www.baeldung.com/jpa-entity-lifecycle-events)。

## 3. DTO

**DTO主要充当纯数据载体，不包含任何业务逻辑**，它们用于在不同应用程序之间或同一应用程序的不同部分之间传输数据。

在简单的应用程序中，通常将领域对象直接用作DTO。然而，随着应用程序复杂度的增加，从安全性和封装性的角度来看，将整个领域模型暴露给外部客户端可能变得不那么可取。

### 3.1 DTO示例

为了使我们的应用程序尽可能简单，我们将仅实现创建新用户和检索当前用户的功能。为此，让我们首先创建一个DTO来表示一本书：

```java
public class BookDto {

    @JsonProperty("NAME")
    private final String name;

    @JsonProperty("AUTHOR")
    private final String author;

    // standard constructors / getters
}
```

对于用户，我们定义两个DTO，一个用于创建用户，另一个用于响应：

```java
public class UserCreationDto {

    @JsonProperty("FIRST_NAME")
    private final String firstName;

    @JsonProperty("LAST_NAME")
    private final String lastName;

    @JsonProperty("ADDRESS")
    private final String address;

    @JsonProperty("BOOKS")
    private final List<BookDto> books;

    // standard constructors / getters
}
```

```java
public class UserResponseDto {

    @JsonProperty("ID")
    private final Long id;

    @JsonProperty("FIRST_NAME")
    private final String firstName;

    @JsonProperty("LAST_NAME")
    private final String lastName;

    @JsonProperty("BOOKS")
    private final List<BookDto> books;

    // standard constructors / getters
}
```

### 3.2 DTO特点

根据我们的示例，我们可以识别出一些特殊性：不变性、验证注解和JSON映射注解。

**使DTO不可变是一种最佳实践**，不可变性可确保传输的数据在传输过程中不会被意外更改，实现这一点的一种方法是将所有属性声明为final，并且不实现Setter方法。或者，[Lombok](https://www.baeldung.com/intro-to-project-lombok)中的[@Value](https://projectlombok.org/features/Value)注解或Java 14中引入的[Java记录](https://www.baeldung.com/java-record-keyword)提供了一种简洁的方法来创建不可变的DTO。

接下来，**DTO也可以从验证中受益**，以确保通过DTO传输的数据符合特定标准。这样，我们可以在数据传输过程的早期检测并拒绝无效数据，**防止不可靠信息污染域**。

此外，我们通常可以在DTO中找到[JSON映射注解](https://www.baeldung.com/jackson-annotations)，**将JSON属性映射到DTO的字段**。例如，@JsonProperty注解允许我们指定DTO的JSON名称。

## 4. Repository、映射器和控制器

为了演示在我们的应用程序中同时使用实体和DTO来表示数据的实用性，我们需要完成代码，首先为User实体创建一个Repository：

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
}
```

接下来，我们将继续创建一个映射器，以便能够从一个转换为另一个：

```java
public class UserMapper {

    public static UserResponseDto toDto(User entity) {
        return new UserResponseDto(
                entity.getId(),
                entity.getFirstName(),
                entity.getLastName(),
                entity.getBooks().stream().map(UserMapper::toDto).collect(Collectors.toList())
        );
    }

    public static User toEntity(UserCreationDto dto) {
        return new User(
                dto.getFirstName(),
                dto.getLastName(),
                dto.getAddress(),
                dto.getBooks().stream().map(UserMapper::toEntity).collect(Collectors.toList())
        );
    }

    public static BookDto toDto(Book entity) {
        return new BookDto(entity.getName(), entity.getAuthor());
    }

    public static Book toEntity(BookDto dto) {
        return new Book(dto.getName(), dto.getAuthor());
    }
}
```

在我们的示例中，我们手动完成了实体和DTO之间的映射。对于更复杂的模型，为了避免样板代码，我们可以使用[MapStruct](https://www.baeldung.com/mapstruct)之类的工具。

现在，我们只需要创建控制器：

```java
@RestController
@RequestMapping("/users")
public class UserController {

    private final UserRepository userRepository;

    public UserController(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @GetMapping
    public List<UserResponseDto> getUsers() {
        return userRepository.findAll().stream().map(UserMapper::toDto).collect(Collectors.toList());
    }

    @PostMapping
    public UserResponseDto createUser(@RequestBody UserCreationDto userCreationDto) {
        return UserMapper.toDto(userRepository.save(UserMapper.toEntity(userCreationDto)));
    }
}
```

请注意，使用findAll()可能会影响大型集合的性能。在这种情况下，添加[分页](https://www.baeldung.com/spring-data-jpa-pagination-sorting)之类的功能可能会有所帮助。

## 5. 为什么我们同时需要实体和DTO？

### 5.1 关注点分离

在我们的示例中，**实体与数据库模式和特定于域的操作紧密相关**。另一方面，**DTO仅用于数据传输目的**。

在某些架构范式中，例如[六边形架构](https://www.baeldung.com/hexagonal-architecture-ddd-spring)，我们可能会发现一个额外的层，通常称为模型或领域模型，这一层的关键作用是将领域与任何侵入式技术完全解耦。这样，核心业务逻辑就能够独立于数据库、框架或外部系统的实现细节。

### 5.2 隐藏敏感数据

在与外部客户端或系统打交道时，控制哪些数据暴露给外界至关重要。实体可能包含敏感信息或业务逻辑，这些信息或逻辑应该对外部用户隐藏。DTO充当一道屏障，帮助我们只向客户端公开安全且相关的数据。

### 5.3 性能

Martin Fowler提出的[DTO模式](https://martinfowler.com/eaaCatalog/dataTransferObject.html)，指的是在一次调用中批量处理多个参数，我们无需多次调用来获取单个数据，而是将相关数据打包成一个DTO，并在一次请求中传输，**这种方法可以减少多次网络调用带来的开销**。

实现DTO模式的一种方法是通过[GraphQL](https://www.baeldung.com/spring-graphql)，它允许客户端指定所需的数据，从而允许在单个请求中进行多个查询。

## 6. 总结

**正如我们在本文中所了解到的，实体和DTO具有不同的角色，并且可能截然不同**。实体和DTO的结合可以确保复杂软件系统中的数据安全、关注点分离和高效的数据管理，这种方法可以带来更健壮、更易于维护的软件解决方案。