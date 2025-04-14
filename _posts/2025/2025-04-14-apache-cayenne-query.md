---
layout: post
title:  Apache Cayenne中的高级查询
category: persistence
copyright: persistence
excerpt: Apache Cayenne
---

## 1. 概述

[之前](../apache-cayenne-orm)，我们重点介绍了如何开始使用Apache Cayenne。

在本文中，我们将介绍如何使用ORM编写简单和高级查询。

## 2. 设置

该设置与上一篇文章中使用的设置类似。

此外，每次测试之前，我们都会保存三个作者，最后再删除他们：

- Paul Xavier
- pAuL Smith
- Vicky Sarra

## 3. 对象选择

让我们从简单的开始，看看如何获取名字中包含“Paul”的所有作者：

```java
@Test
public void whenContainsObjS_thenWeGetOneRecord() {
    List<Author> authors = ObjectSelect.query(Author.class)
        .where(Author.NAME.contains("Paul"))
        .select(context);

    assertEquals(authors.size(), 1);
}
```

接下来，让我们看看如何在作者name列上应用不区分大小写的LIKE类型查询：

```java
@Test
void whenLikeObjS_thenWeGetTwoAuthors() {
    List<Author> authors = ObjectSelect.query(Author.class)
        .where(Author.NAME.likeIgnoreCase("Paul%"))
        .select(context);

    assertEquals(authors.size(), 2);
}
```

接下来，endsWith()表达式将仅返回一条记录，因为只有一位作者具有匹配的名称：

```java
@Test
void whenEndsWithObjS_thenWeGetOrderedAuthors() {
    List<Author> authors = ObjectSelect.query(Author.class)
        .where(Author.NAME.endsWith("Sarra"))
        .select(context);
    Author firstAuthor = authors.get(0);

    assertEquals(authors.size(), 1);
    assertEquals(firstAuthor.getName(), "Vicky Sarra");
}
```

更复杂的方法是查询名字在列表中的作者：

```java
@Test
void whenInObjS_thenWeGetAuthors() {
    List names = Arrays.asList("Paul Xavier", "pAuL Smith", "Vicky Sarra");
 
    List<Author> authors = ObjectSelect.query(Author.class)
        .where(Author.NAME.in(names))
        .select(context);

    assertEquals(authors.size(), 3);
}
```

nin则相反，结果中只会出现“Vicky”：

```java
@Test
void whenNinObjS_thenWeGetAuthors() {
    List names = Arrays.asList("Paul Xavier", "pAuL Smith");
    List<Author> authors = ObjectSelect.query(Author.class)
        .where(Author.NAME.nin(names))
        .select(context);
    Author author = authors.get(0);

    assertEquals(authors.size(), 1);
    assertEquals(author.getName(), "Vicky Sarra");
}
```

请注意，以下两段代码是相同的，因为它们都将创建具有相同参数的相同类型的表达式：

```java
Expression qualifier = ExpressionFactory
    .containsIgnoreCaseExp(Author.NAME.getName(), "Paul");
Author.NAME.containsIgnoreCase("Paul");
```

