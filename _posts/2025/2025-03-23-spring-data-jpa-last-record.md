---
layout: post
title:  如何在Spring Data JPA中获取最后一条记录
category: springdata
copyright: springdata
excerpt: Spring Data JPA
---

## 1. 概述

在本简短教程中，我们将探索在Spring Data JPA中获取最后一条记录的不同方法。首先，我们将了解如何使用派生查询方法来实现这一点。然后，我们将探索如何使用@Query注解来实现相同的目的。

## 2. 设置

首先，让我们创建并初始化我们要查询的表，让我们从创建一个Post[实体类](https://www.baeldung.com/jpa-entities)开始：

```java
@Entity
public class Post {

    @Id
    private Long id;
    private String title;
    private LocalDate publicationDate;

    // standard getters and setters
}
```

这里，@Entity表示被标注的类代表数据库中的一个表。同样，@Id注解定义了主键。

为了简单起见，我们将使用[H2](https://www.baeldung.com/spring-boot-h2-database)作为内存数据库。首先，让我们添加一个基本的SQL脚本来创建映射到Post类的post表：

```sql
DROP TABLE IF EXISTS post;
CREATE TABLE post(
    id INT PRIMARY KEY,
    title VARCHAR(200),
    publication_date DATE
)
```

接下来，让我们用数据填充表：

```sql
INSERT INTO post (id, title, publication_date) VALUES(1, 'Facebook post', '2020-11-10');
INSERT INTO post (id, title, publication_date) VALUES(2, 'Instagram post', '2020-12-24');
INSERT INTO post (id, title, publication_date) VALUES(3, 'Twitter post', '2023-01-10');
INSERT INTO post (id, title, publication_date) VALUES(4, 'tiktok post', '2023-03-18');
INSERT INTO post (id, title, publication_date) VALUES(5, 'Pinterest post', '2023-09-09');
```

我们可以看到，这里的最后一条记录是id为5的记录。因此，为了实现获取最后一条记录的目标，我们将根据publication_date反转记录的顺序。然后，我们将使用Spring Data JPA方法从排序结果中获取第一条记录。这样，我们就可以得到表的最后一条记录。

## 3. 使用派生查询方法

Spring Data JPA因其[派生查询方法](https://www.baeldung.com/spring-data-derived-queries)而备受赞誉，此功能提供了一种方便的方法，可以根据方法名称生成查询，而无需手动编写SQL语句。

Spring Data JPA不提供任何直接方法来获取最后一条记录。另一方面，它提供了从一组记录的开头检索数据的直接方法。

例如，我们可以使用[findFirst](https://www.baeldung.com/spring-data-jpa-findfirst-vs-findtop#using-spring-data-jpa-findfirst)前缀来创建获取第一条记录的派生查询。那么，让我们看看它的实际效果：

```java
public interface PostRepository extends JpaRepository<Post, Integer> {

    Post findFirstByOrderByPublicationDateDesc();
}
```

方法名称findFirstByOrderByPublicationDateDesc()的每个部分都有其意义，动词“find”告诉Spring Data JPA生成select查询，“First”表示它应该从结果集中检索第一条记录。

此外，“OrderByPublicationDateDesc”表示我们想要按照publicationDate属性的反向顺序对记录进行排序。

这里，Spring Data JPA智能地评估方法名称。它首先按发布日期降序对帖子进行排序，这样，它将最后一条记录放在结果的开头。

然后，它解释“findFirst”以返回排序记录的第一个元素。结果，我们得到了表的最后一条记录。

现在，让我们添加一个测试用例来确认一切是否按预期工作：

```java
@Test
void givenPosts_whenUsingFindFirstDerivedQuery_thenReturnLastPost() {
    Post post = postRepository.findFirstByOrderByPublicationDateDesc();

    assertNotNull(post);
    assertEquals(5, post.getId());
}
```

我们可以看到我们的测试成功通过。

类似地，我们可以使用[findTop](https://www.baeldung.com/spring-data-jpa-findfirst-vs-findtop#using-spring-data-jpa-findtop)关键字来实现相同的结果，我们可以交替使用firstFirst或findTop而不会出现任何问题：

```java
Post findTopByOrderByPublicationDateDesc();
```

最后，让我们为findTopByOrderByPublicationDateDesc()方法创建另一个测试用例：

```java
@Test
void givenPosts_whenUsingFindTopDerivedQuery_thenReturnLastPost() {
    Post post = postRepository.findTopByOrderByPublicationDateDesc();

    assertNotNull(post);
    assertEquals(5, post.getId());
}
```

如上图，测试用例成功通过。

## 4. 使用@Query注解

另一个解决方案是使用[@Query](https://www.baeldung.com/spring-data-jpa-query)注解将方法绑定到检索最后一条记录的查询。默认情况下，@Query接收JPQL查询。因此，让我们将另一个名为findLastPost()的方法添加到PostRepository，并使用@Query指定获取最后一篇帖子的查询：

```java
@Query("SELECT p FROM Post p ORDER BY p.publicationDate DESC LIMIT 1")
Post findLastPost();
```

简而言之，我们选择了按发布日期倒序排序的帖子。然后，我们使用LIMIT 1仅检索一篇帖子，返回的帖子表示最后一条记录。

与往常一样，让我们添加一个测试用例来测试我们的新方法：

```java
@Test
void givenPosts_whenUsingQueryAnnotation_thenReturnLastPost() {
    Post post = postRepository.findLastPost();

    assertNotNull(post);
    assertEquals(5, post.getId());
}
```

不出所料，最后一条记录是id为5的帖子。

## 5. 总结

在本教程中，我们探索了使用Spring Data JPA检索特定表的最后一条记录的不同方法。首先，我们了解了如何使用派生查询方法来实现它。然后，我们在@Query注解中编写了一个JPQL查询，获得了相同的结果。