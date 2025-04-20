---
layout: post
title:  垂直切片架构
category: ddd
copyright: ddd
excerpt: DDD
---

## 1. 概述

在本教程中，我们将学习垂直切片架构(VSL)及其如何解决分层设计相关的问题。我们将讨论如何根据业务功能构建代码，从而将代码库组织成松散耦合且内聚的模块，使其更具表现力。之后，我们将从领域驱动设计(DDD)的角度探索这种方法，并讨论其灵活性。

## 2. 分层架构

在探索垂直切片架构之前，我们先来回顾一下其主要对应物-分层架构的主要特性。分层设计非常流行且应用广泛，其变体包括[六边形架构](https://www.baeldung.com/hexagonal-architecture-ddd-spring)、洋葱架构、端口和适配器架构以及[清洁架构](https://www.baeldung.com/spring-boot-clean-architecture)。

**分层架构使用一系列堆叠或同心的层来保护域逻辑免受外部组件和因素的影响**，这些架构的一个关键特征是所有依赖关系都指向内部，指向域：

![](/assets/images/2025/ddd/javaverticalslicearchitecture01.png)

### 2.1 按技术关注点对组件进行分组

分层方法仅关注根据技术问题对组件进行分组，而不是根据其业务能力。

对于本文的代码示例，假设我们正在构建一个博客网站的后端应用程序，该应用程序将支持以下用例：

- 作者可以发布和编辑文章
- 作者可以看到包含其文章统计数据的仪表板
- 读者可以阅读、点赞和评论文章
- 读者会收到文章推荐通知

例如，**我们的包名称反映了技术层，但没有传达项目的真正目的**：

![](/assets/images/2025/ddd/javaverticalslicearchitecture02.png)

### 2.2 高耦合

此外，**将相同的领域服务复用到不相关的业务用例中可能会导致紧耦合**。例如，ArticleService目前依赖于：

- ArticleRepository：查询数据库
- UserService：获取文章作者的数据
- RecommendationService：在新文章发布时更新读者推荐
- CommentService：管理文章评论

因此，当我们添加或修改用例时，可能会干扰不相关的流程。此外，这种高度耦合的方法常常会导致充满Mock的混乱测试。

### 2.3 低内聚

最后，这种代码结构往往会导致组件内部的内聚性较低。**由于业务用例的代码分散在项目的各个包中，任何细微的改动都需要我们修改各个层的文件**。

让我们在Article实体中添加一个slug字段，如果我们想允许客户端使用这个新字段来查询数据库，我们需要修改各个层级中的许多文件：

![](/assets/images/2025/ddd/javaverticalslicearchitecture03.png)

即使是一个简单的修改，也会导致应用程序的几乎每个包都发生变化。这些一起变化的类却无法共存，这表明内聚力较低。

## 3. 垂直切片架构

垂直切片架构旨在通过按业务功能组织代码来解决分层架构的一些问题。**遵循这种方法，我们的组件可以反映业务用例并跨越多个层级**。

因此，所有控制器不会被分组到一个公共包中，而是会被移动到与各自切片关联的包中：

![](/assets/images/2025/ddd/javaverticalslicearchitecture04.png)

此外，**我们可以将相关的用例分组，使其与业务领域保持一致**。让我们根据作者、读者和推荐领域重新组织一下示例：

![](/assets/images/2025/ddd/javaverticalslicearchitecture05.png)

将项目划分为垂直切片使我们能够对大多数类使用默认的包私有访问修饰符，**这确保了意外的依赖关系不会跨越域边界**。

最后，它使不熟悉代码库的人也能通过查看文件结构来了解应用程序的功能。**《代码整洁之道》的作者Robert C.Martin将此称为“[尖叫架构](https://blog.cleancoder.com/uncle-bob/2011/09/30/Screaming-Architecture.html)”**：他认为，软件项目的设计应该清晰地传达其目的，就像建筑物的建筑蓝图能够揭示其功能一样。

## 4. 耦合与内聚

如前所述，选择垂直切片架构而不是洋葱架构可以改善耦合和内聚的管理。

### 4.1 通过应用程序事件实现松散耦合

我们不必消除切片之间的耦合，而应该专注于定义跨边界通信的正确接口。**使用应用程序事件是一种强大的技术，它使我们能够在促进跨边界交互的同时保持松散耦合**。

在分层架构方法中，不相关的服务相互依赖以完成业务功能。具体来说，ArticleService依赖RecommendationService来通知它有新文章。相反，推荐流程可以异步执行，并通过监听应用程序事件来响应主流程。

由于我们在代码示例中使用了Spring框架，因此当创建新文章时我们会发布一个[Spring事件](https://www.baeldung.com/spring-events)：

```java
@Component
class CreateArticleUseCase {

    private final ApplicationEventPublisher eventPublisher;

    // constructor

    void createArticle(CreateArticleRequest article) {
        saveToDatabase(article);

        var event = new ArticleCreatedEvent(article.slug(), article.name(), article.category());
        eventPublisher.publishEvent(event);
    }

    private void saveToDatabase(CreateArticleRequest aticle) { /* ... */ }
    // ...
}
```

现在，SendArticleRecommendationUseCase可以使用@EventListener对ArticleCreatedEvent做出反应并执行其逻辑：

```java
@Component
class SendArticleRecommendationUseCase {

    @EventListener
    void onArticleRecommendation(ArticleCreatedEvent article) {
        findTopicFollowers(article.name(), article.category())
            .forEach(follower -> sendArticleViaEmail(article.slug(), article.name(), follower));
    }

    private void sendArticleViaEmail(String slug, String name, TopicFollower follower) {
        // ...
    }

    private List<TopicFollower> findTopicFollowers(String articleName, String topic) {
        // ...
    }

    record TopicFollower(Long userId, String email, String name) {}
}
```

可以看出，各个模块独立运行，彼此之间没有直接依赖关系。此外，任何对新创建的文章感兴趣的组件只需监听ArticleCreatedEvent即可。

### 4.2 高内聚

找到正确的边界可以构建具有凝聚力的切片和用例，**用例类通常应该只有一个公共方法和一个变更原因，以遵循[单一职责原则](https://www.baeldung.com/java-single-responsibility-principle)**。

让我们在垂直切片架构中为Article类添加一个slug字段，并创建一个通过slug查找文章的端点。这次，修改范围限定在一个包内。我们将创建一个SearchArticleUseCase，它使用JdbcClient查询数据库并返回Article的投影。因此，我们只需修改一个包中的两个文件：

![](/assets/images/2025/ddd/javaverticalslicearchitecture06.png)

我们创建了一个用例，并修改了ReaderController以暴露新的端点。这两个文件位于同一个包中，这表明项目内部的内聚性更高。

## 5. 设计灵活性

垂直切片架构允许我们为每个组件定制方法，**并确定为每个用例组织代码的最有效方法**。换句话说，我们可以使用各种工具、模式或范例，而无需在整个应用程序中强制执行特定的编码风格或依赖关系。

此外，这种灵活性有利于[领域驱动设计(DDD)](https://www.baeldung.com/java-modules-ddd-bounded-contexts)和[CQRS](https://www.baeldung.com/cqrs-event-sourcing-java#2-cqrs)等方法。虽然不是强制性的，但它们非常适合垂直切片应用程序。

### 5.1 使用DDD建模领域

领域驱动设计是一种强调基于核心业务领域及其逻辑进行软件建模的方法，在领域驱动设计(DDD)中，代码必须使用业务人员和客户熟悉的术语和语言，以确保技术和业务视角的一致性。

在垂直切片架构中，我们可能会遇到用例之间代码重复的问题。对于扩展的切片，我们可以决定提取通用业务规则，并使用DDD创建特定于它们的领域模型：

![](/assets/images/2025/ddd/javaverticalslicearchitecture07.png)

此外，**DDD使用有界上下文来定义特定的边界，确保系统不同部分之间的明确区分**。

让我们回顾一个遵循分层方法的项目，我们会注意到，我们通过UserService、UserRepository和User实体等对象与系统用户进行交互。相比之下，在垂直切片项目中，用户的概念在不同的限界上下文中有所不同。**每个切片都有其自己的用户表示形式，将其称为“读者”、“作者”或“主题关注者”**，以反映他们在该上下文中扮演的特定角色。

### 5.2 绕过域的简单用例

严格遵循分层架构的另一个缺点是，它可能导致方法只是简单地将调用传递给下一层，而不会增加任何价值。这也被称为“中间人”反模式，它会导致层与层之间紧密耦合。

例如，当通过slug查找文章时，控制器会调用服务，然后服务会调用Repository。即使在这种情况下服务没有增加任何价值，但分层架构的严格规则阻止我们绕过领域层直接访问持久层。

相比之下，垂直切片应用程序可以灵活地选择每个特定用例所需的层。**这使我们能够绕过简单用例的领域层，并直接查询数据库以获取投影**：

![](/assets/images/2025/ddd/javaverticalslicearchitecture08.png)

让我们简化通过slug查询文章的用例，使用垂直切片架构来绕过领域层：

```java
@Component
class ViewArticleUseCase {

    private static final String FIND_BY_SLUG_SQL = """
            SELECT id, name, slug, content, authorid
            FROM articles
            WHERE slug = ?
            """;

    private final JdbcClient jdbcClient;

    // constructor

    public Optional<ViewArticleProjection> view(String slug) {
        return jdbcClient.sql(FIND_BY_SLUG_SQL)
                .param(slug)
                .query(this::mapArticleProjection)
                .optional();
    }

    record ViewArticleProjection(String name, String slug, String content, Long authorId) {
    }

    private ViewArticleProjection mapArticleProjection(ResultSet rs, int rowNum) throws SQLException {
        // ...
    }
}
```

如我们所见，ViewArticleUseCase直接使用JdbcClient查询数据库。此外，它定义了自己的文章投影，而不是复用通用的DTO，这会导致该用例与其他组件耦合。因此，**不相关的用例不会被强制纳入相同的结构，从而消除了不必要的依赖关系**。

## 6. 总结

在本文中，我们了解了垂直切片架构，并将其与分层架构进行了比较。我们学习了如何创建内聚组件，并避免不相关的业务用例之间的耦合。

我们讨论了有界上下文，以及它们如何帮助我们定义特定于系统各个部分的不同投影。最后，我们发现这种方法在设计每个垂直切片时提供了更高的灵活性。