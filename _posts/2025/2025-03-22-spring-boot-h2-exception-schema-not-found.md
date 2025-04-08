---
layout: post
title:  修复Spring Boot H2异常：“Schema not found”
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 概述

[H2](https://www.baeldung.com/spring-boot-h2-database)是Java社区中经常用于测试目的的开源SQL数据库，它具有内存特性，不会将任何内容持久保存到磁盘，因此速度非常快。

当我们将其与Spring Boot集成时，我们可能会遇到错误消息“Schema not found”。在本教程中，我们将探讨其原因并研究两种不同的解决方法。

## 2. 了解原因

**H2的默认模式是PUBLIC**，如果我们映射不使用PUBLIC模式的[JPA实体](https://www.baeldung.com/jpa-entities)类，则必须确保在H2上创建该模式。当目标模式不存在时，Spring Boot会报告错误消息“Schema not found”。

为了模拟该场景，让我们在Spring Boot应用程序中创建以下实体类和[Repository](https://www.baeldung.com/spring-data-repositories)。@Table注解指定实体映射到test模式中的student表的表映射详细信息：

```java
@Entity
@Table(name = "student", schema = "test")
public class Student {
    @Id
    @Column(name = "student_id", length = 10)
    private String studentId;

    @Column(name = "name", length = 100)
    private String name;

    // constructor, getters and setters
}
```

```java
public interface StudentRepository extends JpaRepository<Student, String> {
}
```

接下来，我们启动Spring Boot应用程序并访问Repository。我们将遇到一个抛出的异常，表明模式不存在。我们可以通过集成测试来验证这一点：

```java
@ExtendWith(SpringExtension.class)
@SpringBootTest(classes = SampleSchemaApplication.class)
class SampleSchemaApplicationIntegrationTest {
    @Autowired
    private StudentRepository studentRepository;

    @Test
    void whenSaveStudent_thenThrowsException() {
        Student student = Student.builder()
                .studentId("24567433")
                .name("David Lloyds")
                .build();

        assertThatThrownBy(() -> studentRepository.save(student))
                .isInstanceOf(InvalidDataAccessResourceUsageException.class);
    }
}
```

测试执行后，我们会在控制台中看到以下错误消息：

```text
org.hibernate.exception.SQLGrammarException: could not prepare statement [Schema "TEST" not found; SQL statement:
select s1_0.student_id,s1_0.name from test.student s1_0 where s1_0.student_id=? [90079-214]] [select s1_0.student_id,s1_0.name from test.student s1_0 where s1_0.student_id=?]
```

## 3. 通过数据库URL创建模式

为了解决这个问题，我们必须在Spring Boot应用程序启动时创建相应的模式，有两种不同的方法可以做到这一点。

**第一种方法是建立数据库连接时创建数据库模式**，当客户端通过INIT属性连接到数据库时，H2数据库URL允许我们执行[DDL或DML](https://www.baeldung.com/sql/ddl-dml-dcl-tcl-differences)命令。在Spring Boot应用程序中，我们可以在application.yaml中定义spring.datasource.url属性：

```yaml
spring:
    jpa:
        hibernate:
            ddl-auto: create
    datasource:
        driverClassName: org.h2.Driver
        url: jdbc:h2:mem:test;INIT=CREATE SCHEMA IF NOT EXISTS test
```

如果不存在模式，初始化DDL将创建模式。值得注意的是，这种方法是H2数据库的专用方法，可能不适用于其他数据库。

此方法通过数据库URL创建模式，而无需明确创建表。我们依靠Hibernate中的自动模式创建，方法是在YAML文件中设置spring.jpa.hibernate.ddl-auto属性来创建。

## 4. 通过初始化脚本创建模式

第二种方法是通用的，也可以应用于其他数据库。我们通过初始化脚本创建所有数据库组件，包括模式和表。

**Spring Boot在执行初始化脚本之前会初始化JPA持久层单元**。因此，我们在application.yaml中明确禁用Hibernate的自动模式生成，因为我们的初始化脚本会处理它：

```yaml
spring:
    jpa:
        hibernate:
            ddl-auto: none
```

如果我们不通过将ddl-auto从create改为none来禁用它，我们会在应用程序启动期间遇到”Schema TEST not found”的异常。在JPA持久层单元初始化期间，该模式尚未创建。

现在，我们可以将创建test模式和student表的schema.sql放在resources文件夹中：

```sql
CREATE SCHEMA IF NOT EXISTS test;

CREATE TABLE test.student (
  student_id VARCHAR(10) PRIMARY KEY,
  name VARCHAR(100)
);
```

**Spring Boot在应用程序启动时默认寻找resources文件夹中的DDL脚本schema.sql来[初始化数据库](https://www.baeldung.com/java-h2-db-execute-sql-file)**。

## 5. 总结

”Schema not found”异常是Spring Boot应用程序启动与H2数据库集成时常见的问题，我们可以通过确保通过数据库URL配置或初始化脚本创建架构来避免这些异常。