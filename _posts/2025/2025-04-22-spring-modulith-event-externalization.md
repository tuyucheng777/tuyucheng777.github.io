---
layout: post
title:  使用Spring Modulith进行事件外部化
category: springboot
copyright: springboot
excerpt: Spring Modulith
---

## 1. 概述

在本文中，我们将讨论在[@Transactional](https://www.baeldung.com/transaction-configuration-with-jpa-and-spring)块中发布消息的必要性以及相关的性能挑战，例如延长数据库连接时间。为了解决这个问题，我们将利用[Spring Modulith](https://www.baeldung.com/spring-modulith)的功能来监听Spring应用程序事件并自动将其发布到[Kafka](https://www.baeldung.com/apache-kafka)主题。

## 2. 事务操作和消息代理

对于本文的代码示例，我们假设正在编写负责保存文章的功能：

```java
@Service
class Tuyucheng {
    private final ArticleRepository articleRepository;

    // constructor

    @Transactional
    public void createArticle(Article article) {
        validateArticle(article);
        article = addArticleTags(article);
        // ... other business logic

        articleRepository.save(article);
    }
}
```

此外，我们需要将这篇新文章通知系统的其他部分。有了这些信息，其他模块或服务将做出相应的反应，创建报告或向网站读者发送新闻通讯。

实现此目的的最简单方法是注入一个知道如何发布此事件的依赖，在我们的示例中，让我们使用KafkaOperations向“tuyucheng.articles.published”主题发送一条消息，并使用Article的slug()作为键：

```java
@Service
class Tuyucheng {
    private final ArticleRepository articleRepository;
    private final KafkaOperations<String, ArticlePublishedEvent> messageProducer;

    // constructor

    @Transactional
    public void createArticle(Article article) {
        // ... business logic
        validateArticle(article);
        article = addArticleTags(article);
        article = articleRepository.save(article);

        messageProducer.send(
                "tuyucheng.articles.published",
                article.slug(),
                new ArticlePublishedEvent(article.slug(), article.title())
        ).join();
    }
}
```

然而，由于一些不同的原因，这种方法并不理想。从设计的角度来看，我们将领域服务与消息生产者耦合在一起。此外，领域服务直接依赖于底层组件，这违反了[“清洁架构”](https://www.baeldung.com/spring-boot-clean-architecture)的一项基本规则。

此外，这种方法也会对性能产生影响，因为所有操作都发生在@Transactional方法中。**因此，用于保存文章的数据库连接将一直保持打开状态，直到消息成功发布为止**。

最后，该解决方案还在持久化数据和发布消息之间创建了一种容易出错的关系：

- **如果生产者发布消息失败，则事务回滚**；
- **即使消息已经发布，事务最终也可以回滚**；

## 3. 使用Spring Events实现依赖反转

**我们可以利用[Spring Events](https://www.baeldung.com/spring-events)来改进解决方案的设计**，我们的目标是避免直接从领域服务向Kafka发布消息，让我们移除对KafkaOperations的依赖，并改为发布一个内部应用程序事件：

```java
@Service
public class Tuyucheng {
    private final ApplicationEventPublisher applicationEvents;
    private final ArticleRepository articleRepository;

    // constructor

    @Transactional
    public void createArticle(Article article) {
        // ... business logic
        validateArticle(article);
        article = addArticleTags(article);
        article = articleRepository.save(article);

        applicationEvents.publishEvent(
                new ArticlePublishedEvent(article.slug(), article.title()));
    }
}
```

除此之外，我们将在基础架构层中引入一个专用的Kafka生产者，该组件将监听ArticlePublishedEvent事件，并将发布委托给底层的KafkaOperations Bean：

```java
@Component
class ArticlePublishedKafkaProducer {
    private final KafkaOperations<String, ArticlePublishedEvent> messageProducer;

    // constructor 

    @EventListener
    public void publish(ArticlePublishedEvent article) {
        Assert.notNull(article.slug(), "Article Slug must not be null!");
        messageProducer.send("tuyucheng.articles.published", article.splug(), event);
    }
}
```

**通过这种抽象，基础架构组件现在依赖于领域服务生成的事件。换句话说，我们成功地降低了耦合度，并反转了源代码依赖关系**。此外，如果其他模块对文章创建感兴趣，它们现在可以无缝地监听这些应用程序事件并做出相应的响应。

另一方面，publish()方法将在与我们的业务逻辑相同的事务中调用。间接地，这两个操作仍然是耦合的，因为其中一个操作失败会导致另一个操作失败或回滚。

## 4. 原子操作与非原子操作

现在，让我们深入探讨性能方面的考虑。首先，我们必须确定当与消息代理的通信失败时回滚是否是理想的行为，此选择因具体情况而异。

**如果我们不需要这种原子性，则必须释放数据库连接并异步发布事件**。为了模拟这种情况，我们可以尝试创建一篇不带slug的文章，导致ArticlePublishedKafkaProducer::publish失败：

```java
@Test
void whenPublishingMessageFails_thenArticleIsStillSavedToDB() {
    var article = new Article(null, "Introduction to Spring Boot", "John Doe", "<p> Spring Boot is [...] </p>");

    tuyucheng.createArticle(article);

    assertThat(repository.findAll())
        .hasSize(1).first()
        .extracting(Article::title, Article::author)
        .containsExactly("Introduction to Spring Boot", "John Doe");
}
```

如果我们现在运行测试，它将失败，这是因为ArticlePublishedKafkaProducer抛出了一个异常，导致领域服务回滚事务。**但是，我们可以通过将@EventListener注解替换为@TransactionalEventListener和@Async，使事件监听器变为异步的**：

```java
@Async
@TransactionalEventListener
public void publish(ArticlePublishedEvent event) {
    Assert.notNull(event.slug(), "Article Slug must not be null!");
    messageProducer.send("tuyucheng.articles.published", event);
}
```

如果我们现在重新运行测试，我们会注意到异常已被记录，事件未被发布，并且实体已保存到数据库中。此外，数据库连接被更快地释放，从而允许其他线程使用它。

## 5. 使用Spring Modulith进行事件外部化

我们通过两步方法成功解决了原始代码示例的设计和性能问题：

- 使用Spring应用程序事件实现依赖反转
- 利用@TransactionalEventListener和@Async进行异步发布

**Spring Modulith允许我们进一步简化代码，并提供对这种模式的内置支持**。首先，将[spring-modulith-events-api](https://central.sonatype.com/artifact/org.springframework.modulith/spring-modulith-events-api/versions)的Maven依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>org.springframework.modulith</groupId>
    <artifactId>spring-modulith-events-api</artifactId>
    <version>1.1.3</version>
</dependency>
```

**此模块可配置为监听应用程序事件并自动将其外部化到[各种消息系统](https://docs.spring.io/spring-modulith/reference/events.html#externalization.infrastructure)**，我们将沿用原先的示例，并重点介绍Kafka。为了实现此集成，我们需要添加[spring-modulith-events-kafka](https://central.sonatype.com/artifact/org.springframework.modulith/spring-modulith-events-kafka/versions)依赖：

```xml
<dependency> 
    <groupId>org.springframework.modulith</groupId> 
    <artifactId>spring-modulith-events-kafka</artifactId> 
    <version>1.1.3</version>
    <scope>runtime</scope> 
</dependency>
```

现在，我们需要更新ArticlePublishedEvent并使用@Externalized注解，此注解需要路由目标的名称和键，换句话说，就是Kafka主题和消息键。对于键，我们将使用[SpEL表达式](https://www.baeldung.com/spring-expression-language)来调用Article::slug()：

```java
@Externalized("tuyucheng.article.published::#{slug()}")
public record ArticlePublishedEvent(String slug, String title) {
}
```

## 6. 事件发布注册表

如前所述，持久化数据和发布消息之间仍然存在一种容易出错的关系-消息发布失败会导致事务回滚。另一方面，即使消息发布成功，事务稍后仍然可能回滚。

Spring Modulith的事件发布注册表实现了“事务发件箱”模式来解决这个问题，从而确保了整个系统的最终一致性。**当事务操作发生时，事件不会立即向外部系统发送消息，而是存储在同一业务事务中的事件发布日志中**。

### 6.1 事件发布日志

首先，我们需要引入与我们的持久化技术对应的spring-modulith-starter依赖，我们可以查阅[官方文档](https://docs.spring.io/spring-modulith/reference/events.html#starters)，获取支持的Starter的完整列表。由于我们使用[Spring Data JPA](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)和PostgreSQL数据库，因此我们将添加[spring-modulith-starter-jpa](https://mvnrepository.com/artifact/org.springframework.modulith/spring-modulith-starter-jpa)依赖：

```xml
<dependency>
    <groupId>org.springframework.modulith</groupId>
    <artifactId>spring-modulith-starter-jpa</artifactId>
    <version>1.1.2</version>
</dependency>
```

此外，我们将启用Spring Modulith来创建“event_publication”表，该表包含与外部化应用程序事件相关的数据，让我们在application.yml中添加以下属性：

```yaml
spring.modulith:
    events.jdbc-schema-initialization.enabled: true
```

我们的设置使用[Testcontainer](https://www.baeldung.com/spring-boot-testcontainers-integration-test)启动一个包含PostgreSQL数据库的Docker容器，因此，我们可以利用Testcontainers Desktop应用程序“冻结容器关闭”并“打开一个连接到容器本身的终端”。然后，我们可以使用以下命令检查数据库：

- “psql -U test_user -d test_db”：打开PostgreSQL交互式终端
- “\\d”：列出数据库对象

![](/assets/images/2025/kafka/springmodulitheventexternalization01.png)

我们可以看到，“even_publication”表已成功创建，让我们执行一个查询来查看测试持久化的事件：

![](/assets/images/2025/kafka/springmodulitheventexternalization02.png)

在第一行，我们可以看到第一个测试创建的事件，该事件覆盖了正常的流程。然而，在第二个测试中，我们故意创建了一个无效事件，省略了“slug”，以模拟事件发布过程中的失败。由于这篇文章已保存到数据库但未成功发布，因此它在events_publication表中显示为缺少的complete_date。

### 6.2 重新提交事件

**我们可以通过republish-outstanding-events-on-restart属性使Spring Modulith在应用程序重启时自动重新提交事件**：

```yaml
spring.modulith:
    republish-outstanding-events-on-restart: true
```

**此外，我们可以使用IncompleteEventPublications Bean以编程方式重新提交早于给定时间的失败事件**：

```java
@Component
class EventPublications {
    private final IncompleteEventPublications incompleteEvents;
    private final CompletedEventPublications completeEvents;

    // constructor

    void resubmitUnpublishedEvents() {
        incompleteEvents.resubmitIncompletePublicationsOlderThan(Duration.ofSeconds(60));
    }
}
```

类似地，我们可以使用CompletedEventPublications Bean轻松查询或清除event_publications表：

```java
void clearPublishedEvents() {
    completeEvents.deletePublicationsOlderThan(Duration.ofSeconds(60));
}
```

## 7. 事件外部化配置

尽管@Externalized注解的值对于简洁的SpEL表达式很有用，但在某些情况下我们可能希望避免使用它：

- 如果表达式变得过于复杂
- 当我们试图将主题信息与应用事件分离时
- 如果我们想要为应用程序事件和外部化事件建立不同的模型

对于这些用例，我们可以**使用EventExternalizationConfiguration的构建器配置必要的路由和事件映射**。之后，我们只需将此配置公开为[Spring Bean](https://www.baeldung.com/spring-bean)即可：

```java
@Bean
EventExternalizationConfiguration eventExternalizationConfiguration() {
    return EventExternalizationConfiguration.externalizing()
            .select(EventExternalizationConfiguration.annotatedAsExternalized())
            .route(
                    ArticlePublishedEvent.class,
                    it -> RoutingTarget.forTarget("tuyucheng.articles.published").andKey(it.slug())
            )
            .mapping(
                    ArticlePublishedEvent.class,
                    it -> new PostPublishedKafkaEvent(it.slug(), it.title())
            )
            .build();
}
```

EventExternalizationConfiguration使我们能够以声明的方式定义应用程序事件的路由和映射，此外，**它还允许我们处理各种类型的应用程序事件**。例如，如果我们需要处理像“WeeklySummaryPublishedEvent”这样的附加事件，我们可以通过添加一个特定类型的routing和mapping来轻松实现：

```java
@Bean
EventExternalizationConfiguration eventExternalizationConfiguration() {
    return EventExternalizationConfiguration.externalizing()
            .select(EventExternalizationConfiguration.annotatedAsExternalized())
            .route(
                    ArticlePublishedEvent.class,
                    it -> RoutingTarget.forTarget("tuyucheng.articles.published").andKey(it.slug())
            )
            .mapping(
                    ArticlePublishedEvent.class,
                    it -> new PostPublishedKafkaEvent(it.slug(), it.title())
            )
            .route(
                    WeeklySummaryPublishedEvent.class,
                    it -> RoutingTarget.forTarget("tuyucheng.articles.published").andKey(it.handle())
            )
            .mapping(
                    WeeklySummaryPublishedEvent.class,
                    it -> new PostPublishedKafkaEvent(it.handle(), it.heading())
            )
            .build();
}
```

正如我们所观察到的，mapping和routing需要两样东西：类型本身以及一个用于解析Kafka主题和负载的函数。在我们的示例中，两个应用程序事件都将映射到一个通用类型并发送到同一个主题。

此外，由于我们现在在配置中声明了路由，因此我们可以从事件本身中删除此信息，这样一来，该事件将只带有@Externalized注解，而没有任何值：

```java
@Externalized
public record ArticlePublishedEvent(String slug, String title) {
}

@Externalized
public record WeeklySummaryPublishedEvent(String handle, String heading) {
}
```

## 8. 总结

在本文中，我们讨论了需要在事务块内发布消息的场景，我们发现这种模式可能会对性能产生很大的影响，因为它会阻塞数据库连接更长时间。

之后，我们使用Spring Modulith的功能来监听Spring应用程序事件，并自动将其发布到Kafka主题。这种方法使我们能够异步地外部化事件，并更快地释放数据库连接。