---
layout: post
title:  如何在测试后拆除或清空HSQLDB数据库
category: spring-test
copyright: spring-test
excerpt: Spring Test
---

## 1. 概述

编写集成测试时，在测试之间保持干净的数据库状态对于确保可重现的结果和避免意外的副作用至关重要。**HSQLDB([HyperSQL](https://www.baeldung.com/spring-boot-hsqldb)数据库)是一种轻量级的内存数据库，由于其速度快且简单，经常用于测试目的**。

在本文中，我们将探讨在测试运行后在Spring Boot中拆除或清空HSQLDB数据库的各种方法。虽然我们的重点是HSQLDB，但这些概念也广泛适用于其他数据库类型。

对于测试隔离，我们使用了[@DirtiesContext](https://www.baeldung.com/spring-dirtiescontext)注解，因为我们正在Mock不同的方法，并且我们希望确保每种方法都经过隔离测试，以避免它们之间产生任何副作用。

## 2. 方法

我们将探讨拆除HSQLDB数据库的五种方法，即：

- 使用Spring的@Transactional注解进行事务管理
- 应用程序属性配置
- 使用JdbcTemplate执行查询
- 使用JdbcTestUtils进行清理
- 自定义SQL脚本

### 2.1 使用@Transactional注解进行事务管理

当测试使用@Transactional注解时，[Spring的事务管理](https://www.baeldung.com/transaction-configuration-with-jpa-and-spring)将被激活。这意味着每个测试方法都在其自己的事务中运行，并自动回滚测试期间所做的任何更改：

```java
@SpringBootTest
@ActiveProfiles("hsql")
@Transactional
@DirtiesContext
public class TransactionalIntegrationTest {

    @Autowired
    private CustomerRepository customerRepository;

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @BeforeEach
    void setUp() {
        String insertSql = "INSERT INTO customers (name, email) VALUES (?, ?)";
        Customer customer = new Customer("John", "john@domain.com");
        jdbcTemplate.update(insertSql, customer.getName(), customer.getEmail());
    }

    @Test
    void givenCustomer_whenSaved_thenReturnSameCustomer() throws Exception {
        Customer customer = new Customer("Doe", "doe@domain.com");
        Customer savedCustomer = customerRepository.save(customer);
        Assertions.assertEquals(customer, savedCustomer);
    }
    // ... integration tests
}
```

**这种方法快速且有效，因为它不需要手动清理**。

### 2.2 应用程序属性配置

在此方法中，我们有一个简单的设置，其中我们为属性spring.jpa.hibernate.ddl-auto分配一个首选选项。有不同的值可用，但create和create-drop通常对于清理最有用。

**create选项会在会话开始时删除并重新创建模式，而create-drop则会在开始时创建模式并在会话结束时删除它**。

在我们的代码示例中，我们为每个测试方法添加了@DirtiesContext注解，以确保每个测试都会触发模式创建和删除过程。然而，能力越大，责任越大；我们必须明智地使用这些选项，否则我们可能会意外清除生产数据库：

```java
@SpringBootTest
@ActiveProfiles("hsql")
@TestPropertySource(properties = { "spring.jpa.hibernate.ddl-auto=create-drop" })
public class PropertiesIntegrationTest {

    @Autowired
    private CustomerRepository customerRepository;

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @BeforeEach
    void setUp() {
        String insertSql = "INSERT INTO customers (name, email) VALUES (?, ?)";
        Customer customer = new Customer("John", "john@domain.com");
        jdbcTemplate.update(insertSql, customer.getName(), customer.getEmail());
    }

    @Test
    @DirtiesContext
    void givenCustomer_whenSaved_thenReturnSameCustomer() throws Exception {
        Customer customer = new Customer("Doe", "doe@domain.com");
        Customer savedCustomer = customerRepository.save(customer);
        Assertions.assertEquals(customer, savedCustomer);
    }

    // ... integration tests
}
```

**此方法的优点是配置简单，无需在测试类中编写自定义代码，对于需要持久化数据的测试尤其有用**。但是，与其他涉及自定义代码的方法相比，它提供的控制粒度较差。

我们可以在测试类中设置此属性，也可以将其分配给整个Profile，但我们必须小心，因为它会影响同一Profile内的所有测试，这可能并不总是理想的。

### 2.3 使用JdbcTemplate执行查询

在这个方法中，我们使用[JdbcTemplate](https://www.baeldung.com/spring-jdbctemplate-testing) API，它允许我们执行各种查询。具体来说，我们可以使用它在每次测试后或一次在所有测试后截断表。**这种方法简单明了，因为我们只需要编写查询，它就会保留表结构**：

```java
@SpringBootTest
@ActiveProfiles("hsql")
@DirtiesContext
class JdbcTemplateIntegrationTest {

    @Autowired
    private CustomerRepository customerRepository;

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @AfterEach
    void tearDown() {
        this.jdbcTemplate.execute("TRUNCATE TABLE customers");
    }

    @Test
    void givenCustomer_whenSaved_thenReturnSameCustomer() throws Exception {
        Customer customer = new Customer("Doe", "doe@domain.com");
        Customer savedCustomer = customerRepository.save(customer);
        Assertions.assertEquals(customer, savedCustomer);
    }
    // ... integration tests
}
```

但是这种方法需要手动列出或者编写所有表的查询，处理大量表时可能会比较慢。

### 2.4 使用JdbcTestUtils进行清理

在这个方法中，Spring Boot框架中的JdbcTestUtils类提供了对数据库清理过程的更多控制，**它允许我们执行任意SQL语句来从特定表中删除数据**。

虽然它与JdbcTemplate类似，但还是存在一些关键差异，特别是在语法简单性方面，因为一行代码可以处理多个表。它也很灵活，但仅限于从表中删除所有行：

```java
@SpringBootTest
@ActiveProfiles("hsql")
@DirtiesContext
public class JdbcTestUtilsIntegrationTest {

    @Autowired
    private CustomerRepository customerRepository;

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @AfterEach
    void tearDown() {
        JdbcTestUtils.deleteFromTables(jdbcTemplate, "customers");
    }

    @Test
    void givenCustomer_whenSaved_thenReturnSameCustomer() throws Exception {
        Customer customer = new Customer("Doe", "doe@domain.com");
        Customer savedCustomer = customerRepository.save(customer);
        Assertions.assertEquals(customer, savedCustomer);
    }

    // ... integration tests
}
```

但是，这种方法也有缺点。对于大型数据库，删除操作比截断操作速度慢，并且**它不处理引用完整性约束**，这可能会导致具有外键关系的数据库出现问题。

### 2.5 自定义SQL脚本

此方法使用SQL脚本来清理数据库，这对于更复杂的设置尤其有用。**它既灵活又可定制，因为它可以处理复杂的清理场景，同时将数据库逻辑与测试代码分开**。

在Spring Boot中，SQL脚本可以放在类级别，在每个方法之后或所有测试方法之后执行。例如，我们来看一种配置方法：

```java
@SpringBootTest
@ActiveProfiles("hsql")
@DirtiesContext
@Sql(scripts = "/cleanup.sql", executionPhase = Sql.ExecutionPhase.AFTER_TEST_METHOD)
public class SqlScriptIntegrationTest {

    @Autowired
    private CustomerRepository customerRepository;

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @BeforeEach
    void setUp() {
        String insertSql = "INSERT INTO customers (name, email) VALUES (?, ?)";
        Customer customer = new Customer("John", "john@domain.com");
        jdbcTemplate.update(insertSql, customer.getName(), customer.getEmail());
    }

    @Test
    void givenCustomer_whenSaved_thenReturnSameCustomer() throws Exception {
        Customer customer = new Customer("Doe", "doe@domain.com");
        Customer savedCustomer = customerRepository.save(customer);
        Assertions.assertEquals(customer, savedCustomer);
    }
}
```

它也可以应用于单独的测试方法，如下例所示，我们将它放在最后一个方法上，以便在所有测试之后进行清理：

```java
@Test
@Sql(scripts = "/cleanup.sql", executionPhase = Sql.ExecutionPhase.AFTER_TEST_METHOD)
void givenExistingCustomer_whenFoundByName_thenReturnCustomer() throws Exception {
    // ... integration test
}
```

但是，这种方法需要维护单独的SQL文件，这对于更简单的用例来说可能有点过度。

## 3. 总结

每次测试后清理数据库是软件测试的一个基本概念。通过遵循本文概述的方法，我们可以确保测试保持独立，并在每次运行时产生可靠的结果。

虽然我们在此讨论的每种方法都有其优点和缺点，但选择最适合我们项目特定要求的方法非常重要。