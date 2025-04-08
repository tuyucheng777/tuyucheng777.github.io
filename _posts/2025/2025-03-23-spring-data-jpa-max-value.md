---
layout: post
title:  在Spring Data JPA中查找最大值
category: springdata
copyright: springdata
excerpt: Spring Data JPA
---

## 1. 简介

使用[Spring Data JPA](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)时，在数据库中查找特定值是一项常见任务，其中一项任务是在特定列中查找最大值。

在本教程中，我们将探索使用Spring Data JPA实现此目的的几种方法。我们将检查**如何使用Repository方法、JPQL和原生查询以及Criteria API来查找数据库列中的最大值**。

## 2. 实体示例

在继续之前，我们必须为我们的项目添加必需的[spring-boot-starter-data-jpa](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-jpa)依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

之后，让我们定义一个简单的实体来使用：

```java
@Entity
public class Employee {
    @Id
    @GeneratedValue
    private Integer id;
    private String name;
    private Long salary;

    // constructors, getters, setters, equals, hashcode
}
```

在下面的例子中，我们将使用不同的方法找出所有员工salary列的最大值。

## 3. 在Repository中使用派生查询

Spring Data JPA提供了一种强大的机制来使用Repository方法定义自定义查询，**这些机制之一是[派生查询](https://www.baeldung.com/spring-data-derived-queries)，它允许我们通过声明方法名称来实现SQL查询**。

让我们为Employee类创建一个Repository：

```java
public interface EmployeeRepository extends JpaRepository<Employee, Integer> {
    Optional<Employee> findTopByOrderBySalaryDesc();
}
```

我们刚刚实现了一种使用查询派生机制来生成适当SQL的方法，根据方法名称，我们将所有员工按salary降序排序，然后返回第一个，即salary最高的员工。

值得注意的是，**此方法始终返回已设置所有急切属性的实体。但是，如果我们只想检索单个salary值，我们可以通过实现[投影功能](https://www.baeldung.com/spring-data-jpa-projections)稍微修改代码**。

我们来创建一个额外接口并修改Repository：

```java
public interface EmployeeSalary {
    Long getSalary();
}
```

```java
public interface EmployeeRepository extends JpaRepository<Employee, Integer> {
    Optional<EmployeeSalary> findTopSalaryByOrderBySalaryDesc(); 
}
```

**当我们只需要返回实体的特定部分时，此解决方案很有用**。

## 4. 使用JPQL

另一种直接的方法是使用@Query注解，**这让我们可以直接在Repository接口中定义自定义[JPQL](https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-query-4)(Java持久化查询语言)查询**。

让我们实现JQPL查询来检索最高salary值：

```java
public interface EmployeeRepository extends JpaRepository<Employee, Integer> {
    @Query("SELECT MAX(e.salary) FROM Employee e")
    Optional<Long> findTopSalaryJQPL();
}
```

与之前一样，该方法返回所有员工中最高的salary值。此外，我们**可以轻松检索实体的单个列，而无需任何额外的投影**。

## 5. 使用原生查询

我们刚刚在Repository中引入了@Query注解，**这种方法还允许我们使用原生查询直接编写原始SQL**。

为了达到相同的结果，我们可以实现：

```java
public interface EmployeeRepository extends JpaRepository<Employee, Integer> {
    @Query(value = "SELECT MAX(salary) FROM Employee", nativeQuery = true)
    Optional<Long> findTopSalaryNative();
}
```

该解决方案类似于JPQL，使用原生查询**可以有助于利用特定的SQL功能或优化**。

## 6. 实现默认Repository方法

我们还可以使用自定义Java代码来查找最大值，让我们实现另一种解决方案，而无需添加额外的查询：

```java
public interface EmployeeRepository extends JpaRepository<Employee, Integer> {
    default Optional<Long> findTopSalaryCustomMethod() {
        return findAll().stream()
                .map(Employee::getSalary)
                .max(Comparator.naturalOrder());
    }
}
```

我们通过添加具有自定义逻辑的新默认方法扩展了Repository，我们使用内置的findAll()方法检索所有Employee实体，然后对其进行流式传输并查找最高salary。**与以前的方法不同，所有过滤逻辑都发生在应用程序层，而不是数据库层**。

## 7. 使用分页

**Spring Data JPA提供了对[分页和排序功能](https://www.baeldung.com/spring-data-jpa-pagination-sorting)的支持**，我们仍然可以使用它们来查找员工的最高salary。

即使不实现任何专用查询或扩展Repository，我们也可以实现我们的目标：

```java
public Optional<Long> findTopSalary() {
    return findAll(PageRequest.of(0, 1, Sort.by(Sort.Direction.DESC, "salary")))
            .stream()
            .map(Employee::getSalary)
            .findFirst();
}
```

我们知道，[PagingAndSortingRepository](https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/repository/PagingAndSortingRepository.html)接口为[Pageable](https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/domain/Pageable.html)和[Sort](https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/domain/Sort.html)类型提供了额外的支持。因此，JpaRepository中内置的findAll()方法也可以接收这些参数。**我们只是实现了不同的方法，而无需在Repository中添加其他方法**。

## 8. 使用Criteria API

**Spring Data JPA还提供了[Criteria API](https://www.baeldung.com/hibernate-criteria-queries)，这是一种更具编程性的查询构建方法**。这是一种更动态且类型安全的方法，无需使用原始SQL即可构建复杂查询。

首先，让我们将EntityManager Bean注入到我们的服务中，然后创建一个使用Criteria API来查找最高salary的方法：

```java
@Service
public class EmployeeMaxValueService {
    @Autowired
    private EntityManager entityManager;

    public Optional<Long> findMaxSalaryCriteriaAPI() {
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery<Long> query = cb.createQuery(Long.class);

        Root<Employee> root = query.from(Employee.class);
        query.select(cb.max(root.get("salary")));

        TypedQuery<Long> typedQuery = entityManager.createQuery(query);
        return Optional.ofNullable(typedQuery.getSingleResult());
    }
}
```

在这个方法中，我们首先从注入的EntityManager Bean中获取一个CriteriaBuilder实例，然后我们创建一个CriteriaBuilder来指定结果类型，并创建一个Root来定义FROM子句。最后，我们选择salary字段的最大值并执行查询。

再次，我们刚刚检索了所有员工的最高salary。但是，**这种方法比前一种方法更复杂，因此如果我们需要实现一个简单的查询，它可能有点难以应付**。如果我们有更复杂的结构，无法通过简单地扩展Repository来处理，则此解决方案可能很有用。

## 9. 总结

在本文中，我们探索了使用Spring Data JPA查找列最大值的各种方法。

**我们从派生查询开始，它提供了一种简单直观的方法，只需通过方法命名约定即可定义查询**。然后，我们研究了**使用JPQL和带有@Query注解的原生查询，从而提供更大的灵活性和对正在执行的SQL的直接控制**。

我们还**在Repository中实现了自定义默认方法，以利用Java的Stream API在应用程序级别处理数据**。此外，我们还检查了如何使用分页和排序仅使用内置API来查找结果。

最后，我们**利用Criteria API以更具编程性和类型安全性的方法来构建复杂查询**。通过了解这些不同的方法，我们可以为特定用例选择最合适的方法，在简单性、控制和性能之间取得平衡。