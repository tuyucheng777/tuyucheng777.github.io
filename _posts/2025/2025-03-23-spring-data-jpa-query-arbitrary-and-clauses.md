---
layout: post
title:  具有任意AND子句的动态Spring Data JPA Repository查询
category: spring-data
copyright: spring-data
excerpt: Spring Data JPA
---

## 1. 概述

在使用Spring Data开发应用程序时，我们经常需要根据选择条件构建动态查询来从数据库中获取数据。

本教程探讨了在Spring Data JPA Repository中创建动态查询的三种方法：按Example查询、按Specification查询和按Querydsl查询。

## 2. 场景设置

在我们的演示中，我们将创建两个实体School和Student。这些类之间的关联是一对多的，一个School可以有多个Student：

```java
@Entity
@Table
public class School {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column
    private Long id;

    @Column
    private String name;

    @Column
    private String borough;

    @OneToMany(mappedBy = "school")
    private List<Student> studentList;

    // constructor, getters and setters
}
```

```java
@Entity
@Table
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column
    private Long id;

    @Column
    private String name;

    @Column
    private Integer age;

    @ManyToOne
    private School school;

    // constructor, getters and setters
}
```

除了实体类之外，我们还为Student实体定义一个Spring Data Repository：

```java
public interface StudentRepository extends JpaRepository<Student, Long> {
}
```

最后，我们将向School表添加一些示例数据：

|  id  |           name            |       borough        |
| :--: | :-----------------------: | :------------------: |
|  1   | University of West London |        Ealing        |
|  2   |    Kingston University    | Kingston upon Thames |

我们对学生表也做同样的事情：

|  id  |     name      | age  | school_id |
| :--: | :-----------: | :--: | :-------: |
|  1   |  Emily Smith  |  20  |     2     |
|  2   |  James Smith  |  20  |     1     |
|  3   | Maria Johnson |  22  |     1     |
|  4   | Michael Brown |  21  |     1     |
|  5   | Sophia Smith  |  22  |     1     |

在后续章节中，我们将根据以下标准采用不同的方法查找记录：

- 学生的名字以Smith结尾，并且
- 学生年龄为20岁
- 学生的学校位于Ealing

## 3. 通过Example进行查询

