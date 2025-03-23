---
layout: post
title:  使用Spring Data JPA查找不同的行
category: spring-data
copyright: spring-data
excerpt: Spring Data JPA
---

## 1. 概述

在某些情况下，我们需要从数据库中获取唯一元素。本教程重点介绍如何使用Spring Data JPA查询不同数据，并研究检索不同实体和字段的各种方法。

## 2. 场景设置

为了进行说明，我们创建两个实体类，School和Student：

```java
@Entity
@Table(name = "school")
public class School {
    @Id
    @Column(name = "school_id")
    @GeneratedValue(strategy= GenerationType.IDENTITY)
    private int id;

    @Column(name = "name", length = 100)
    private String name;

    @OneToMany(mappedBy = "school")
    private List<Student> students;

    // constructors, getters and setters
}
```

```java
@Entity
@Table(name = "student")
public class Student {
    @Id
    @Column(name = "student_id")
    private int id;

    @Column(name = "name", length = 100)
    private String name;

    @Column(name = "birth_year")
    private int birthYear;

    @ManyToOne
    @JoinColumn(name = "school_id", referencedColumnName = "school_id")
    private School school;

    // constructors, getters and setters
}
```

我们定义了一对多关系，其中每所学校与多名学生相关联。

## 3. 不同实体

我们的样本实体现已设置完毕，让我们继续创建一个[Repository](https://www.baeldung.com/spring-data-repositories)，用于按学生出生年份检索不同的学校。使用Spring Data JPA有不同的方法可以获取不同的行，第一种方法是使用[派生查询](https://www.baeldung.com/spring-data-derived-queries)：

```java
@Repository
public interface SchoolRepository extends JpaRepository<School, Integer> {
    List<School> findDistinctByStudentsBirthYear(int birthYear);
}
```

派生查询是不言自明的，易于理解。它根据学生的出生年份查找所有不同的School实体。如果我们调用该方法，我们将在控制台日志中看到JPA在School实体上执行的SQL。它显示除关系之外的所有字段都已检索：

```text
Hibernate: 
    select
        distinct s1_0.school_id,
        s1_0.name 
    from
        school s1_0 
    left join
        student s2_0 
            on s1_0.school_id=s2_0.school_id 
    where
        s2_0.birth_year=?
```

**如果我们想要不同的计数而不是整个实体，我们可以通过在方法名称中用count替换find来创建另一个派生查询**：

```java
Long countDistinctByStudentsBirthYear(int birthYear);
```

## 4. 自定义查询的不同字段

在某些情况下，我们不需要从实体中检索每个字段。假设我们想在Web界面中显示实体的搜索结果，**搜索结果可能只需要显示实体中的几个字段。对于这种情况，我们可以通过限制所需的字段来减少检索时间，尤其是在结果集很大的情况下**。

在我们的示例中，我们只对不同的学校名称感兴趣。因此，我们将创建一个[自定义查询](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa#2-manual-custom-queries)来仅检索学校名称。我们用@Query标注该方法并将JPQL查询放在其中，我们通过@Param注解将出生年份参数传递到JPQL中：

```java
@Query("SELECT DISTINCT sch.name FROM School sch JOIN sch.students stu WHERE stu.birthYear = :birthYear")
List<String> findDistinctSchoolNameByStudentsBirthYear(@Param("birthYear") int birthYear);
```

执行后，我们会在控制台日志中看到JPA生成的SQL如下，其中只涉及到学校名称字段，而不是所有字段：

```text
Hibernate: 
    select
        distinct s1_0.name 
    from
        school s1_0 
    join
        student s2_0 
            on s1_0.school_id=s2_0.school_id 
    where
        s2_0.birth_year=?
```

## 5. 通过投影区分字段

Spring Data JPA查询方法通常使用实体作为返回类型，但是，我们可以应用Spring Data提供的[投影](https://www.baeldung.com/spring-data-jpa-projections)作为自定义查询方法的替代。这使我们能够从实体中检索部分字段，而不是全部。

因为我们只想将检索限制在学校名称字段，所以我们将创建一个包含School实体中name字段的Getter方法的接口：

```java
public interface NameView {
    String getName();
}
```

**投影接口中的方法名必须与我们目标实体中的Getter方法名相同**，毕竟，我们必须将以下方法添加到Repository中：

```java
List<NameView> findDistinctNameByStudentsBirthYear(int birthYear);
```

执行后，我们将看到生成的SQL与自定义查询中生成的SQL类似：

```text
Hibernate: 
    select
        distinct s1_0.name 
    from
        school s1_0 
    left join
        student s2_0 
            on s1_0.school_id=s2_0.school_id 
    where
        s2_0.birth_year=?
```

## 6. 总结

在本文中，我们探讨了通过Spring Data JPA从数据库查询不同行的不同方法，包括不同实体和不同字段。我们可以根据自己的需要使用不同的方法。