以下是[Expression](https://cayenne.apache.org/docs/4.0/api/org/apache/cayenne/exp/Expression.html)和[ExpressionFactory](https://cayenne.apache.org/docs/4.0/api/org/apache/cayenne/exp/ExpressionFactory.html)类中一些可用表达式的列表：

- likeExp：用于构建LIKE表达式
- likeIgnoreCaseExp：用于构建LIKE_IGNORE_CASE表达式
- containsExp：用于LIKE查询的表达式，其模式与字符串中的任意位置匹配
- containsIgnoreCaseExp：与containsExp相同，但使用不区分大小写的方法
- startsWithExp：模式应与字符串的开头匹配
- startsWithIgnoreCaseExp：类似于startsWithExp但使用不区分大小写的方法
- endsWithExp：匹配字符串结尾的表达式
- endsWithIgnoreCaseExp：使用不区分大小写的方法匹配字符串结尾的表达式
- expTrue：布尔真表达式
- expFalse：用于布尔值假表达式
- andExp：用于通过and运算符链接两个表达式
- orExp：使用或运算符链接两个表达式

更多书面测试可在文章的代码源中找到，请查看[Github](https://github.com/eugenp/tutorials/tree/master/persistence-modules/apache-cayenne)仓库。

## 4. SelectQuery

它是用户应用程序中使用最广泛的查询类型，SelectQuery描述了一个简单而强大的API，其行为类似于SQL语法，但仍然使用Java对象和方法，并遵循构建器模式来构建更复杂的表达式。

这里我们讨论的是一种表达式语言，我们使用表达式(构建表达式)又名限定符和排序(对结果进行排序)类来构建查询，然后由ORM转换为原生SQL。

为了看到这一点，我们进行了一些测试，在实践中展示了如何构建一些表达式和对数据进行排序。

让我们应用LIKE查询来获取名字类似于“Paul”的作者：

```java
@Test
void whenLikeSltQry_thenWeGetOneAuthor() {
    Expression qualifier = ExpressionFactory.likeExp(Author.NAME.getName(), "Paul%");
    SelectQuery query = new SelectQuery(Author.class, qualifier);
    
    List<Author> authorsTwo = context.performQuery(query);

    assertEquals(authorsTwo.size(), 1);
}
```

这意味着如果你不向查询(SelectQuery)提供任何表达式，则结果将是Author表的所有记录。

可以使用containsIgnoreCaseExp表达式执行类似的查询，以获取名称中包含Paul的所有作者，无论字母的大小写如何：

```java
@Test
void whenCtnsIgnorCaseSltQry_thenWeGetTwoAuthors() {
    Expression qualifier = ExpressionFactory
        .containsIgnoreCaseExp(Author.NAME.getName(), "Paul");
    SelectQuery query = new SelectQuery(Author.class, qualifier);
    
    List<Author> authors = context.performQuery(query);

    assertEquals(authors.size(), 2);
}
```

类似地，让我们以不区分大小写的方式(containsIgnoreCaseExp)获取名字包含“Paul”的作者，并且名字以字母h结尾(endsWithExp)：

```java
@Test
void whenCtnsIgnorCaseEndsWSltQry_thenWeGetTwoAuthors() {
    Expression qualifier = ExpressionFactory
        .containsIgnoreCaseExp(Author.NAME.getName(), "Paul")
        .andExp(ExpressionFactory
            .endsWithExp(Author.NAME.getName(), "h"));
    SelectQuery query = new SelectQuery(Author.class, qualifier);
    List<Author> authors = context.performQuery(query);

    Author author = authors.get(0);

    assertEquals(authors.size(), 1);
    assertEquals(author.getName(), "pAuL Smith");
}
```

可以使用Ordering类执行升序排列：

```java
@Test
void whenAscOrdering_thenWeGetOrderedAuthors() {
    SelectQuery query = new SelectQuery(Author.class);
    query.addOrdering(Author.NAME.asc());
 
    List<Author> authors = query.select(context);
    Author firstAuthor = authors.get(0);

    assertEquals(authors.size(), 3);
    assertEquals(firstAuthor.getName(), "Paul Xavier");
}
```

这里我们不需要使用query.addOrdering(Author.NAME.asc())，也可以直接使用SortOrder类来获取升序：

```java
query.addOrdering(Author.NAME.getName(), SortOrder.ASCENDING);
```

相对而言，其顺序为降序：

```java
@Test
void whenDescOrderingSltQry_thenWeGetOrderedAuthors() {
    SelectQuery query = new SelectQuery(Author.class);
    query.addOrdering(Author.NAME.desc());

    List<Author> authors = query.select(context);
    Author firstAuthor = authors.get(0);

    assertEquals(authors.size(), 3);
    assertEquals(firstAuthor.getName(), "pAuL Smith");
}
```

正如我们在前面的例子中看到的- 设置此顺序的另一种方法是：

```java
query.addOrdering(Author.NAME.getName(), SortOrder.DESCENDING);
```

## 5. SQLTemplate

SQLTemplate也是我们可以与Cayenne一起使用的一种替代方案，可以不使用对象样式查询。

使用SQLTemplate构建查询与编写带有参数的原生SQL语句直接相关。让我们实现一些简单的例子。

以下是我们在每次测试后删除所有作者的方法：

```java
@After
void deleteAllAuthors() {
    SQLTemplate deleteAuthors = new SQLTemplate(Author.class, "delete from author");
    context.performGenericQuery(deleteAuthors);
}
```

要查找所有记录的作者，我们只需要应用SQL查询select \* from Author，我们将直接看到结果是正确的，因为我们恰好有三个保存的作者：

```java
@Test
void givenAuthors_whenFindAllSQLTmplt_thenWeGetThreeAuthors() {
    SQLTemplate select = new SQLTemplate(Author.class, "select * from Author");
    List<Author> authors = context.performQuery(select);

    assertEquals(authors.size(), 3);
}
```

接下来，让我们获取名为“Vicky Sarra”的作者：

```java
@Test
void givenAuthors_whenFindByNameSQLTmplt_thenWeGetOneAuthor() {
    SQLTemplate select = new SQLTemplate(Author.class, "select * from Author where name = 'Vicky Sarra'");
    List<Author> authors = context.performQuery(select);
    Author author = authors.get(0);

    assertEquals(authors.size(), 1);
    assertEquals(author.getName(), "Vicky Sarra");
}
```

## 6. EJBQLQuery

接下来，让我们通过EJBQLQuery查询数据，它是作为在Cayenne中采用Java持久化API的实验的一部分而创建的。

这里，查询采用参数化对象样式；让我们看一些实际的例子。

首先，对所有已保存作者的搜索将如下所示：

```java
@Test
void givenAuthors_whenFindAllEJBQL_thenWeGetThreeAuthors() {
    EJBQLQuery query = new EJBQLQuery("select a FROM Author a");
    List<Author> authors = context.performQuery(query);

    assertEquals(authors.size(), 3);
}
```

让我们再次使用名字“Vicky Sarra”搜索作者，但现在使用EJBQLQuery：

```java
@Test
void givenAuthors_whenFindByNameEJBQL_thenWeGetOneAuthor() {
    EJBQLQuery query = new EJBQLQuery("select a FROM Author a WHERE a.name = 'Vicky Sarra'");
    List<Author> authors = context.performQuery(query);
    Author author = authors.get(0);

    assertEquals(authors.size(), 1);
    assertEquals(author.getName(), "Vicky Sarra");
}
```

一个更好的例子是更新作者：

```java
@Test
void whenUpdadingByNameEJBQL_thenWeGetTheUpdatedAuthor() {
    EJBQLQuery query = new EJBQLQuery("UPDATE Author AS a SET a.name " + "= 'Vicky Edison' WHERE a.name = 'Vicky Sarra'");
    QueryResponse queryResponse = context.performGenericQuery(query);

    EJBQLQuery queryUpdatedAuthor = new EJBQLQuery("select a FROM Author a WHERE a.name = 'Vicky Edison'");
    List<Author> authors = context.performQuery(queryUpdatedAuthor);
    Author author = authors.get(0);

    assertNotNull(author);
}
```

如果我们只想选择一列，应该使用这个查询“select a.name FROM Author a”，[Github](https://github.com/eugenp/tutorials/tree/master/persistence-modules/apache-cayenne)上文章的源代码中提供了更多示例。

## 7. SQLExec

SQLExec也是从Cayenne M4版本开始引入的新的流式查询API。

简单的插入如下所示：

```java
@Test
void whenInsertingSQLExec_thenWeGetNewAuthor() {
    int inserted = SQLExec
        .query("INSERT INTO Author (name) VALUES ('Tuyucheng')")
        .update(context);

    assertEquals(inserted, 1);
}
```

接下来，我们可以根据作者的名字更新作者：

```java
@Test
void whenUpdatingSQLExec_thenItsUpdated() {
    int updated = SQLExec.query(
        "UPDATE Author SET name = 'Tuyucheng' "
        + "WHERE name = 'Vicky Sarra'")
        .update(context);

    assertEquals(updated, 1);
}
```

我们可以从[文档](https://cayenne.apache.org/docs/4.0/index.html)中获得更多详细信息。

## 8. 总结

在本文中，我们研究了使用Cayenne编写简单和更高级查询的多种方法。