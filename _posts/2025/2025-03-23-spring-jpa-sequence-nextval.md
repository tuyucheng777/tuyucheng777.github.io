---
layout: post
title:  使用Spring JPA从序列中获取Nextval
category: spring-data
copyright: spring-data
excerpt: Spring Data JPA
---

## 1. 简介

[序列](https://www.baeldung.com/hibernate-identifiers#3-sequence-generation)是唯一ID的数字生成器，用于避免数据库中出现重复条目。Spring JPA提供了在大多数情况下自动处理序列的方法。但是，在特定情况下，我们需要在持久化实体之前手动检索下一个序列值。例如，我们可能希望在将发票详细信息保存到数据库之前生成唯一的发票号。

在本教程中，我们将探讨使用Spring Data JPA从数据库序列中获取下一个值的几种方法。

## 2. 设置项目依赖

在深入研究使用Spring Data JPA序列之前，让我们确保我们的项目已正确设置。我们需要将[Spring Data JPA](https://mvnrepository.com/artifact/org.springframework.data/spring-data-jpa)和[PostgreSQL驱动程序](https://mvnrepository.com/artifact/org.postgresql/postgresql)依赖添加到Maven pom.xml文件中，并在数据库中创建序列。

### 2.1 Maven依赖

首先，让我们向我们的Maven项目添加必要的依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>
```

### 2.2 测试数据

下面是在运行测试用例之前我们用来准备数据库的SQL脚本，我们可以将此脚本保存在.sql文件中，并将其放在项目的src/test/resources目录中：

```sql
DROP SEQUENCE IF EXISTS my_sequence_name;
CREATE SEQUENCE my_sequence_name START 1;
```

此命令创建一个从1开始的序列，每调用一次NEXTVAL就会增加1。

然后，我们在测试类中使用@Sql注解并将executionPhase属性设置为BEFORE_TEST_METHOD，在每次测试方法执行之前将测试数据插入数据库：

```java
@Sql(scripts = "/testsequence.sql", executionPhase = Sql.ExecutionPhase.BEFORE_TEST_METHOD)
```

## 3. 使用@SequenceGenerator

**Spring JPA可以在后台与序列配合使用，每当我们添加新元素时都会自动分配一个唯一的编号**。我们通常使用JPA中的@SequenceGenerator注解来配置序列生成器，此生成器可用于自动生成实体类中的主键。

**此外，它通常与@GeneratedValue注解结合使用，以指定生成主键值的策略**。以下是我们可以使用@SequenceGenerator配置主键生成的方法：

```java
@Entity
public class MyEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "mySeqGen")
    @SequenceGenerator(name = "mySeqGen", sequenceName = "my_sequence_name", allocationSize = 1)
    private Long id;

    // Other entity fields and methods
}
```

使用[GenerationType.SEQUENCE](https://www.baeldung.com/jpa-get-auto-generated-id#using-id-generation-strategies)策略的@GeneratedValue注解表示将使用序列生成主键值，**随后，generator属性将此策略与名为“mySeqGen”的指定序列生成器关联起来**。

此外，@SequenceGenerator注解配置了名为“mySeqGen”的序列生成器，它指定了数据库序列的名称“my_sequence_name”和可选参数分配大小。

**assignmentSize是一个整数值，指定一次从数据库预取多少个序列号**。例如，如果我们将assignmentSize设置为50，则持久化提供程序会在一次调用中请求50个序列号并将它们存储在内部。**然后，它会使用这些预取的数字来生成未来的实体ID，这对于写入量大的应用程序非常有用**。

**通过此配置，当我们持久化一个新的MyEntity实例时，持久化提供程序会自动从名为“my_sequence_name”的序列中获取下一个值**，然后将检索到的序列号分配给实体的id字段，然后再将其保存到数据库。

以下示例说明如何在持久化实体ID之后检索序列号：

```java
MyEntity entity = new MyEntity();

myEntityRepository.save(entity);
long generatedId = entity.getId();

assertNotNull(generatedId);
assertEquals(1L, generatedId);
```

保存实体后，我们可以使用实体对象上的getId()方法访问生成的ID。

## 4. Spring Data JPA自定义查询

在某些情况下，我们可能需要下一个数字或唯一ID，然后才能将其保存到数据库。为此，Spring Data JPA提供了一种使用自定义查询来[查询](https://www.baeldung.com/spring-data-jpa-query)下一个序列的方法，**此方法涉及在Repository中使用原生SQL查询来访问序列**。

检索下一个值的具体语法取决于数据库系统，**例如，在PostgreSQL或Oracle中，我们使用NEXTVAL函数从序列中获取下一个值**。以下是使用@Query注解的示例实现：

```java
@Repository
public interface MyEntityRepository extends JpaRepository<MyEntity, Long> {
    @Query(value = "SELECT NEXTVAL('my_sequence_name')", nativeQuery = true)
    Long getNextSequenceValue();
}
```

在示例中，我们使用@Query标注getNextSequenceValue()方法。使用@Query注解，我们可以指定一个原生SQL查询，该查询使用NEXTVAL函数从序列中检索下一个值，这样就可以直接访问序列值：

```java
@Autowired
private MyEntityRepository myEntityRepository;

long generatedId = myEntityRepository.getNextSequenceValue();

assertNotNull(generatedId);
assertEquals(1L, generatedId);
```

由于这种方法涉及编写特定于数据库的代码，如果我们更改数据库，则可能需要调整SQL查询。

## 5. 使用EntityManager

另外，Spring JPA还提供了[EntityManager](https://www.baeldung.com/hibernate-entitymanager) API，我们可以使用它直接检索下一个序列值。**此方法提供更细粒度的控制，但绕过了JPA的对象关系映射功能**。

下面是如何使用Spring Data JPA中的EntityManager API从序列中检索下一个值的示例：

```java
@PersistenceContext
private EntityManager entityManager;

public Long getNextSequenceValue(String sequenceName) {
    BigInteger nextValue = (BigInteger) entityManager.createNativeQuery("SELECT NEXTVAL(:sequenceName)")
        .setParameter("sequenceName", sequenceName)
        .getSingleResult();
    return nextValue.longValue();
}
```

我们使用createNativeQuery()方法创建原生SQL查询，在此查询中，调用NEXTVAL函数从序列中检索下一个值。**我们可以注意到PostgreSQL中的NEXTVAL函数返回[BigInteger](https://www.baeldung.com/java-biginteger)类型的值。因此，我们使用longValue()方法将BigInteger转换为Long**。

使用getNextSequenceValue()方法，我们可以这样调用它：

```java
@Autowired
private MyEntityService myEntityService;

long generatedId = myEntityService.getNextSequenceValue("my_sequence_name");
assertNotNull(generatedId);
assertEquals(1L, generatedId);
```

## 6. 总结

在本文中，我们探讨了使用Spring Data JPA从数据库序列中获取下一个值的各种方法，Spring JPA通过@SequenceGenerator和@GeneratedValue等注解提供与数据库序列的无缝集成。在保存实体之前需要下一个序列值的场景中，我们可以使用Spring Data JPA的自定义查询。