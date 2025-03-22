---
layout: post
title:  Spring Boot H2 JdbcSQLSyntaxErrorException expected “identifier”
category: spring-boot
copyright: spring-boot
excerpt: Spring Boot
---

## 1. 概述

在这个简短的教程中，我们将仔细研究异常org.h2.jdbc.JdbcSQLSyntaxErrorException：Syntax error in SQL statement expected “identifier”。

首先，我们将阐明导致异常的主要原因。然后，我们将使用实际示例说明如何重现该异常，最后说明如何解决该异常。

## 2. 原因

在进入解决方案之前，让我们先了解一下异常。

通常，[H2](https://www.baeldung.com/spring-boot-h2-database)会抛出JdbcSQLSyntaxErrorException来表示SQL语句中的语法错误。因此，“expected identifier”消息表示SQL需要一个合适的[标识符](https://www.ibm.com/docs/en/informix-servers/12.10?topic=information-sql-identifiers)，而我们未能提供。

**导致此异常的最常见原因是使用保留关键字作为标识符**。

例如，使用关键字table来命名特定的SQL表将导致JdbcSQLSyntaxErrorException。

另一个原因可能是SQL语句中缺少或放错关键字。

## 3. 重现异常

作为开发人员，我们经常使用“user”一词来表示处理用户的表。不幸的是，它是H2中的[保留关键字](http://www.h2database.com/html/advanced.html#keywords)。

因此，为了重现异常，我们假装使用关键字“user”。

因此，首先，让我们添加一个基本的SQL脚本来初始化H2数据库并用数据填充：

```sql
INSERT INTO user VALUES (1, 'admin', 'p@ssw@rd'); 
INSERT INTO user VALUES (2, 'user', 'userpasswd');
```

接下来，我们将创建一个[实体类](https://www.baeldung.com/jpa-entities)来映射user表：

```java
@Entity
public class User {

    @Id
    private int id;
    private String login;
    private String password;

    // standard getters and setters
}
```

请注意，@Entity是一个JPA注解，用于将某个类标识为实体类。

此外，@Id表示映射数据库中主键的字段。

现在，如果我们运行主应用程序，Spring Boot将失败并出现以下情况：

```text
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'dataSourceScriptDatabaseInitializer' 
...
nested exception is org.h2.jdbc.JdbcSQLSyntaxErrorException: Syntax error in SQL statement "INSERT INTO [*]user VALUES (1, 'admin', 'p@ssw@rd')"; expected "identifier"; SQL statement:
INSERT INTO user VALUES (1, 'admin', 'p@ssw@rd') [42001-214]
...
```

正如我们在日志中看到的，H2抱怨插入查询，因为关键字user是保留的，不能用作标识符。

## 4. 解决方案

为了修复异常，我们需要确保没有使用SQL保留关键字作为标识符。

或者，我们可以使用分隔符来转义它们。H2支持双引号作为标准标识符分隔符。

首先，让我们用双引号括住关键字user：

```sql
INSERT INTO "user" VALUES (1, 'admin', 'p@ssw@rd');
INSERT INTO "user" VALUES (2, 'user', 'userpasswd');
```

接下来，我们将为实体User创建一个[JPA Repository](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)：

```java
@Repository
public interface UserRepository extends JpaRepository<User, Integer> {
}
```

现在，让我们添加一个测试用例来确认一切是否按预期工作：

```java
@Test
public void givenValidInitData_whenCallingFindAll_thenReturnData() {
    List<User> users = userRepository.findAll();

    assertThat(users).hasSize(2);
}
```

如上所示，findAll()完成了其工作并且没有因JdbcSQLSyntaxErrorException而失败。

避免异常的另一种解决方法是将NON_KEYWORDS=user附加到JDBC URL：

```properties
spring.datasource.url=jdbc:h2:mem:mydb;NON_KEYWORDS=user
```

这样，**我们告诉H2将user关键字从保留字列表中排除**。

如果我们使用Hibernate，我们可以将hibernate.globally_quoted_identifiers属性设置为true。

```properties
spring.jpa.properties.hibernate.globally_quoted_identifiers=true
```

正如属性名称所暗示的，Hibernate将自动引用所有数据库标识符。

话虽如此，**在使用@Table或@Column注解时我们不需要在手动转义表名或列名**。

简而言之，以下是需要考虑的一些重要关键点：

- 确保使用正确的SQL关键字并按正确的顺序排列
- 避免使用保留关键字作为标识符
- 仔细检查SQL中不允许的任何特殊字符
- 确保正确转义或引用任何保留关键字

## 5. 总结

在本文中，我们详细解释了异常org.h2.jdbc.JdbcSQLSyntaxErrorException: Syntax error in SQL statement expected “identifier”背后的原因。然后，我们展示了如何产生异常以及如何修复它。