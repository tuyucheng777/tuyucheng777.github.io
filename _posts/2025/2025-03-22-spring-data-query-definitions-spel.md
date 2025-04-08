---
layout: post
title:  Spring Data JPA中支持SpEL的@Query定义
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 概述

[SpEL](https://www.baeldung.com/spring-expression-language)代表Spring表达式语言，是一个强大的工具，可以显著增强我们与Spring的交互，并提供对配置、属性设置和查询操作的额外抽象。

在本教程中，我们将学习如何使用此工具使自定义查询更加动态，并在Repository层中隐藏特定于数据库的操作。我们将使用[@Query](https://www.baeldung.com/spring-data-jpa-query)注解，它允许我们使用[JPQL](https://www.baeldung.com/jpql-hql-criteria-query#jpql)或原生SQL来自定义与数据库的交互。

## 2. 访问参数

我们首先检查一下如何使用SpEL来处理方法参数。

### 2.1 通过索引访问

**通过索引访问参数并不是最佳选择，因为它可能会给代码带来难以调试的问题**，尤其是当参数具有相同类型时。

同时，它为我们提供了更大的灵活性，尤其是在参数名称经常变化的开发阶段。IDE可能无法正确处理代码和查询中的更新。

**[JDBC](https://www.baeldung.com/java-jdbc)为我们提供了?占位符，我们可以使用它来标识参数在查询中的位置**，Spring支持此约定并允许编写以下内容：

```java
@Modifying
@Transactional
@Query(value = "INSERT INTO articles (id, title, content, language) "
        + "VALUES (?1, ?2, ?3, ?4)",
        nativeQuery = true)
void saveWithPositionalArguments(Long id, String title, String content, String language);
```

到目前为止，没有发生任何有趣的事情，我们使用的方法与之前在JDBC应用程序中使用的方法相同。**请注意，任何对数据库进行更改的查询都需要[@Modifying](https://www.baeldung.com/spring-data-jpa-modifying-annotation)和[@Transactional](https://www.baeldung.com/transaction-configuration-with-jpa-and-spring)注解，[INSERT](https://www.baeldung.com/jpa-insert)就是其中之一**。INSERT的所有示例都将使用原生查询，因为JPQL不支持它们。

我们可以使用SpEL重写上面的查询：

```java
@Modifying
@Transactional
@Query(value = "INSERT INTO articles (id, title, content, language) "
        + "VALUES (?#{[0]}, ?#{[1]}, ?#{[2]}, ?#{[3]})",
        nativeQuery = true)
void saveWithPositionalSpELArguments(long id, String title, String content, String language);
```

结果类似，但看起来比前一个更混乱。然而，由于它是SpEL，因此为我们提供了所有丰富的功能。例如，我们可以在查询中使用条件逻辑：

```java
@Modifying
@Transactional
@Query(value = "INSERT INTO articles (id, title, content, language) "
        + "VALUES (?#{[0]}, ?#{[1]}, ?#{[2] ?: 'Empty Article'}, ?#{[3]})",
        nativeQuery = true)
void saveWithPositionalSpELArgumentsWithEmptyCheck(long id, String title, String content, String isoCode);
```

我们在此查询中使用了[Elvis运算符](https://www.baeldung.com/kotlin/elvis-operator)来检查是否提供了内容。尽管我们可以在查询中编写更复杂的逻辑，但应谨慎使用它，因为它可能会带来调试和验证代码的问题。

### 2.2 通过名称访问

**访问参数的另一种方法是使用命名占位符，它通常与参数名称匹配，但这不是严格要求**。这是JDBC的另一个约定；命名参数用:name占位符标记，我们可以直接使用它：

```java
@Modifying
@Transactional
@Query(value = "INSERT INTO articles (id, title, content, language) "
        + "VALUES (:id, :title, :content, :language)",
        nativeQuery = true)
void saveWithNamedArguments(@Param("id") long id, @Param("title") String title,
                            @Param("content") String content, @Param("isoCode") String language);
```

唯一需要做的额外事情是确保Spring知道参数的名称，**我们可以以更隐式的方式执行此操作，并使用-parameters标志编译代码或显式执行此操作使用[@Param](https://www.baeldung.com/spring-data-jpa-query#1-jpql-3)注解**。

显式方式总是更好，因为它提供了对名称的更多控制，并且我们不会因为不正确的编译而遇到问题。

不过，让我们使用SpEL重写相同的查询：

```java
@Modifying
@Transactional
@Query(value = "INSERT INTO articles (id, title, content, language) "
        + "VALUES (:#{#id}, :#{#title}, :#{#content}, :#{#language})",
        nativeQuery = true)
void saveWithNamedSpELArguments(@Param("id") long id, @Param("title") String title,
                                @Param("content") String content, @Param("language") String language);
```

这里，我们有标准的SpEL语法，但此外，我们需要使用#来区分参数名称和应用程序Bean。**如果我们省略它，Spring将尝试在上下文中寻找名称为id、title、content和language的Bean**。

总的来说，这个版本与没有SpEL的简单方法非常相似。然而，正如上一节所讨论的，SpEL提供了更多的能力和功能。例如，我们可以调用传递的对象上可用的函数：

```java
@Modifying
@Transactional
@Query(value = "INSERT INTO articles (id, title, content, language) "
        + "VALUES (:#{#id}, :#{#title}, :#{#content}, :#{#language.toLowerCase()})",
        nativeQuery = true)
void saveWithNamedSpELArgumentsAndLowerCaseLanguage(@Param("id") long id, @Param("title") String title,
                                                    @Param("content") String content, @Param("language") String language);
```

我们可以在String对象上使用[toLowerCase()](https://www.baeldung.com/string/to-lower-case)方法，我们可以执行条件逻辑、方法调用、字符串拼接等。**同时，@Query内部的逻辑过多可能会使其变得模糊，并容易将业务逻辑泄露到基础架构代码中**。

### 2.3 访问对象的字段

虽然以前的方法或多或少反映了JDBC和[预编译查询](https://www.baeldung.com/java-statement-preparedstatement#preparedstatement)的功能，但这种方法允许我们以更面向对象的方式使用原生查询。正如我们之前看到的，我们可以使用简单的逻辑并调用SpEL中对象的方法。此外，我们还可以访问对象的字段：

```java
@Modifying
@Transactional
@Query(value = "INSERT INTO articles (id, title, content, language) "
        + "VALUES (:#{#article.id}, :#{#article.title}, :#{#article.content}, :#{#article.language})",
        nativeQuery = true)
void saveWithSingleObjectSpELArgument(@Param("article") Article article);
```

我们可以使用对象的公共API来获取其内部结构，**这是一项非常有用的技术，因为它允许我们保持Repository的签名整洁并且不会暴露太多信息**。它甚至允许我们达到到嵌套对象，假设我们有一个文章包装器：

```java
public class ArticleWrapper {
    private final Article article;
    public ArticleWrapper(Article article) {
        this.article = article;
    }
    public Article getArticle() {
        return article;
    }
}
```

我们可以在我们的示例中使用它：

```java
@Modifying
@Transactional
@Query(value = "INSERT INTO articles (id, title, content, language) "
        + "VALUES (:#{#wrapper.article.id}, :#{#wrapper.article.title}, "
        + ":#{#wrapper.article.content}, :#{#wrapper.article.language})",
        nativeQuery = true)
void saveWithSingleWrappedObjectSpELArgument(@Param("wrapper") ArticleWrapper articleWrapper);
```

因此，我们可以将参数视为SpEL中的Java对象，并使用任何可用的字段或方法。我们也可以向该查询添加逻辑和方法调用。

**此外，我们可以将此技术与[Pageable](https://www.baeldung.com/spring-data-jpa-pagination-sorting#paginate)结合使用来从对象获取信息，例如偏移量或页面大小，以及将其添加到我们的原生查询中**。虽然[Sort](https://www.baeldung.com/spring-data-jpa-pagination-sorting#sort)也是一个对象，但它具有更复杂的结构，并且会是更难使用。

## 3. 引用实体

减少重复代码是一个很好的做法，然而，自定义查询可能会使其变得具有挑战性。即使我们有类似的逻辑来提取到基础Repository，表的名称也不同，因此很难重用它们。

SpEL为实体名称提供占位符，该占位符是从Repository参数化中推断出来的。让我们创建一个这样的基础Repository：

```java
@NoRepositoryBean
public interface BaseNewsApplicationRepository<T, ID> extends JpaRepository<T, ID> {
    @Query(value = "select e from #{#entityName} e")
    List<Article> findAllEntitiesUsingEntityPlaceholder();

    @Query(value = "SELECT * FROM #{#entityName}", nativeQuery = true)
    List<Article> findAllEntitiesUsingEntityPlaceholderWithNativeQuery();
}
```

**我们必须使用一些额外的注解才能使其工作**，第一个是[@NoRepositoryBean](https://www.baeldung.com/spring-data-jpa-method-in-all-repositories#defining-a-base-repository-interface)，我们需要它来从实例化中排除这个基础Repository。由于它没有特定的参数化，尝试创建这样的Repository将导致上下文失败。因此，我们需要将其排除。

使用JPQL的查询非常简单，将使用给定Repository的实体名称：

```java
@Query(value = "select e from #{#entityName} e")
List<Article> findAllEntitiesUsingEntityPlaceholder();
```

但是，原生查询的情况并不那么简单。**无需任何其他更改和配置，它将尝试使用实体名称(在我们的示例中为Article)来找到表**：

```java
@Query(value = "SELECT * FROM #{#entityName}", nativeQuery = true)
List<Article> findAllEntitiesUsingEntityPlaceholderWithNativeQuery();
```

但是，我们的数据库中没有这样的表。在实体定义中，我们明确指出了表的名称：

```java
@Entity
@Table(name = "articles")
public class Article {
    // ...
}
```

为了解决这个问题，我们需要向我们的表提供名称匹配的实体：

```java
@Entity(name = "articles")
@Table(name = "articles")
public class Article {
    // ...
}
```

在这种情况下，JPQL和原生查询都将推断出正确的实体名称，并且我们将能够在应用程序中的所有实体中重用相同的基本查询。

## 4. 添加SpEL上下文

正如所指出的，在引用参数或占位符时，我们必须在其名称之前提供额外的#，这样做是为了区分Bean名称和参数名称。

但是，我们不能直接在查询中使用Spring上下文中的Bean，IDE通常会从上下文中提供有关Bean的提示，但上下文会失败。**发生这种情况是因为[@Value](https://www.baeldung.com/spring-value-annotation)和@Query的处理方式不同**，我们可以引用前者的上下文中的Bean，但不能引用后者的。

**同时，我们可以使用EvaluationContextExtension在SpEL上下文中注册Bean，这样就可以在@Query中使用它们**。让我们想象以下情况-我们希望从数据库中找出所有文章，但根据用户的语言环境设置对它们进行过滤：

```java
@Query(value = "SELECT * FROM articles WHERE language = :#{locale.language}", nativeQuery = true)
List<Article> findAllArticlesUsingLocaleWithNativeQuery();
```

此查询将失败，因为默认情况下我们无法访问该语言环境，我们需要提供自定义的EvaluationContextExtension来保存有关用户区域设置的信息：

```java
@Component
public class LocaleContextHolderExtension implements EvaluationContextExtension {

    @Override
    public String getExtensionId() {
        return "locale";
    }

    @Override
    public Locale getRootObject() {
        return LocaleContextHolder.getLocale();
    }
}
```

我们可以使用LocaleContextHolder访问应用程序中任意位置的当前区域设置，**唯一需要注意的是，它与用户的请求相关，并且在此范围之外无法访问**，我们需要提供根对象和名称。我们还可以选择性地添加属性和函数，但在本例中我们仅使用根对象。

在能够在@Query中使用区域设置之前，我们还需要执行另一个步骤是注册[语言环境拦截器](https://www.baeldung.com/spring-boot-internationalization#localechangeinterceptor)：

```java
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        LocaleChangeInterceptor localeChangeInterceptor = new LocaleChangeInterceptor();
        localeChangeInterceptor.setParamName("locale");
        registry.addInterceptor(localeChangeInterceptor);
    }
}
```

在这里，我们可以添加有关我们将跟踪的参数的信息，因此每当请求包含区域设置参数时，上下文中的区域设置都会更新。可以通过在请求中提供区域设置来检查逻辑：

```java
@ParameterizedTest
@CsvSource({"eng,2","fr,2", "esp,2", "deu, 2","jp,0"})
void whenAskForNewsGetAllNewsInSpecificLanguageBasedOnLocale(String language, int expectedResultSize) {
    webTestClient.get().uri("/articles?locale=" + language)
            .exchange()
            .expectStatus().isOk()
            .expectBodyList(Article.class)
            .hasSize(expectedResultSize);
}
```

**EvaluationContextExtension可用于显著增强SpEL的功能，尤其是在使用@Query注解时**。使用此功能的方法包括：架构之间的功能标记和交互的安全性和角色限制。

## 5. 总结

SpEL是一个强大的工具，与所有强大的工具一样，人们往往会过度使用它们并试图仅使用它来解决所有问题。最好合理地使用复杂的表达式，并且只在必要的情况下使用。

虽然IDE提供SpEL支持和突出显示，但复杂的逻辑可能会隐藏难以调试和验证的错误。因此，请谨慎使用SpEL，并避免可能会导致错误的“智能代码”。在Java中更好地表达，而不是隐藏在SpEL中。