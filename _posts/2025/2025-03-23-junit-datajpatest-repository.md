---
layout: post
title:  JUnit中的@DataJpaTest和Repository类
category: springdata
copyright: springdata
excerpt: Spring Data JPA
---

## 1. 简介

在使用利用[Spring Data JPA](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)进行数据持久化的Spring Boot应用程序时，测试与数据库交互的Repository至关重要。在本教程中，我们将探讨如何使用Spring Boot提供的@DataJpaTest注解以及[JUnit](https://www.baeldung.com/junit-5)有效地测试Spring Data JPA Repository。

## 2. 理解@DataJpaTest和Repository类

在本节中，我们将深入研究Spring Boot应用程序上下文中@DataJpaTest和类Repository之间的交互。

### 2.1 @DataJpaTest

@DataJpaTest注解用于测试Spring Boot应用程序中的JPA Repository，**它是一种专门的测试注解，为测试持久层提供了最小的Spring上下文**。此注解可与其他测试注解(如@RunWith和[@SpringBootTest](https://www.baeldung.com/spring-boot-testing#integration-testing-with-springboottest))结合使用。

**此外，@DataJpaTest的范围仅限于应用程序的JPA Repository层**。它不会加载整个应用程序上下文，这可以使测试更快、更有针对性。此注解还提供了预配置的EntityManager 和TestEntityManager来测试JPA实体。

### 2.2 Repository类

在Spring Data JPA中，Repository充当JPA实体之上的抽象层，**它提供了一组用于执行CRUD(创建、读取、更新、删除)操作和执行自定义查询的方法**。这些Repository通常从[JpaRepository](https://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/JpaRepository.html)等接口扩展而来，负责处理与特定实体类型相关的数据库交互。

## 3. 可选参数

@DataJpaTest确实有一些可选参数，我们可以用来定制测试环境。

### 3.1 properties

此参数允许我们指定将应用于测试上下文的Spring Boot配置属性，**这对于调整数据库连接详细信息、事务行为或与我们的测试需求相关的其他应用程序属性等设置非常有用**：

```java
@DataJpaTest(properties = {
        "spring.datasource.url=jdbc:h2:mem:testdb",
        "spring.jpa.hibernate.ddl-auto=create-drop"
})
public class UserRepositoryTest {
    // ... test methods
}
```

### 3.2 showSql

这将为我们的测试启用SQL日志记录，并允许我们查看Repository方法执行的实际SQL查询。此外，**这可以帮助调试或了解JPA查询的转换方式**。默认情况下，SQL日志记录处于启用状态，我们可以通过将值设置为false来将其关闭：

```java
@DataJpaTest(showSql = false)
public class UserRepositoryTest {
    // ... test methods
}
```

### 3.3 includeFilters和excludeFilters

这些参数使我们能够在组件扫描期间包含或排除特定组件，**我们可以使用它们来缩小扫描范围**，并通过仅关注相关组件来优化测试性能：

```java
@DataJpaTest(includeFilters = @ComponentScan.Filter(
        type = FilterType.ASSIGNABLE_TYPE,
        classes = UserRepository.class),
        excludeFilters = @ComponentScan.Filter(
                type = FilterType.ASSIGNABLE_TYPE,
                classes = SomeIrrelevantRepository.class))
public class UserRepositoryTest {
    // ... test methods
}
```

## 4. 主要特点

在Spring Boot应用程序中测试JPA Repository时，@DataJpaTest注解可能是一个方便的工具，让我们详细探讨其主要功能和优势。

### 4.1 测试环境配置

为JPA Repository设置适当的测试环境可能非常耗时且棘手，**@DataJpaTest提供了一个现成的测试环境，其中包括测试JPA Repository的基本组件，例如EntityManager和DataSource**。

此环境专为测试JPA Repository而设计，它确保我们的Repository方法在测试事务的上下文中运行，并与安全的内存数据库(如H2)进行交互，而不是与生产数据库进行交互。

### 4.2 依赖注入

**@DataJpaTest简化了测试类中的依赖注入过程**，Repository以及其他必要的Bean会自动注入到测试上下文中，这种无缝集成使开发人员能够专注于编写简洁有效的测试用例，而无需进行显式Bean注入。

### 4.3 默认回滚

此外，保持测试独立可靠也至关重要。**默认情况下，每个使用@DataJpaTest标注的测试方法都在事务边界内运行**，这可确保对数据库所做的更改在测试结束时自动回滚，为下一次测试留下一个干净的记录。

## 5. 配置和设置

要使用@DataJpaTest，我们需要将[spring-boot-starter-test](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-test)依赖添加到我们的项目中，范围为“test”。此轻量级依赖包括用于测试的基本测试库(如JUnit)，确保它不包含在我们的生产版本中。

### 5.1 在pom.xml中添加依赖

让我们向pom.xml添加以下依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

添加依赖后，我们可以在测试中使用@DataJpaTest注解。此注解会设置内存中的H2数据库并为我们配置Spring Data JPA，使我们能够编写与Repository类交互的测试。

### 5.2 创建实体类

现在，让我们创建User实体类，它将代表用户数据：

```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String username;
    private String password;

    // getters and setters
}
```

### 5.3 创建Repository接口

接下来，我们定义UserRepository，这是一个用于管理User实体的Spring Data JPA Repository接口：

```java
public interface UserRepository extends JpaRepository<User, Long> {
    // Add custom methods if needed
}
```

通过扩展JpaRepository<User, Long\>，我们的UserRepository可以开箱即用地访问Spring Data JPA提供的标准CRUD操作。

此外，我们可以在此接口内定义自定义查询方法来满足特定的数据访问检索需求，例如findByUsername()：

```java
public interface UserRepository extends JpaRepository<User, Long> {
    // Custom query method to find a user by username
    User findByUsername(String username);
}
```

## 6. 实施Repository测试

为了测试应用程序的Repository层，我们将使用@DataJpaTest注解。**通过使用此注解，将设置内存中的H2数据库，并配置Spring Data JPA**，这使我们能够编写与Repository类交互的测试。

### 6.1 设置测试类

首先，让我们通过使用@DataJpaTest注解来设置测试类。此注解扫描使用@Entity标注的实体类和Spring Data JPA Repository接口，这可确保仅加载相关组件进行测试，从而提高测试重点和性能：

```java
@DataJpaTest
public class UserRepositoryTest {
    // Add test methods here
}
```

要创建Repository测试用例，我们首先需要将要测试的Repository注入到测试类中，这可以使用@Autowired注解来完成：

```java
@Autowired
private UserRepository userRepository;
```

### 6.2 测试生命周期管理

在测试生命周期管理中，@BeforeEach和@AfterEach注解分别用于在每个测试方法之前和之后执行设置和拆卸操作，**这可确保每个测试方法都在干净、隔离的环境中运行，并具有一致的初始条件和清理程序**。

以下是我们如何将测试生命周期管理纳入我们的测试类：

```java
private User testUser;

@BeforeEach
public void setUp() {
    // Initialize test data before each test method
    testUser = new User();
    testUser.setUsername("testuser");
    testUser.setPassword("password");
    userRepository.save(testUser);
}

@AfterEach
public void tearDown() {
    // Release test data after each test method
    userRepository.delete(testUser);
}
```

在用@BeforeEach标注的setUp()方法中，我们可以执行每次测试方法执行之前所需的任何必要的设置操作，这可能包括初始化测试数据、设置Mock对象或准备测试所需的资源。

相反，在用@AfterEach标注的tearDown()方法中，我们可以在每次执行测试方法后执行清理操作，这可能涉及重置测试期间所做的任何更改，释放资源或执行任何必要的清理任务以将测试环境恢复到其原始状态。

### 6.3 测试插入操作

现在，我们可以编写与JPA Repository交互的测试方法。例如，我们可能想要测试是否可以将新用户保存到数据库。由于每次测试之前都会自动保存用户，因此我们可以直接专注于测试与JPA Repository的交互：

```java
@Test
void givenUser_whenSaved_thenCanBeFoundById() {
    User savedUser = userRepository.findById(testUser.getId()).orElse(null);
    assertNotNull(savedUser);
    assertEquals(testUser.getUsername(), savedUser.getUsername());
    assertEquals(testUser.getPassword(), savedUser.getPassword());
}
```

如果我们观察测试用例的控制台日志，我们会注意到以下日志：

```text
Began transaction (1) for test context  
.....

Rolled back transaction for test:  
```

这些日志表明@BeforeEach和@AfterEach方法按预期运行。

### 6.4 测试更新操作

另外，我们可以创建一个测试用例，用于测试更新操作：

```java
@Test
void givenUser_whenUpdated_thenCanBeFoundByIdWithUpdatedData() {
    testUser.setUsername("updatedUsername");
    userRepository.save(testUser);

    User updatedUser = userRepository.findById(testUser.getId()).orElse(null);

    assertNotNull(updatedUser);
    assertEquals("updatedUsername", updatedUser.getUsername());
}
```

### 6.5 测试findByUsername()方法

现在，让我们测试一下我们创建的findByUsername()自定义查询方法：

```java
@Test
void givenUser_whenFindByUsernameCalled_thenUserIsFound() {
    User foundUser = userRepository.findByUsername("testuser");

    assertNotNull(foundUser);
    assertEquals("testuser", foundUser.getUsername());
}
```

## 7. 事务行为

默认情况下，所有使用@DataJpaTest标注的测试都在事务中执行，**这意味着在测试期间对数据库所做的任何更改都会在测试结束时回滚，从而确保数据库保持其原始状态**，此默认行为通过防止测试之间的干扰和数据损坏来简化测试。

然而，有时我们需要禁用事务行为来测试某些场景。例如，测试结果可能需要在测试结束后继续保留。

在这种情况下，我们可以使用@Transactional注解和[propagation = propagation.NOT_SUPPORTED](https://www.baeldung.com/spring-transactional-propagation-isolation#5notsupported-propagation)禁用特定测试类的事务：

```java
@DataJpaTest
@Transactional(propagation = Propagation.NOT_SUPPORTED)
public class UserRepositoryIntegrationTest {
    // ... test methods
}
```

或者我们可以针对单个测试方法禁用事务：

```java
@Test
@Transactional(propagation = Propagation.NOT_SUPPORTED)
public void testMyMethodWithoutTransactions() {
    // ... code that modifies the database
}
```

## 8. 总结

在本文中，我们学习了如何使用@DataJpaTest在JUnit中测试我们的JPA Repository。总体而言，@DataJpaTest是用于在Spring Boot应用程序中测试JPA Repository的强大注解，它提供了一个集中的测试环境和用于测试持久层的预配置工具。通过使用@DataJpaTest，我们可以确保我们的JPA Repository正常运行，而无需启动整个Spring上下文。