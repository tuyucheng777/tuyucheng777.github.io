---
layout: post
title:  实现不返回数据的GraphQL修改
category: graphql
copyright: graphql
excerpt: GraphQL
---

## 1. 简介

**[GraphQL](https://www.baeldung.com/graphql)是一种强大的API查询语言，它提供了一种灵活高效的数据交互方式**。处理变更时，通常会在服务器上执行数据的更新或添加。然而，在某些情况下，我们可能需要在不返回任何数据的情况下进行变更。

在GraphQL中，默认行为是强制模式中的字段不可为空，这意味着字段必须始终返回值，并且除非明确标记为可空，否则不能为空。虽然这种严格性有助于提高API的清晰度和可预测性，但在某些情况下可能需要返回null。不过，通常认为避免返回null值是最佳做法。

在本文中，我们将探讨无需检索或返回特定信息即可实现GraphQL变异的技术。

## 2. 先决条件

对于我们的示例，我们需要以下依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-graphql</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

**[Spring Boot GraphQL Starter](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-graphql)提供了快速搭建GraphQL服务器的绝佳解决方案**，通过利用自动配置并采用基于注解的编程方法，我们只需专注于编写服务所需的基本代码。

由于GraphQL与传输无关，我们在配置中包含了[Web Starter](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web)，它利用Spring MVC通过HTTP暴露GraphQL API，我们可以通过默认的/graphql端点访问它。我们也可以使用其他Starter，例如[Spring Webflux](https://www.baeldung.com/spring-webflux)，以实现不同的底层实现。

## 3. 使用可空类型

与某些编程语言不同，GraphQL要求对模式中的每个字段进行显式声明，使其可空性。这种方法提高了清晰度，使我们能够明确地告知字段何时可能缺少值。

### 3.1 编写模式

Spring Boot GraphQL Starter会自动将GraphQL Schema文件定位到src/main/resources/graphql/\*\*目录下，它会根据这些文件构建正确的结构，并将特殊的Bean注入到该结构。

我们将首先创建schema.graphqls文件，并为我们的示例定义模式：

```graphql
type Post {
    id: ID
    title: String
    text: String
    category: String
    author: String
}

type Mutation {
    createPostReturnNullableType(title: String!, text: String!, category: String!, authorId: String!) : Int
}
```

我们将创建一个Post实体和一个用于创建新帖子的变更，此外，为了使我们的模式能够通过验证，它必须包含一个查询。因此，我们将实现一个返回帖子列表的虚拟查询：

```graphql
type Query {
    recentPosts(count: Int, offset: Int): [Post]!
}
```

### 3.2 使用Bean表示类型

**在GraphQL服务器中，每个复杂类型都与一个Java Bean关联**，这些关联是基于对象和属性名称建立的。因此，我们将为我们的帖子创建一个POJO类：

```java
public class Post {
    private String id;
    private String title;
    private String text;
    private String category;
    private String author;

    // getters, setters, constructor
}
```

GraphQL模式中忽略了Java Bean上未映射的字段或方法，因此不会造成任何问题。

### 3.3 创建变异解析器

**我们必须使用@MutationMapping注解标记处理函数**，这些方法应该放在我们应用程序中的常规@Controller组件中，并在GraphQL应用程序中将这些类注册为数据修改组件：

```java
@Controller
public class PostController {

    List<Post> posts = new ArrayList<>();

    @MutationMapping
    public Integer createPost(@Argument String title, @Argument String text, @Argument String category, @Argument String author) {
        Post post = new Post();
        post.setId(UUID.randomUUID().toString());
        post.setTitle(title);
        post.setText(text);
        post.setCategory(category);
        post.setAuthor(author);
        posts.add(post);
        return null;
    }
}
```

**我们必须根据模式中的属性，使用@Argument注解方法的参数**。在声明模式时，我们确定突变将返回Int类型，不带感叹号。这样，返回值就可以为null。

## 4. 创建自定义标量

在GraphQL中，标量是表示GraphQL查询或模式中的叶节点的原子数据类型。

### 4.1 标量和扩展标量

根据[GraphQL规范](https://spec.graphql.org/draft/#sec-Scalars)，所有实现都必须包含以下标量类型：String、Boolean、Int、Float或ID。此外，[graphql-java-extended-scalars](https://github.com/graphql-java/graphql-java-extended-scalars)还添加了更多自定义标量，例如Long、BigDecimal或LocalDate。**但是，原始标量集和扩展标量集都没有针对空值的特殊标量，因此，我们将在本节中构建我们的标量**。

### 4.2 创建自定义标量

**要创建自定义标量，我们应该初始化一个GraphQLScalarType单例实例**，我们将利用[构建器设计模式](https://www.baeldung.com/creational-design-patterns#builder)来创建标量：

```java
public class GraphQLVoidScalar {

    public static final GraphQLScalarType Void = GraphQLScalarType.newScalar()
            .name("Void")
            .description("A custom scalar that represents the null value")
            .coercing(new Coercing() {
                @Override
                public Object serialize(Object dataFetcherResult) {
                    return null;
                }

                @Override
                public Object parseValue(Object input) {
                    return null;
                }

                @Override
                public Object parseLiteral(Object input) {
                    return null;
                }
            })
            .build();
}
```

**标量的关键组件是名称、描述和强制转换**。虽然名称和描述不言自明，但创建自定义标量的难点在于graphql.schema.Coercing的实现，该类负责三个功能：

- parseValue()：接收变量输入对象并将其转换为相应的Java运行时表示。
- parseLiteral()：接收AST文字graphql.language.Value作为输入，并将其转换为Java运行时表示。
- serialize()：接收Java对象并将其转换为该标量的输出形状。

尽管对于复杂对象来说强制的实现可能非常复杂，但在我们的例子中，我们将为每种方法返回null。

### 4.3 注册自定义标量

我们首先创建一个配置类来注册我们的标量：

```java
@Configuration
public class GraphQlConfig {
    @Bean
    public RuntimeWiringConfigurer runtimeWiringConfigurer() {
        return wiringBuilder -> wiringBuilder.scalar(GraphQLVoidScalar.Void);
    }	
}
```

我们创建一个[RuntimeWiringConfigurer](https://docs.spring.io/spring-graphql/docs/current/api/org/springframework/graphql/execution/RuntimeWiringConfigurer.html) Bean，用于配置GraphQL模式的运行时连接。**在这个Bean中，我们使用RuntimeWiring类提供的scalar()方法来注册我们的自定义类型**。

### 4.4 集成自定义标量

**最后一步是将自定义标量集成到我们的GraphQL模式中，方法是使用定义的名称引用它**。在本例中，我们只需在模式中声明标量Void即可使用它。

此步骤确保GraphQL引擎能够在整个模式中识别并使用我们的自定义标量，现在，我们可以将标量集成到我们的变更中：

```graphql
scalar Void

type Mutation {
    createPostReturnCustomScalar(title: String!, text: String!, category: String!, authorId: String!) : Void
}
```

此外，我们将更新映射的方法签名以返回我们的标量：

```java
public Void createPostReturnCustomScalar(@Argument String title, @Argument String text, @Argument String category, @Argument String author)
```

## 5. 总结

在本文中，我们探索了如何在不返回特定数据的情况下实现GraphQL突变。我们演示了如何使用Spring Boot GraphQL Starter快速设置服务器，此外，我们还引入了一个自定义Void标量来处理空值，展示了如何扩展GraphQL的功能。