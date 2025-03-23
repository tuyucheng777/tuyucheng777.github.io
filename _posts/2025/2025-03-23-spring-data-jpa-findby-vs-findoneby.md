---
layout: post
title:  Spring Data JPA中findBy和findOneBy之间的区别
category: spring-data
copyright: spring-data
excerpt: Spring Data JPA
---

## 1. 概述

Spring Data Repository带有许多简化数据访问逻辑实现的方法，然而，选择正确的方法并不总是像我们想象的那么容易。

一个例子是带有前缀findBy和findOneBy的方法。尽管从名称上看它们似乎在做同样的事情，但它们还是有点不同的。

## 2. Spring Data中的派生查询方法

[Spring Data JPA](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)经常因其派生查询方法功能而受到称赞，这些方法提供了一种从方法名称派生特定查询的方法。例如，如果我们想通过foo属性检索数据，我们只需编写findByFoo()即可。

通常，我们可以使用多个前缀来构造派生查询方法。这些前缀包括findBy和findOneBy。那么，让我们在实践中看看它们。

## 3. 实例

首先，让我们考虑一下Person[实体类](https://www.baeldung.com/jpa-entities)：

```java
@Entity
public class Person {

    @Id
    private int id;
    private String firstName;
    private String lastName;

    // standard getters and setters
}
```

在这里，我们将使用[H2](https://www.baeldung.com/spring-boot-h2-database)作为数据库，让我们使用一个基本的SQL脚本来为数据库添加数据：

```sql
INSERT INTO person (id, first_name, last_name) VALUES(1, 'Azhrioun', 'Abderrahim');
INSERT INTO person (id, first_name, last_name) VALUES(2, 'Brian', 'Wheeler');
INSERT INTO person (id, first_name, last_name) VALUES(3, 'Stella', 'Anderson');
INSERT INTO person (id, first_name, last_name) VALUES(4, 'Stella', 'Wheeler');
```

最后，让我们创建一个JPA Repository来管理我们的Person实体：

```java
@Repository
public interface PersonRepository extends JpaRepository<Person, Integer> {
}
```

### 3.1 findBy前缀

findBy是用于创建表示搜索查询的派生查询方法的最常用前缀之一。

**动词“find”告诉Spring Data生成select查询。另一方面，关键字“By”充当where子句，因为它会过滤返回的结果**。

接下来，让我们向PersonRepository添加一个派生查询方法，通过名字获取一个人：

```java
Person findByFirstName(String firstName);
```

我们可以看到，我们的方法返回一个Person对象。现在，让我们为findByFirstName()添加一个测试用例：

```java
@Test
void givenFirstName_whenCallingFindByFirstName_ThenReturnOnePerson() {
    Person person = personRepository.findByFirstName("Azhrioun");

    assertNotNull(person);
    assertEquals("Abderrahim", person.getLastName());
}
```

现在我们已经了解了如何使用findBy创建返回单个对象的查询方法，让我们看看是否可以使用它来获取对象列表。为此，我们将向PersonRepository添加另一个查询方法：

```java
List<Person> findByLastName(String lastName);
```

顾名思义，这种新方法将帮助我们找到所有具有相同lastName的对象。

类似地，让我们使用另一个测试用例来测试findByLastName()：

```java
@Test
void givenLastName_whenCallingFindByLastName_ThenReturnList() {
    List<Person> person = personRepository.findByLastName("Wheeler");

    assertEquals(2, person.size());
}
```

不出所料，测试成功通过。

简而言之，**我们可以使用findBy来获取一个对象或一个对象集合**。

**这里的区别在于查询方法的返回类型，Spring Data根据返回类型来决定返回一个或多个对象**。

### 3.2 findOneBy前缀

通常，findOneBy只是findBy的一个特定变体。**它明确表示要查找一条记录**。那么，让我们看看它的实际效果：

```java
Person findOneByFirstName(String firstName);
```

接下来，我们将添加另一个测试来确认我们的方法运行正常：

```java
@Test
void givenFirstName_whenCallingFindOneByFirstName_ThenReturnOnePerson() {
    Person person = personRepository.findOneByFirstName("Azhrioun");

    assertNotNull(person);
    assertEquals("Abderrahim", person.getLastName());
}
```

现在，如果我们使用findOneBy获取对象列表会发生什么？让我们来看看！

首先，我们添加另一种查询方法来查找所有具有相同lastName的Person对象：

```java
List<Person> findOneByLastName(String lastName);
```

接下来，让我们使用测试用例来测试我们的方法：

```java
@Test
void givenLastName_whenCallingFindOneByLastName_ThenReturnList() {
    List<Person> persons = personRepository.findOneByLastName("Wheeler");

    assertEquals(2, persons.size());
}
```

如上所示，findOneByLastName()返回一个列表，没有抛出任何异常。

从技术角度来看，findOneBy和findBy之间没有区别。但是，**创建一个返回带有前缀findOneBy的集合的查询方法在语义上是没有意义的**。

简而言之，前缀findOneBy仅为需要返回一个对象提供了语义描述。

Spring Data依赖这个[正则表达式](https://github.com/spring-projects/spring-data-commons/blob/14d5747f68737bb44441dc511cf16393d9d85dc8/src/main/java/org/springframework/data/repository/query/parser/PartTree.java#L65)来忽略动词“find”和关键字“By”之间的所有字符。所以findBy、findOneBy、findXyzBy...都是类似的。

使用find关键字创建派生查询方法时，需要记住几个关键点：

- 派生查询方法的重要部分是关键字find和By
- 我们可以在find和By之间添加单词来在语义上表示某些东西
- Spring Data根据方法的返回类型决定返回一个对象或一个集合

## 4. IncorrectResultSizeDataAccessException

**这里需要提到的一个重要警告是，当返回的结果不是预期的大小时，findByLastName()和findOneByLastName()方法都会抛出IncorrectResultSizeDataAccessException**。

例如，如果有多个Person对象具有给定的名字，则Person findByFirstName(String firstName)将引发异常。

因此，让我们使用测试用例来确认这一点：

```java
@Test
void givenFirstName_whenCallingFindByFirstName_ThenThrowException() {
    IncorrectResultSizeDataAccessException exception = assertThrows(IncorrectResultSizeDataAccessException.class, () -> personRepository.findByFirstName("Stella"));

    assertEquals("query did not return a unique result: 2", exception.getMessage());
}
```

异常的原因是，尽管我们声明方法返回一个对象，但执行的查询返回了多条记录。

类似地，让我们使用测试用例确认findOneByFirstName()是否抛出IncorrectResultSizeDataAccessException：

```java
@Test
void givenFirstName_whenCallingFindOneByFirstName_ThenThrowException() {
    IncorrectResultSizeDataAccessException exception = assertThrows(IncorrectResultSizeDataAccessException.class, () -> personRepository.findOneByFirstName("Stella"));

    assertEquals("query did not return a unique result: 2", exception.getMessage());
}
```

## 5. 总结

在本文中，我们详细探讨了Spring Data JPA中findBy和findOneBy前缀之间的异同。

在此过程中，我们解释了Spring Data JPA中派生的查询方法。然后，我们强调，尽管findBy和findOneBy之间的语义意图不同，但它们在本质上是相同的。

最后，我们展示了如果我们选择错误的返回类型，两者都会抛出IncorrectResultSizeDataAccessException。