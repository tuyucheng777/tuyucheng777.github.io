---
layout: post
title:  Blaze Persistence入门
category: persistence
copyright: persistence
excerpt: Blaze Persistence
---

## 1. 简介

在本教程中，我们将讨论在Spring Boot应用程序中使用[Blaze Persistence](https://github.com/Blazebit/blaze-persistence)库。

该库提供了丰富的Criteria API，用于以编程方式创建SQL查询，它允许我们应用各种过滤器、函数和逻辑条件。

我们将介绍项目设置，提供一些如何创建查询的示例以及如何将实体映射到DTO对象。

## 2. Maven依赖

为了在我们的项目中包含Blaze Persistence核心，我们需要在pom.xml文件中添加依赖[blaze-persistence-core-api-jakarta](https://mvnrepository.com/artifact/com.blazebit/blaze-persistence-core-api-jakarta)、[blaze-persistence-core-impl-jakarta](https://mvnrepository.com/artifact/com.blazebit/blaze-persistence-core-impl)和[blaze-persistence-integration-hibernate-6.2](https://mvnrepository.com/artifact/com.blazebit/blaze-persistence-integration-hibernate-6.2/1.6.9)：

```xml
<dependency>
    <groupId>com.blazebit</groupId>
    <artifactId>blaze-persistence-core-api-jakarta</artifactId>
    <scope>compile</scope>
</dependency>
<dependency>
    <groupId>com.blazebit</groupId>
    <artifactId>blaze-persistence-core-impl-jakarta</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>com.blazebit</groupId>
    <artifactId>blaze-persistence-integration-hibernate-6.2</artifactId>
    <scope>runtime</scope>
</dependency>
```

根据我们使用的Hibernate版本，后者的依赖可能会有所不同。

## 3. 实体模型

首先，我们来定义一下示例中使用的数据模型。为了自动创建表，我们将使用Hibernate。

我们有两个实体，Person和Post，它们使用一对多关系连接：

```java
@Entity
public class Person {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    private int age;

    @OneToMany(mappedBy = "author")
    private Set<Post> posts = new HashSet<>();
}
```

```java
@Entity
public class Post {

    @Id
    @GeneratedValue
    private Long id;

    private String title;

    private String content;

    @ManyToOne(fetch = FetchType.LAZY)
    private Person author;
}
```

## 4. Criteria API

Blaze Persistence库是[JPA Criteria API](https://www.baeldung.com/hibernate-criteria-queries)的替代方案，这两个API都使我们能够在运行时定义动态查询。

然而，JPA Criteria API在开发人员中并不那么受欢迎，因为它难以读写。相比之下，Blaze Persistence的设计更加用户友好，更易于使用。此外，它与各种JPA实现集成，并提供了丰富的查询功能。

### 4.1 配置

**为了使用Blaze Persistence Criteria API，我们需要在配置类中定义CriteriaBuilderFactory Bean**：

```java
@Autowired
private EntityManagerFactory entityManagerFactory;

@Bean
public CriteriaBuilderFactory createCriteriaBuilderFactory() {
    CriteriaBuilderConfiguration config = Criteria.getDefault();
    return config.createCriteriaBuilderFactory(entityManagerFactory);
}
```

### 4.2 基本查询

现在，让我们从一个简单的查询开始，该查询从数据库中选择每个Post，我们只需要两个方法调用来定义和执行查询：

```java
List<Post> posts = builderFactory.create(entityManager, Post.class).getResultList();
```

create方法创建一个查询，而getResultList方法的调用返回查询返回的结果。

此外，在create方法中，Post.class参数有几个用途：

- 标识查询的结果类型
- 标识隐式查询根
- 为Post表添加隐式SELECT和FROM子句

一旦执行，查询将生成以下JPQL：

```sql
SELECT post
FROM Post post;
```

### 4.3 Where子句

我们可以在create方法之后调用where方法来在我们的标准构建器中添加WHERE子句。

让我们看看如何获取由至少发表过两篇帖子并且年龄在18至40岁之间的人所发表的帖子：

```java
CriteriaBuilder<Person> personCriteriaBuilder = builderFactory.create(entityManager, Person.class, "p")
    .where("p.age")
        .betweenExpression("18")
        .andExpression("40")
    .where("SIZE(p.posts)").geExpression("2")
    .orderByAsc("p.name")
    .orderByAsc("p.id");
```

**由于Blaze Persistence支持直接函数调用语法，我们可以轻松检索与该人相关的帖子的大小**。

**此外，我们可以通过调用whereAnd或whereOr方法来定义复合谓词**。它们返回构建器实例，我们可以使用该实例通过一次或多次调用where方法来定义嵌套的复合谓词。完成后，我们需要调用endAnd或endOr方法来关闭复合谓词。

例如，让我们创建一个查询来选择具有特定标题或作者姓名的帖子：

```java
CriteriaBuilder<Post> postCriteriaBuilder = builderFactory.create(entityManager, Post.class, "p")
    .whereOr()
        .where("p.title").like().value(title + "%").noEscape()
        .where("p.author.name").eq(authorName)
    .endOr();
```

### 4.4 From子句

FROM子句包含需要查询的实体，如前所述，我们可以在create方法中指定根实体。但是，我们可以定义from子句来指定根实体。这样，隐式创建的查询根将被删除：

```java
CriteriaBuilder<Post> postCriteriaBuilder = builderFactory.create(entityManager, Post.class)
    .from(Person.class, "person")
    .select("person.posts");
```

在这个例子中，Post.class参数只定义返回类型。

由于我们从不同的表中进行选择，因此构建器将在生成的查询中添加隐式JOIN：

```sql
SELECT posts_1 
FROM Person person 
LEFT JOIN person.posts posts_1;
```

## 5. 实体视图模块

**Blaze Persistence实体视图模块试图解决实体类和DTO类之间的高效映射问题**，使用此模块，我们可以将DTO类定义为接口，并使用注解提供到实体类的映射。

### 5.1 Maven依赖

我们需要在项目中包含额外的[实体视图依赖](https://mvnrepository.com/artifact/com.blazebit/blaze-persistence-entity-view-api-jakarta)：

```xml
<dependency>
    <groupId>com.blazebit</groupId>
    <artifactId>blaze-persistence-entity-view-api-jakarta</artifactId>
</dependency>
<dependency>
    <groupId>com.blazebit</groupId>
    <artifactId>blaze-persistence-entity-view-impl-jakarta</artifactId>
</dependency>
<dependency>
    <groupId>com.blazebit</groupId>
    <artifactId>blaze-persistence-entity-view-processor-jakarta</artifactId>
</dependency>
```

### 5.2 配置

此外，我们需要一个注册了实体视图类的EntityViewManager Bean：

```java
@Bean
public EntityViewManager createEntityViewManager(CriteriaBuilderFactory criteriaBuilderFactory, EntityViewConfiguration entityViewConfiguration) {
    return entityViewConfiguration.createEntityViewManager(criteriaBuilderFactory);
}
```

**因为EntityViewManager同时绑定到EntityManagerFactory和CriteriaBuilderFactory，所以它的范围应该相同**。

### 5.3 映射

实体视图表示是一个简单的接口或一个抽象类，描述我们想要的投影的结构。

让我们创建一个接口来代表Post类的实体视图：

```java
@EntityView(Post.class)
public interface PostView {
  
    @IdMapping
    Long getId();

    String getTitle();

    String getContent();
}
```

**我们需要用@EntityView注解来标注我们的接口并提供一个实体类**。

虽然并非强制要求，但我们应该尽可能使用@IdMapping注解定义id映射，没有此类映射的实体视图会使用equals和hashCode实现来考虑所有属性，而使用id映射时，只会考虑id。

但是，如果我们想为Getter方法使用不同的名称，可以添加@Mapping注解。使用此注解，我们也可以定义整个表达式：

```java
@Mapping("UPPER(title)")
String getTitle();
```

因此，映射将返回Post实体的大写标题。

此外，我们可以扩展实体视图。假设我们想要定义一个视图，它返回一篇包含附加作者信息的文章。

首先，我们定义一个PersonView接口：

```java
@EntityView(Person.class)
public interface PersonView {
    
    @IdMapping
    Long getId();

    int getAge();

    String getName();
}
```

其次，我们定义一个扩展PostView接口的新接口和一个返回PersonView信息的方法：

```java
@EntityView(Post.class)
public interface PostWithAuthorView extends PostView {
    PersonView getAuthor();
}
```

最后，让我们将视图与条件构建器一起使用，它们可以直接应用于查询。

我们可以定义一个基本查询，然后创建映射：

```java
CriteriaBuilder<Post> postCriteriaBuilder = builderFactory.create(entityManager, Post.class, "p")
    .whereOr()
        .where("p.title").like().value("title%").noEscape()
        .where("p.author.name").eq(authorName)
    .endOr();

CriteriaBuilder<PostWithAuthorView> postWithAuthorViewCriteriaBuilder =
    viewManager.applySetting(EntityViewSetting.create(PostWithAuthorView.class), postCriteriaBuilder);
```

上面的代码将创建一个优化的查询并根据结果构建我们的实体视图：

```sql
SELECT p.id AS PostWithAuthorView_id,
       p.author.id AS PostWithAuthorView_author_id,
       author_1.age AS PostWithAuthorView_author_age,
       author_1.name AS PostWithAuthorView_author_name,
       p.content AS PostWithAuthorView_content,
       UPPER(p.title) AS PostWithAuthorView_title
FROM cn.tuyucheng.taketoday.model.Post p
         LEFT JOIN p.author author_1
WHERE p.title LIKE REPLACE(:param_0, '\\', '\\\\')
   OR author_1.name = :param_1
```

### 5.4 实体视图和Spring Data

除了与Spring集成之外，Blaze Persistence还提供了Spring Data集成模块，使实体视图的使用就像使用实体一样方便。

此外，我们还需要包含[Spring集成依赖](https://mvnrepository.com/artifact/com.blazebit/blaze-persistence-integration-spring-data-base-3.1)：

```xml
<dependency>
    <groupId>com.blazebit</groupId>
    <artifactId>blaze-persistence-integration-spring-data-3.1</artifactId>
</dependency>
```

此外，为了启用Spring Data，我们需要在配置类上添加@EnableBlazeRepositories注解。此外，我们还可以指定Repository类扫描的基础包。

**该集成带有一个基本EntityViewRepository接口，我们可以使用它来定义我们的Repository**。

现在，让我们定义一个与PostWithAuthorView一起使用的接口：

```java
@Repository
@Transactional(readOnly = true)
public interface PostViewRepository extends EntityViewRepository<PostWithAuthorView, Long> {
}
```

在这里，我们的接口继承了最常用的Repository方法，例如findAll，findOne和exist。如果需要，我们可以使用Spring Data JPA方法命名约定定义自己的方法。

## 6. 总结

在本文中，我们学习了如何使用Blaze Persistence库配置和创建简单查询。