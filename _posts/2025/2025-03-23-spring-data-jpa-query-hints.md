---
layout: post
title:  Spring Data JPA中的查询提示
category: springdata
copyright: springdata
excerpt: Spring Data JPA
---

## 1. 简介

在本教程中，我们将探讨[Spring Data JPA](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)中查询提示的基础知识。这些提示有助于优化数据库查询，并通过影响优化器的决策过程来潜在地提高应用程序性能。我们还将讨论它们的功能以及如何有效地应用它们。

## 2. 理解查询提示

**Spring Data JPA中的查询提示是一个强大的工具，可以帮助优化数据库查询并提高应用程序性能**。与直接控制执行不同，它们会影响优化器的决策过程。

在Spring Data JPA中，我们在org.hibernate.annotations包中可以找到这些提示，以及与流行的持久化提供程序[Hibernate](https://www.baeldung.com/learn-jpa-hibernate)相关的各种注解和类。**值得注意的是，这些提示的解释和执行通常取决于底层持久化提供程序，例如Hibernate或EclipseLink，这使得它们特定于供应商**。

## 3. 使用查询提示

Spring Data JPA提供了多种利用查询提示来优化数据库查询的方法，让我们探索一下常见的方法。

### 3.1 基于注解的配置

Spring Data JPA提供了一种使用注解向JPA查询添加查询提示的简单方法，**@QueryHints注解允许指定一组JPA@QueryHint提示，用于应用于生成的SQL查询**。

让我们考虑以下示例，其中我们设置JDBC获取大小提示来限制结果返回大小：

```java
@Repository
public interface EmployeeRepository extends JpaRepository<Employee, Long> {
    @QueryHints(value = { @QueryHint(name = "org.hibernate.fetchSize", value = "50") })
    List<Employee> findByGender(String gender);
}
```

在此示例中，我们在EmployeeRepository接口的findByGender()方法中添加了@QueryHints注解，以控制一次获取的实体数量。此外，我们可以在Repository级别应用@QueryHints注解来影响Repository中的所有查询：

```java
@Repository
@QueryHints(value = { @QueryHint(name = "org.hibernate.fetchSize", value = "50") })
public interface EmployeeRepository extends JpaRepository<Employee, Long> {
    // Repository methods...
}
```

此操作可确保指定的查询提示适用于EmployeeRepository接口内的所有查询，从而提高Repository查询的一致性。

### 3.2 以编程方式配置查询提示

除了基于注解和动态的方法之外，我们还可以使用[EntityManager](https://www.baeldung.com/hibernate-entitymanager)对象以编程方式配置查询提示，此方法可以对查询提示配置进行精细控制。以下是以编程方式设置自定义SQL注解提示的示例：

```java
@PersistenceContext
private EntityManager entityManager;

@Override
List<Employee> findRecentEmployees(int limit, boolean readOnly) {
    Query query = entityManager.createQuery("SELECT e FROM Employee e ORDER BY e.joinDate DESC", Employee.class)
        .setMaxResults(limit)
        .setHint("org.hibernate.readOnly", readOnly);
    return query.getResultList();
}
```

在此示例中，我们传递一个boolean标志作为参数，以指示提示是否应设置为true或false。这种灵活性使我们能够根据运行时条件调整查询行为。

### 3.3 在实体中定义命名查询

**可以使用[@NamedQuery](https://www.baeldung.com/hibernate-named-query)注解直接在Entity类中应用查询提示**，这允许我们定义命名查询以及特定提示。例如，让我们考虑以下代码片段：

```java
@Entity
@NamedQuery(name = "selectEmployee", query = "SELECT e FROM Employee e", hints = @QueryHint(name = "org.hibernate.fetchSize", value = "50"))
public class Employee {
    // Entity properties and methods
}
```

一旦在Entity类中定义，就可以使用EntityManager的createNamedQuery()方法调用带有相关提示的命名查询selectEmployee：

```java
List<Employee> employees = em.createNamedQuery("selectEmployee").getResultList();
```

## 4. 查询提示使用场景

查询提示可用于多种场景以优化查询性能，以下是一些常见的用例。

### 4.1 超时管理

在查询可能长时间运行的情况下，实施有效的超时管理策略变得至关重要。**通过利用javax.persistence.query.timeout提示，我们可以为查询设定最大执行时间，此做法可确保查询不超过指定的时间阈值**。

提示接收以毫秒为单位的值，如果查询超出限制，则会引发LockTimeoutException。以下是一个例子，我们设置了5000毫秒的超时来检索活跃员工：

```java
@QueryHints(value = {@QueryHint(name = "javax.persistence.query.timeout", value = "5000")})
List<Employee> findActiveEmployees(long inactiveDaysThreshold);
```

### 4.2 缓存查询结果

**查询提示可用于通过jakarta.persistence.cache.retrieveMode提示启用查询结果缓存**。设置为USE时，JPA会先尝试从缓存中检索实体，然后再转至数据库。另一方面，将其设置为BYPASS会指示JPA忽略缓存并直接从数据库获取实体。

**此外，我们还可以使用jakarta.persistence.cache.storeMode来指定JPA应如何处理在二级缓存中存储实体**。设置为USE时，JPA会将实体添加到缓存中并更新现有实体。BYPASS模式指示Hibernate仅更新缓存中的现有实体，而不添加新实体。最后，REFRESH模式在检索缓存中的实体之前刷新它们，确保缓存的数据是最新的。

下面是演示如何使用这些提示的示例：

```java
@QueryHints(value = {
    @QueryHint(name = "jakarta.persistence.cache.retrieveMode", value = "USE"),
    @QueryHint(name = "jakarta.persistence.cache.storeMode", value = "USE")
})
List<Employee> findEmployeesByName(String name);
```

在这种情况下，retrieveMode和storeMode都配置为USE，表明Hibernate主动利用二级缓存来检索和存储实体。

### 4.3 优化查询执行计划

**查询提示可用于影响数据库优化器生成的执行计划**。例如，当数据保持不变时，我们可以使用org.hibernate.readOnly提示来表示查询是只读的：

```java
@QueryHints(@QueryHint(name = "org.hibernate.readOnly", value = "true"))
User findByUsername(String username);
```

### 4.4 自定义SQL注解

**org.hibernate.comment提示允许向查询添加自定义SQL注解，帮助进行查询分析和调试**。当我们想在生成的SQL语句中提供上下文或注解时，此功能特别有用。

以下是一个例子：

```java
@QueryHints(value = { @QueryHint(name = "org.hibernate.comment", value = "Retrieve employee older than specified age\"") })
List findByAgeGreaterThan(int age);
```

## 5. 总结

在本文中，我们了解了Spring Data JPA中查询提示的重要性及其对优化数据库查询以提高应用程序性能的重大影响。我们探索了各种技术，包括基于注解的配置和直接JPQL操作，以有效地应用查询提示。