---
layout: post
title:  在Spring Data JPA查询中使用枚举
category: springdata
copyright: springdata
excerpt: Spring Data JPA
---

## 1. 概述

在使用[Spring Data JPA](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)构建持久层时，我们经常使用带有[枚举](https://www.baeldung.com/a-guide-to-java-enums)字段的实体。这些枚举字段代表一组固定的常量，例如订单的状态、用户的角色或发布系统中文章的阶段。

根据枚举字段查询实体是一个常见的需求，Spring Data JPA提供了几种方法来实现这一点。

在本教程中，我们将探讨如何**使用标准JPA方法和原生查询查询实体类中声明的枚举字段**。

## 2. 应用程序设置

### 2.1 数据模型

首先，让我们定义数据模型，包括一个枚举字段。我们示例中的中心实体是Article类，它声明了一个枚举字段ArticleStage来表示文章可以处于的不同阶段：

```java
public enum ArticleStage {
    TODO, IN_PROGRESS, PUBLISHED
}
```

ArticleStage枚举包含三个可能的阶段，代表文章从最初创建到最终发布状态的生命周期。

接下来，让我们使用ArticleStage枚举字段创建Article实体类 ：

```java
@Entity
@Table(name = "articles")
public class Article {

    @Id
    private UUID id;

    private String title;

    private String author;

    @Enumerated(EnumType.STRING)
    private ArticleStage stage;

    // standard constructors, getters and setters
}
```

我们将Article实体类映射到articles数据库表。此外，**我们使用@Enumerated注解来指定stage字段应作为字符串保留在数据库中**。

### 2.2 Repository层

定义了数据模型后，我们现在可以创建一个扩展JpaRepository的Repository接口来与我们的数据库进行交互：

```java
@Repository
public interface ArticleRepository extends JpaRepository<Article, UUID> {
}
```

在接下来的部分中，我们将向该接口添加查询方法，以探索通过枚举字段查询Article实体的不同方法。

## 3. 标准JPA查询方法

Spring Data JPA允许我们使用方法名称在Repository接口中定义[派生查询方法](https://www.baeldung.com/spring-data-derived-queries)，这种方法非常适合简单查询。

让我们研究一下如何使用它来查询实体类中的枚举字段。

### 3.1 通过单个枚举值查询

我们可以通过在ArticleRepository接口中定义一个方法，通过单个ArticleStage枚举值来查找文章：

```java
List<Article> findByStage(ArticleStage stage);
```

Spring Data JPA将根据方法名称生成适当的SQL查询。

我们还可以将stage参数与其他字段组合起来，以创建更具体的查询。例如，我们可以声明一个方法，通过其title和stage来查找文章：

```java
Article findByTitleAndStage(String title, ArticleStage stage);
```

我们将使用[Instancio](https://www.baeldung.com/java-test-data-instancio)生成测试文章数据并测试这些查询：

```java
Article article = Instancio.create(Article.class);
articleRepository.save(article);

List<Article> retrievedArticles = articleRepository.findByStage(article.getStage());

assertThat(retrievedArticles).element(0).usingRecursiveComparison().isEqualTo(article);
```

```java
Article article = Instancio.create(Article.class);
articleRepository.save(article);

Article retrievedArticle = articleRepository.findByTitleAndStage(article.getTitle(), article.getStage());

assertThat(retrievedArticle).usingRecursiveComparison().isEqualTo(article);
```

### 3.2 通过多个枚举值进行查询

我们还可以通过多个ArticleStage枚举值查找文章：

```java
List<Article> findByStageIn(List<ArticleStage> stages);
```

Spring Data JPA将生成一个SQL查询，该查询使用IN子句来查找stage与任何提供的值匹配的文章。

为了验证我们声明的方法是否按预期工作，让我们对其进行测试：

```java
List<Article> articles = Instancio.of(Article.class).stream().limit(100).toList();
articleRepository.saveAll(articles);

List<ArticleStage> stagesToQuery = List.of(ArticleStage.TODO, ArticleStage.IN_PROGRESS);
List<Article> retrievedArticles = articleRepository.findByStageIn(stagesToQuery);

assertThat(retrievedArticles)
    .isNotEmpty()
    .extracting(Article::getStage)
    .doesNotContain(ArticleStage.PUBLISHED)
    .hasSameElementsAs(stagesToQuery);
```

## 4. 原生查询

除了上一节中探讨的标准JPA方法之外，Spring Data JPA还支持原生SQL查询。原生查询对于执行复杂的SQL查询很有用，并允许我们调用特定于数据库的函数。

此外，我们可以使用带有[@Query](https://www.baeldung.com/spring-data-jpa-query)注解的[SpEL(Spring Expression Language)](https://www.baeldung.com/spring-expression-language)来根据方法参数构建动态查询。

让我们看看如何使用SpEL的原生查询通过ArticleStage枚举值查询我们的实体类Article。

### 4.1 通过单个枚举值查询

要使用原生查询通过单个枚举值查询文章记录，我们可以在ArticleRepository接口中定义一个方法并使用@Query注解对其进行标注：

```java
@Query(nativeQuery = true, value = "SELECT * FROM articles WHERE stage = :#{#stage?.name()}")
List<Article> getByStage(@Param("stage") ArticleStage stage);
```

**我们将nativeQuery属性设置为true，以表明我们正在使用原生SQL查询而不是默认的JPQL定义**。

**我们在查询中使用SpEL表达式:#{#stage?.name()}来引用传递给方法参数的枚举值，表达式中的?运算符用于优雅地处理空输入**。

让我们验证一下原生查询方法是否按预期工作：

```java
Article article = Instancio.create(Article.class);
articleRepository.save(article);

List<Article> retrievedArticles = articleRepository.getByStage(article.getStage());

assertThat(retrievedArticles).element(0).usingRecursiveComparison().isEqualTo(article);
```

### 4.2 通过多个枚举值进行查询

要使用原生查询通过多个枚举值查询文章记录，我们可以在ArticleRepository接口中定义另一种方法：

```java
@Query(nativeQuery = true, value = "SELECT * FROM articles WHERE stage IN (:#{#stages.![name()]})")
List<Article> getByStageIn(@Param("stages") List<ArticleStage> stages);
```

为了实现这种情况，我们在SQL查询中使用IN子句来获取stage与任何提供的值匹配的文章。

**SpEL表达式#stages.![name()\]将枚举值列表转换为表示其名称的字符串列表**。

让我们看看这个方法的行为：

```java
List<Article> articles = Instancio.of(Article.class).stream().limit(100).toList();
articleRepository.saveAll(articles);

List<ArticleStage> stagesToQuery = List.of(ArticleStage.TODO, ArticleStage.IN_PROGRESS);
List<Article> retrievedArticles = articleRepository.findByStageIn(stagesToQuery);

assertThat(retrievedArticles)
    .isNotEmpty()
    .extracting(Article::getStage)
    .doesNotContain(ArticleStage.PUBLISHED)
    .hasSameElementsAs(stagesToQuery);
```

## 5. 总结

在本文中，我们探讨了如何使用Spring Data JPA查询实体类中的枚举字段，我们研究了标准JPA方法和使用SpEL的原生查询来实现此目的。

我们学习了如何使用单个和多个枚举值查询实体，标准JPA方法提供了一种简洁直接的查询枚举字段的方法，而原生查询则提供了更多控制和灵活性来执行复杂的SQL查询。