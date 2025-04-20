---
layout: post
title:  使用jMolecules进行DDD
category: ddd
copyright: ddd
excerpt: jMolecules
---

## 1. 概述

在本文中，我们将重新讨论关键的[领域驱动设计(DDD)](https://www.baeldung.com/hexagonal-architecture-ddd-spring)概念，并演示如何使用[jMolecules](https://github.com/xmolecules/jmolecules)将这些技术问题表达为元数据。

我们将探讨这种方法如何使我们受益，并讨论jMolecules与Java和Spring生态系统中流行库和框架的集成。

最后，我们将重点关注[ArchUnit](https://www.baeldung.com/java-archunit-intro)集成，并学习如何使用它来在构建过程中强制遵循DDD原则的代码结构。

## 2. jMolecules的目标

jMolecules是一个库，它使我们能够清晰地表达架构概念，从而提高代码的清晰度和可维护性，作者的[研究论文](https://xmolecules.org/aea-paper.pdf)详细阐述了该项目的目标和主要功能。

**总而言之，jMolecules帮助我们使领域特定代码摆脱技术依赖，并通过注解和基于类型的接口表达这些技术概念**。

根据我们选择的方法和设计，我们可以导入相关的jMolecules模块来表达特定于该风格的技术概念。例如，以下是一些受支持的设计风格以及我们可以使用的相关注解：

- 领域驱动设计(DDD)：使用@Entity、@ValueObject、@Repository和@AggregateRoot等注解
- CQRS架构：利用@Command、@CommandHandler和@QueryModel等注解
- 分层架构：应用@DomainLayer、@ApplicationLayer和@InfrastructureLayer等注解

**此外，这些元数据还可以被工具和插件用于生成样板代码、创建文档或确保代码库具有正确的结构等任务**。尽管该项目仍处于早期阶段，但它支持与各种框架和库的[集成](https://github.com/xmolecules/jmolecules-integrations)。

例如，我们可以导入[Jackson](https://www.baeldung.com/jackson)和[Byte-Buddy](https://www.baeldung.com/byte-buddy)集成来生成样板代码，或者包含[JPA](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)和Spring特定模块来将jMolecules注解转换为它们的Spring等效项。

## 3. jMolecules和DDD

在本文中，我们将重点介绍jMolecules的DDD模块，并使用它创建一个博客应用程序的领域模型。首先，我们将[jmolecumes-starter-ddd](https://mvnrepository.com/artifact/org.jmolecules.integrations/jmolecules-starter-ddd)和[jmolecules-starter-test](https://mvnrepository.com/artifact/org.jmolecules.integrations/jmolecules-starter-test)依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>org.jmolecules.integrations</groupId>
    <artifactId>jmolecules-starter-ddd</artifactId>
    <version>0.21.0</version>
</dependency>
<dependency>
    <groupId>org.jmolecules.integrations</groupId>
    <artifactId>jmolecules-starter-test</artifactId>
    <version>0.21.0</version>
    <scope>test</scope>
</dependency>
```

在下面的代码示例中，我们会注意到jMolecules注解与其他框架的注解之间存在相似之处。这是因为[Spring Boot](https://www.baeldung.com/spring-boot)或[JPA](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)等框架也遵循DDD原则，让我们简要回顾一些关键的DDD概念及其相关的注解。

### 3.1 值对象

**值对象是一个不可变的领域对象，它封装了属性和逻辑，但没有明确的标识**。此外，值对象仅由其属性定义。

在文章和博客的语境中，文章的slug是不可变的，并且可以在创建时自行进行验证，这使得它非常适合被标记为@ValueObject：

```java
@ValueObject
class Slug {
    private final String value;

    public Slug(String value) {
        Assert.isTrue(value != null, "Article's slug cannot be null!");
        Assert.isTrue(value.length() >= 5, "Article's slug should be at least 5 characters long!");
        this.value = value;
    }

    // getter
}
```

Java记录本质上是不可变的，这使得它们成为实现值对象的绝佳选择，让我们使用记录来创建另一个@ValueObject来表示帐户Username：

```java
@ValueObject
record Username(String value) {
    public Username {
        Assert.isTrue(value != null && !value.isBlank(), "Username value cannot be null or blank.");
    }
}
```

### 3.2 实体

实体与值对象的区别在于，它们拥有唯一的身份标识并封装了可变的状态。**它们表示需要独特标识的领域概念，并且可以随时间推移进行修改，同时在不同状态下保持其身份**。

例如，我们可以将文章评论想象成一个实体：每条评论都有一个唯一的标识符、一个作者、一条消息和一个时间戳。此外，该实体还可以封装编辑评论消息所需的逻辑：

```java
@Entity
class Comment {
    @Identity
    private final String id;
    private final Username author;
    private String message;
    private Instant lastModified;

    // constructor, getters

    public void edit(String editedMessage) {
        this.message = editedMessage;
        this.lastModified = Instant.now();
    }
}
```

### 3.3 聚合根

在DDD中，[聚合](https://www.baeldung.com/cs/aggregate-root-ddd)是一组相关对象，它们被视为数据变更的单个单元，并在集群中指定一个对象作为根。**聚合根封装了相应的逻辑，以确保对自身以及所有相关实体的变更都发生在单个原子事务中**。

例如，Article将成为我们模型的聚合根。Article可以通过其唯一的slug来识别，并负责管理其content、likes和comments的状态：

```java
@AggregateRoot
class Article {
    @Identity
    private final Slug slug;
    private final Username author;
    private String title;
    private String content;
    private Status status;
    private List<Comment> comments;
    private List<Username> likedBy;

    // constructor, getters

    void comment(Username user, String message) {
        comments.add(new Comment(user, message));
    }

    void publish() {
        if (status == Status.DRAFT || status == Status.HIDDEN) {
            // ...other logic
            status = Status.PUBLISHED;
        }
        throw new IllegalStateException("we cannot publish an article with status=" + status);
    }

    void hide() { /* ... */ }

    void archive() { /* ... */ }

    void like(Username user) { /* ... */ }

    void dislike(Username user) { /* ... */ }
}
```

可以看出，Article实体是包含Comment实体和一些值对象的聚合的根。**聚合不能直接引用其他聚合中的实体**，因此，我们只能通过文章根与评论实体交互，而不能直接从其他聚合或实体交互。

此外，聚合根可以通过其标识符引用其他聚合。例如，Article引用了另一个聚合Author。它通过Username值对象来实现这一点，该值对象是Author聚合根的自然键。

### 3.4 Repository

**Repository是一种抽象概念，它提供了访问、存储和检索聚合根的方法**。从外部来看，它们只是一些简单的聚合集合。

由于我们将Article定义为聚合根，因此我们可以创建Articles类并用@Repository标注，此类将封装与持久层的交互，并提供类似Collection的接口：

```java
@Repository
class Articles {
    Slug save(Article draft) {
        // save to DB
    }

    Optional<Article> find(Slug slug) {
        // query DB
    }

    List<Article> filterByStatus(Status status) {
        // query DB
    }

    void remove(Slug article) {
        // update DB and mark article as removed
    }
}
```

## 4. 执行DDD原则

使用jMolecules注解，我们可以将代码中的架构概念定义为元数据。如前所述，这使我们能够与其他库集成，从而生成样板代码和文档。**不过，在本文的范围内，我们将重点介绍如何使用[archunit](https://mvnrepository.com/artifact/com.tngtech.archunit/archunit)和[jmolecules-archunit](https://mvnrepository.com/artifact/org.jmolecules/jmolecules-archunit)来执行DDD原则**：

```xml
<dependency>
    <groupId>com.tngtech.archunit</groupId>
    <artifactId>archunit</artifactId>
    <version>1.3.0</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.jmolecules</groupId>
    <artifactId>jmolecules-archunit</artifactId>
    <version>1.0.0</version>
    <scope>test</scope>
</dependency>
```

让我们创建一个新的聚合根，并故意打破一些核心的DDD规则。例如，我们可以创建一个没有标识符的Author类，它直接通过对象引用引用Article，而不是使用文章的Slug。此外，我们还可以创建一个Email值对象，其中包含Author实体作为其字段之一，这也违反了DDD原则：

```java
@AggregateRoot
public class Author { // <-- entities and aggregate roots should have an identifier
    private Article latestArticle; // <-- aggregates should not directly reference other aggregates

    @ValueObject
    record Email(
            String address,
            Author author // <-- value objects should not reference entities
    ) {
    }

    // constructor, getter, setter
}
```

现在，让我们编写一个简单的[ArchUnit](https://www.baeldung.com/java-archunit-intro)测试来验证代码结构，**主要的DDD规则已经通过JMoleculesDddRules定义好了**。因此，我们只需要指定要在此测试中验证的包：

```java
@AnalyzeClasses(packages = "cn.tuyucheng.taketoday.dddjmolecules")
class JMoleculesDddUnitTest {
    @ArchTest
    void whenCheckingAllClasses_thenCodeFollowsAllDddPrinciples(JavaClasses classes) {
        JMoleculesDddRules.all().check(classes);
    }
}
```

如果我们尝试构建项目并运行测试，我们将看到以下违规行为：

```text
Author.java: Invalid aggregate root reference! Use identifier reference or Association instead!

Author.java: Author needs identity declaration on either field or method!

Author.java: Value object or identifier must not refer to identifiables!
```

让我们修复错误并确保我们的代码符合最佳实践：

```java
@AggregateRoot
public class Author {
    @Identity
    private Username username;
    private Email email;
    private Slug latestArticle;

    @ValueObject
    record Email(String address) {
    }

    // constructor, getters, setters
}
```

## 5. 总结

在本教程中，我们讨论了如何将技术关注点与业务逻辑分离，以及明确声明这些技术概念的优势。我们发现，jMolecules有助于实现这种分离，并根据所选的架构风格，从架构角度强制执行最佳实践。

此外，我们重新审视了关键的DDD概念，并使用聚合根、实体、值对象和Repository来构建博客网站的领域模型。理解这些概念有助于我们创建健壮的领域模型，而jMolecules与ArchUnit的集成则使我们能够验证最佳的DDD实践。