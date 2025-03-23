---
layout: post
title:  在MongoRepository中使用Limit和Skip的不同方法
category: spring-data
copyright: spring-data
excerpt: Spring Data MongoDB
---

## 1. 简介

在[Spring Data](https://www.baeldung.com/spring-data-mongodb-tutorial)中使用MongoDB时，MongoRepository接口提供了一种使用内置方法与MongoDB集合交互的简单方法。

在本快速教程中，我们将学习如何在MongoRepository中使用limit和skip。

## 2. 设置

首先，让我们创建一个名为StudentRepository的Repository，我们将在这里存储有关学生的信息：

```java
public interface StudentRepository extends MongoRepository<Student, String> {}
```

然后，我们向此Repository添加一些示例学生记录：

```java
@Before
public void setUp() {
    Student student1 = new Student("A", "Abraham", 15L);
    Student student2 = new Student("B", "Usman", 30L);
    Student student3 = new Student("C", "David", 20L);
    Student student4 = new Student("D", "Tina", 45L);
    Student student5 = new Student("E", "Maria", 33L);

    studentList = Arrays.asList(student1, student2, student3, student4, student5);
    studentRepository.saveAll(studentList);
}
```

之后，我们将深入了解使用skip和limit查找特定学生信息的细节。

## 3. 使用Aggregation管道

**聚合管道是一种用于处理、转换和分析数据的强大工具**，它通过将多个阶段链接在一起来工作，每个阶段执行特定操作。这些操作包括过滤、分组、排序、分页等。

让我们应用limit并跳到一个基本的例子：

```java
@Test
public void whenRetrievingAllStudents_thenReturnsCorrectNumberOfRecords() {
    // WHEN
    List<Student> result = studentRepository.findAll(0L, 5L);
    // THEN
    assertEquals(5, result.size());
}
```

上述聚合管道跳过指定数量的文档并将输出限制为指定数量。

```java
@Test
public void whenLimitingAndSkipping_thenReturnsLimitedStudents() {
    // WHEN
    List<Student> result = studentRepository.findAll(3L, 2L);
    // THEN
    assertEquals(2, result.size());
    assertEquals("Tina", result.get(0).getName());
    assertEquals("Maria", result.get(1).getName());
}
```

我们甚至可以在复杂的聚合管道中应用limit和skip：

```java
@Aggregation(pipeline = {
        "{ '$match': { 'id' : ?0 } }",
        "{ '$sort' : { 'id' : 1 } }",
        "{ '$skip' : ?1 }",
        "{ '$limit' : ?2 }"
})
List<Student> findByStudentId(final String studentId, Long skip, Long limit);
```

测试如下：

```java
@Test
public void whenFilteringById_thenReturnsStudentsMatchingCriteria() {
    // WHEN
    List<Student> result = studentRepository.findByStudentId("A", 0L, 5L);
    // THEN
    assertEquals(1, result.size());
    assertEquals("Abraham", result.get(0).getName());
}
```

## 4. 使用Pageable

在Spring Data中，[Pageable](https://www.baeldung.com/spring-data-jpa-pagination-sorting)是一个接口，表示以分页方式检索数据的请求。**从MongoDB集合查询数据时，Pageable对象允许我们指定参数，例如页码、页面大小和排序条件**。这对于在应用程序中显示大型数据集特别有用，因为一次显示所有元素会很慢。

让我们探索如何使用Pageable定义Repository方法实现高效的数据检索：

```java
Page<Student> findAll(Pageable pageable);
```

让我们添加一个测试：

```java
@Test
public void whenFindByStudentIdUsingPageable_thenReturnsPageOfStudents() {
    // GIVEN
    Sort sort = Sort.by(Sort.Direction.DESC, "id");
    Pageable pageable = PageRequest.of(0, 5, sort);

    // WHEN
    Page<Student> resultPage = studentRepository.findAll(pageable);

    // THEN
    assertEquals(5, resultPage.getTotalElements());
    assertEquals("Maria", resultPage.getContent().get(0).getName());
}
```

## 5. 总结

在这篇短文中，我们探讨了MongoRepository中skip和limit功能的使用。此外，我们还强调了Pageable用于简化分页的实用性以及@Aggregation用于更好地控制查询逻辑或特定过滤需求的灵活性。