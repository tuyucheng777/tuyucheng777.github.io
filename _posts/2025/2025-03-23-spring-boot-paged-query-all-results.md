---
layout: post
title:  使用Spring Boot分页查询方法一次性获取所有结果
category: spring-data
copyright: spring-data
excerpt: Spring Data JPA
---

## 1. 概述

在Spring Boot应用程序中，我们经常需要一次向客户端呈现20或50行的表数据，**[分页](https://www.baeldung.com/spring-data-jpa-pagination-sorting)是从大型数据集中返回部分数据的常见做法**。但是，有些情况下我们需要一次获取整个结果。

在本教程中，我们将首先回顾如何使用Spring Boot在分页中检索数据。接下来，我们将探索如何使用分页一次从一个数据库表中检索所有结果。最后，我们将深入研究一个更复杂的场景，即检索具有关系的数据。

## 2. Repository

**Repository是一个[Spring Data](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)接口，提供数据访问抽象。根据我们选择的Repository子接口，抽象提供了一组预定义的数据库操作**。

我们不需要为标准数据库操作(如选择、保存和删除)编写代码，我们需要做的就是为我们的实体创建一个接口，并将其扩展到所选的Repository子接口。

在运行时，Spring Data会创建一个代理实现来处理Repository的方法调用。当我们调用Repository接口上的方法时，Spring Data会根据方法和参数动态生成查询。

Spring Data中定义了三个常见的Repository[子接口](https://www.baeldung.com/spring-data-repositories)：

- CrudRepository：Spring Data提供的最基本的Repository接口，它提供CRUD(创建、读取、更新和删除)实体操作
- PagingAndSortingRepository：它扩展了CrudRepository接口，并添加了额外的方法来轻松支持分页访问和结果排序
- JpaRepository：它扩展了PagingAndSortingRepository接口并引入了JPA特定的操作，例如保存和刷新实体以及批量删除实体

## 3. 获取分页数据

我们先从一个简单的场景开始，使用分页从数据库获取数据。我们首先创建一个Student实体类：

```java
@Entity
@Table(name = "student")
public class Student {
    @Id
    @Column(name = "student_id")
    private String id;

    @Column(name = "first_name")
    private String firstName;

    @Column(name = "last_name")
    private String lastName;
}
```

随后，我们将创建一个StudentRepository，用于从数据库中检索Student实体。**JpaRepository接口默认包含方法findAll(Pageable pageable)**。因此，我们不需要定义其他方法，因为我们只想检索页面中的数据而不选择字段：

```java
public interface StudentRepository extends JpaRepository<Student, String> {
}
```

我们可以通过在StudentRepository上调用findAll(Pageable)来获取Student的第一页，每页有10行。第一个参数表示当前页面，即0索引，而第二个参数表示每页获取的记录数：

```java
Pageable pageable = PageRequest.of(0, 10);
Page<Student> studentPage = studentRepository.findAll(pageable);
```

**我们经常需要返回按特定字段排序的分页结果，在这种情况下，我们在创建Pageable实例时会提供一个Sort实例**。此示例显示我们将按Student的id字段按升序对分页结果进行排序：

```java
Sort sort = Sort.by(Sort.Direction.ASC, "id");
Pageable pageable = PageRequest.of(0, 10).withSort(sort);
Page<Student> studentPage = studentRepository.findAll(pageable);
```

## 4. 获取所有数据

经常会出现一个常见问题：如果我们想一次性检索所有数据怎么办？我们是否需要调用findAll()来获取所有数据？答案是否定的。**Pageable接口定义了一个静态方法unpaged()，它返回一个不包含分页信息的预定义Pageable实例**。我们通过使用该Pageable实例调用findAll(Pageable)来获取所有数据：

```java
Page<Student> studentPage = studentRepository.findAll(Pageable.unpaged());
```

如果我们需要对结果进行排序，**我们可以从Spring Boot 3.2开始将Sort实例作为参数提供给unpaged()方法**。例如，假设我们想按lastName字段按升序对结果进行排序：

```java
Sort sort = Sort.by(Sort.Direction.ASC, "lastName");
Page<Student> studentPage = studentRepository.findAll(Pageable.unpaged(sort));
```

但是，**在3.2以下的版本中实现同样的功能有点棘手，因为unpaged()不接收任何参数**。相反，我们必须创建一个具有最大页面大小和Sort参数的PageRequest：

```java
Pageable pageable = PageRequest.of(0, Integer.MAX_VALUE).withSort(sort);
Page<Student> studentPage = studentRepository.getStudents(pageable);
```

## 5. 获取具有关系的数据

我们经常在对象关系映射(ORM)框架中定义实体之间的关系，利用JPA等ORM框架可帮助开发人员快速建模实体和关系，而无需编写SQL查询。

但是，如果我们不彻底了解数据检索的底层工作原理，则可能会出现一个问题。在尝试从具有关系的实体中检索结果集合时，我们必须小心谨慎，因为这可能会导致性能影响，尤其是在获取所有数据时。

### 5.1 N+1问题

让我们举一个例子来说明这个问题，考虑我们的Student实体，其中有一个额外的多对一映射：

```java
@Entity
@Table(name = "student")
public class Student {
    @Id
    @Column(name = "student_id")
    private String id;

    @Column(name = "first_name")
    private String firstName;

    @Column(name = "last_name")
    private String lastName;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "school_id", referencedColumnName = "school_id")
    private School school;

    // getters and setters
}
```

现在每个学生都与一所学校相关联，我们将学校实体定义为：

```java
@Entity
@Table(name = "school")
public class School {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "school_id")
    private Integer id;

    private String name;

    // getters and setters
}
```

现在，我们想从数据库中检索所有学生记录，并调查JPA发出的实际SQL查询数量。[Hypersistence Utilities](https://mvnrepository.com/artifact/io.hypersistence/hypersistence-utils-hibernate-62)是一个数据库实用程序库，它提供assertSelectCount()方法来识别执行的选择查询的数量。让我们在pom.xml文件中包含它的Maven依赖：

```xml
<dependency>
    <groupId>io.hypersistence</groupId>
    <artifactId>hypersistence-utils-hibernate-62</artifactId>
    <version>3.7.0</version>
</dependency>
```

现在，我们创建一个测试用例来检索所有学生记录：

```java
@Test
public void whenGetStudentsWithSchool_thenMultipleSelectQueriesAreExecuted() {
    Page<Student> studentPage = studentRepository.findAll(Pageable.unpaged());
    List<StudentWithSchoolNameDTO> list = studentPage.get()
        .map(student -> modelMapper.map(student, StudentWithSchoolNameDTO.class))
        .collect(Collectors.toList());
    assertSelectCount((int) studentPage.getContent().size() + 1);
}
```

**在一个完整的应用中，我们并不想将内部实体暴露给客户端。在实际应用中，我们会将内部实体映射到外部[DTO](https://www.baeldung.com/java-dto-pattern)并返回给客户端**。在这个例子中，我们采用[ModelMapper](https://www.baeldung.com/java-modelmapper)将Student转换为StudentWithSchoolNameDTO，其中包含Student的所有字段以及School的name字段：

```java
public class StudentWithSchoolNameDTO {
    private String id;
    private String firstName;
    private String lastName;
    private String schoolName;

    // constructor, getters and setters
}
```

我们来观察一下执行测试用例后的Hibernate日志：

```text
Hibernate: select studentent0_.student_id as student_1_1_, studentent0_.first_name as first_na2_1_, studentent0_.last_name as last_nam3_1_, studentent0_.school_id as school_i4_1_ from student studentent0_
Hibernate: select schoolenti0_.school_id as school_i1_0_0_, schoolenti0_.name as name2_0_0_ from school schoolenti0_ where schoolenti0_.school_id=?
Hibernate: select schoolenti0_.school_id as school_i1_0_0_, schoolenti0_.name as name2_0_0_ from school schoolenti0_ where schoolenti0_.school_id=?
...
```

假设我们从数据库中检索了N条学生记录，JPA不会在Student表上执行单个选择查询，而是在School表上执行额外的N条查询来获取每个Student的相关记录。

**这种行为在ModelMapper尝试读取Student实例中的school字段时进行转换时出现，对象关系映射性能中的这一问题被称为N+1问题**。

值得一提的是，JPA并不总是在每次提取Student时对School表发出N个查询，实际计数取决于数据。JPA具有一级缓存机制，可确保它不会再次从数据库中提取缓存的School实例。

### 5.2 避免获取关系

当将DTO返回给客户端时，并不总是需要包含实体类中的所有字段。大多数情况下，我们只需要其中的一个子集。**为了避免触发实体中关联关系的额外查询，我们应该仅提取必要的字段**。

在我们的示例中，我们可以创建一个指定的DTO类，其中仅包含来自Student表的字段。如果我们不访问school字段，JPA将不会对School执行任何其他查询：

```java
public class StudentDTO {
    private String id;
    private String firstName;
    private String lastName;

    // constructor, getters and setters
}
```

**这种方法假设我们查询的实体类上定义的关联获取类型设置为对关联实体执行惰性获取**：

```java
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "school_id", referencedColumnName = "school_id")
private School school;
```

值得注意的是，**如果将fetch属性设置为FetchType.EAGER，JPA将在获取Student记录时主动执行其他查询，尽管之后没有访问该字段**。

### 5.3 自定义查询

每当School中的某个字段是DTO中的必需字段时，**我们就可以定义自定义查询来指示JPA执行[获取连接](https://www.baeldung.com/jpa-join-types#fetch)，以便在初始Student查询中急切地检索相关的School实体**：

```java
public interface StudentRepository extends JpaRepository<Student, String> {
    @Query(value = "SELECT stu FROM Student stu LEFT JOIN FETCH stu.school", countQuery = "SELECT COUNT(stu) FROM Student stu")
    Page<Student> findAll(Pageable pageable);
}
```

执行相同的测试用例后，我们可以从Hibernate日志中观察到，现在只执行了一个连接Student和School表的查询：

```text
Hibernate: select s1_0.student_id,s1_0.first_name,s1_0.last_name,s2_0.school_id,s2_0.name 
from student s1_0 left join school s2_0 on s2_0.school_id=s1_0.school_id
```

### 5.4 实体图

**更简洁的解决方案是使用[@EntityGraph](https://www.baeldung.com/spring-data-jpa-named-entity-graphs)注解，这有助于通过在单个查询中获取实体而不是为每个关联执行额外的查询来优化检索性能**，JPA使用此注解来指定应急切获取哪些关联实体。

让我们看一个临时实体图示例，该示例定义attributePaths来指示JPA在查询Student记录时获取School关联：

```java
public interface StudentRepository extends JpaRepository<Student, String> {
    @EntityGraph(attributePaths = "school")
    Page<Student> findAll(Pageable pageable);
}
```

还有另一种定义实体图的方法，即在Student实体上放置@NamedEntityGraph注解：

```java
@Entity
@Table(name = "student")
@NamedEntityGraph(name = "Student.school", attributeNodes = @NamedAttributeNode("school"))
public class Student {
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "school_id", referencedColumnName = "school_id")
    private School school;

    // Other fields, getters and setters
}
```

随后，我们在StudentRepository的findAll()方法上添加注解@EntityGraph，并引用我们在Student类中定义的命名实体图：

```java
public interface StudentRepository extends JpaRepository<Student, String> {
    @EntityGraph(value = "Student.school")
    Page<Student> findAll(Pageable pageable);
}
```

执行测试用例时，我们将看到JPA执行与自定义查询方法相同的连接查询：

```text
Hibernate: select s1_0.student_id,s1_0.first_name,s1_0.last_name,s2_0.school_id,s2_0.name 
from student s1_0 left join school s2_0 on s2_0.school_id=s1_0.school_id
```

## 6. 总结

在本文中，我们学习了如何在Spring Boot中对查询结果进行分页和排序，包括检索部分数据和完整数据。我们还学习了一些在Spring Boot中有效的数据检索实践，特别是在处理关系时。