Spring Data提供了一种[使用Example查询实体](https://www.baeldung.com/spring-data-query-by-example)的简单方法。**这个想法很简单：我们创建一个示例实体，并将我们要查找的搜索条件放入其中。然后，我们使用此示例查找与其匹配的实体**。

要采用此功能，Repository必须实现接口QueryByExampleExecutor。在我们的例子中，该接口已在JpaRepository中扩展，我们通常在Repository中扩展该接口。因此，无需明确实现它。

现在，让我们创建一个Student示例以包含我们想要过滤的三个选择标准：

```java
School schoolExample = new School();
schoolExample.setBorough("Ealing");

Student studentExample = new Student();
studentExample.setAge(20);
studentExample.setName("Smith");
studentExample.setSchool(schoolExample);

Example example = Example.of(studentExample);
```

设置示例后，我们调用Repository findAll(...)方法来获取结果：

```java
List<Student> studentList = studentRepository.findAll(example);
```

但是，上面的示例仅支持精确匹配。如果我们想获取名字以“Smith”结尾的学生，我们需要自定义匹配策略。示例查询提供了ExampleMatcher类来执行此操作，我们需要做的就是在name字段上创建一个ExampleMatcher并将其应用于Example实例：

```java
ExampleMatcher customExampleMatcher = ExampleMatcher.matching()
    .withMatcher("name", ExampleMatcher.GenericPropertyMatchers.endsWith().ignoreCase());
Example<Student> example = Example.of(studentExample, customExampleMatcher);
```

我们在这里使用的ExampleMatcher是不言自明的，它使name字段不区分大小写地匹配，并确保值以我们示例中指定的名称结尾。

通过Example查询易于理解和实现，但是，它不支持更复杂的查询，例如字段的大于或小于条件。

## 4. 按Specification查询

**Spring Data JPA中的[按Specification查询](https://www.baeldung.com/spring-data-criteria-queries#specifications)允许使用Specification接口根据一组条件创建动态查询**。

与传统方法(例如派生查询方法或使用@Query的自定义查询)相比，这种方法更加灵活。对于复杂的查询要求或需要在运行时动态调整查询的情况，这种方法非常有用。

与示例查询类似，我们的Repository方法必须扩展一个接口才能启用此功能。这次，我们需要扩展JpaSpecificationExecutor：

```java
public interface StudentRepository extends JpaRepository<Student, Long>, JpaSpecificationExecutor<Student> {
}
```

接下来，我们必须为每个过滤条件定义三个方法。Specification并没有限制我们为一个过滤条件使用一种方法，这主要是为了清晰起见，使其更具可读性：

```java
public class StudentSpecification {
    public static Specification<Student> nameEndsWithIgnoreCase(String name) {
        return (root, query, criteriaBuilder) ->
                criteriaBuilder.like(criteriaBuilder.lower(root.get("name")), "%" + name.toLowerCase());
    }

    public static Specification<Student> isAge(int age) {
        return (root, query, criteriaBuilder) -> criteriaBuilder.equal(root.get("age"), age);
    }

    public static Specification<Student> isSchoolBorough(String borough) {
        return (root, query, criteriaBuilder) -> {
            Join<Student, School> scchoolJoin = root.join("school");
            return criteriaBuilder.equal(scchoolJoin.get("borough"), borough);
        };
    }
}
```

从上面的方法中，我们知道我们使用[CriteriaBuilder](https://www.baeldung.com/hibernate-criteria-queries)来构造过滤条件。CriteriaBuilder帮助我们以编程方式在JPA中构建动态查询，并为我们提供了类似于编写SQL查询的灵活性，它允许我们使用equal(...)和like(...)等方法创建谓词来定义条件。

对于更复杂的操作，例如将额外表与基表连接起来时，我们将使用Root.join(...)。Root充当FROM子句的锚点，提供对实体的属性和关系的访问。

现在我们调用Repository方法来获取按照Specification过滤的结果：

```java
Specification<Student> studentSpec = Specification
    .where(StudentSpecification.nameEndsWithIgnoreCase("smith"))
    .and(StudentSpecification.isAge(20))
    .and(StudentSpecification.isSchoolBorough("Ealing"));
List<Student> studentList = studentRepository.findAll(studentSpec);
```

## 5. 通过QueryDsl进行查询

**与Example相比，Specification功能强大，能够处理更复杂的查询。然而，当我们处理包含许多选择条件的复杂查询时，Specification接口会变得冗长且难以阅读**。

[QueryDSL](https://www.baeldung.com/intro-to-querydsl)试图用更直观的解决方案来解决Specification的局限性，它是一个类型安全的框架，用于以直观、可读且强类型的方式创建动态查询。

为了使用QueryDSL，我们需要添加一些依赖。让我们将以下[Querydsl JPA](https://mvnrepository.com/artifact/com.querydsl/querydsl-jpa)和[APT支持](https://mvnrepository.com/artifact/com.querydsl/querydsl-apt)依赖添加到我们的pom.xml中：

```xml
<dependency>
    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-jpa</artifactId>
    <version>5.1.0</version>
    <classifier>jakarta</classifier>
</dependency>
<dependency>
    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-apt</artifactId>
    <version>5.1.0</version>
    <classifier>jakarta</classifier>
    <scope>provided</scope>
</dependency>
```

值得注意的是，JPA的包名称在JPA 3.0中从javax.persistence更改为jakarta.persistence。如果我们使用3.0及更高版本，则必须将jakarta分类器放入依赖中。

除了依赖之外，我们还必须在pom.xml的插件部分包含以下注解处理器：

```xml
<plugin>
    <groupId>com.mysema.maven</groupId>
    <artifactId>apt-maven-plugin</artifactId>
    <version>1.1.3</version>
    <executions>
        <execution>
            <phase>generate-sources</phase>
            <goals>
                <goal>process</goal>
            </goals>
            <configuration>
                <outputDirectory>target/generated-sources</outputDirectory>
                <processor>com.mysema.query.apt.jpa.JPAAnnotationProcessor</processor>
            </configuration>
        </execution>
    </executions>
</plugin>
```

**此处理器在编译时为我们的实体类生成元模型类**，将这些设置合并到我们的应用程序中并进行编译后，我们将看到Querydsl在构建文件夹中生成了两个查询类型QStudent和QSchool。

以类似的方式，我们这次必须包含QuerydslPredicateExecutor，以使我们的Repository能够通过以下查询类型获取结果：

```java
public interface StudentRepository extends JpaRepository<Student, Long>, QuerydslPredicateExecutor<Student>{
}
```

接下来，我们将基于这些查询类型创建一个动态查询，并在我们的StudentRepository中使用它进行查询。这些查询类型已经包含了相应实体类的所有属性，因此，我们可以直接引用构建谓词时所需的字段：

```java
QStudent qStudent = QStudent.student;
BooleanExpression predicate = qStudent.name.endsWithIgnoreCase("smith")
    .and(qStudent.age.eq(20))
    .and(qStudent.school.borough.eq("Ealing"));
List studentList = (List) studentRepository.findAll(predicate);
```

如上面的代码所示，使用查询类型定义查询条件非常简单且直观。

尽管设置所需的依赖关系是一项一次性任务，非常复杂，但它提供了与Specification相同的流式方式，直观且易于阅读。

**而且，我们不需要像Specification类中要求的那样手动明确定义过滤条件**。

## 6. 总结

在本文中，我们探讨了在Spring Data JPA中创建动态查询的不同方法。

- 按Example查询最适合用于简单、精确匹配的查询
- 如果我们需要更多类似SQL的表达式和比较，则按Specification查询对于中等复杂的查询非常有用
- QueryDSL最适合高度复杂的查询，因为它可以使用查询类型简单地定义条件