---
layout: post
title:  MongoDB和Quarkus入门
category: quarkus
copyright: quarkus
excerpt: Quarkus
---

## 1. 简介

**Quarkus是一个流行的Java框架，针对创建占用内存最少且启动时间较快的应用程序进行了优化**。

与流行的NoSQL数据库[MongoDB](https://www.baeldung.com/java-mongodb)结合使用时，[Quarkus](https://www.baeldung.com/quarkus-io)提供了用于开发高性能、可扩展应用程序的强大工具包。

在本教程中，我们将探索使用Quarkus配置MongoDB，实现基本的[CRUD](https://www.baeldung.com/spring-boot-react-crud)操作，以及如何使用Panache(Quarkus的对象文档映射器(ODM))简化这些操作。

## 2. 配置

### 2.1 Maven依赖

要将MongoDB与Quarkus一起使用，我们需要包含[quarkus-mongodb-client](https://mvnrepository.com/artifact/io.quarkus/quarkus-mongodb-client)依赖：

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-mongodb-client</artifactId>
    <version>3.13.0</version>
</dependency>
```

此依赖提供了使用MongoDB Java客户端与MongoDB数据库交互所需的工具。

### 2.2 运行MongoDB数据库

在本文中，我们将在Docker容器中运行MongoDB。这是一种设置MongoDB实例的便捷方法，无需直接在我们的机器上安装MongoDB。

我们首先从Docker Hub中提取MongoDB镜像：

```shell
docker pull mongo:latest
```

并启动一个新容器：

```shell
docker run --name mongodb -d -p 27017:27017 mongo:latest
```

### 2.3 配置MongoDB数据库

要配置的主要属性是访问MongoDB的URL，我们可以在[连接字符串](https://www.mongodb.com/docs/manual/reference/connection-string/)中包含几乎所有的配置。

我们可以为多个节点的副本集配置MongoDB客户端，但在我们的示例中，我们将在localhost上使用单个实例：

```markdown
quarkus.mongodb.connection-string = mongodb://localhost:27017
```

## 3. 基本CRUD操作

现在我们已经准备好并连接了数据库和项目，让我们使用Quarkus提供的默认Mongo客户端实现基本的CRUD(创建、读取、更新、删除)操作。

### 3.1 定义实体

在本节中，我们将定义Article实体，代表MongoDB集合中的文档：

```java
public class Article {
    @BsonId
    public ObjectId id;
    public String author;
    public String title;
    public String description;
    
    // getters and setters 
}
```

我们的类包含id、author、title和description字段。ObjectId是一种BSON(JSON的二进制表示)类型，用作MongoDB文档的默认标识符。**@BsonId注解将字段指定为MongoDB文档的标识符(_id)**。

当应用于字段时，它表示该字段应映射到MongoDB集合中的_id字段。使用这种组合，我们确保每个Article文档都有一个唯一的标识符，MongoDB可以使用它来高效地索引和检索文档。

### 3.2 定义Repository

在本节中，我们将创建ArticleRepository类，使用MongoClient对Article实体执行CRUD操作。此类将管理与MongoDB数据库的连接，并提供创建、读取、更新和删除Article文档的方法。

首先，我们将使用依赖注入来获取MongoClient的实例：

```java
@Inject
MongoClient mongoClient;
```

这使得我们无需手动管理连接即可与MongoDB数据库交互。

我们定义一个辅助方法getCollection()来从文章数据库中获取文章集合：

```java
private MongoCollection<Article> getCollection() {
    return mongoClient.getDatabase("articles").getCollection("articles", Article.class);
}
```

现在，我们可以使用集合提供程序执行基本的CRUD操作：

```java
public void create(Article article) {
    getCollection().insertOne(article);
}
```

create()方法将新的Article文档插入MongoDB集合，此方法使用insertOne()添加提供的文章对象，确保将其作为新条目存储在文章集合中。

```java
public List<Article> listAll() {
    return getCollection().find().into(new ArrayList<>());
}
```

listAll()方法从MongoDB集合中检索所有Article文档，它利用find()方法查询所有文档并收集它们，我们还可以指定要返回的集合类型。

```java
public void update(Article article) {
    getCollection().replaceOne(new org.bson.Document("_id", article.id), article);
}
```

update()方法用提供的对象替换现有的Article文档，它使用replaceOne()查找具有匹配_id的文档，并使用新数据更新它。

```java
public void delete(String id) {
    getCollection().deleteOne(new org.bson.Document("_id", new ObjectId(id)));
}
```

delete()方法根据ID从集合中删除Article文档，它构造一个过滤器以匹配_id，并使用deleteOne删除与此过滤器匹配的第一个文档。

### 3.3 定义资源

作为一个简单的例子，我们将限制自己定义资源和Repository，而不实现Service层。

现在，我们需要做的就是创建资源，注入Repository，并为每个操作创建一个方法：

```java
@Path("/articles")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public class ArticleResource {
    @Inject
    ArticleRepository articleRepository;

    @POST
    public Response create(Article article) {
        articleRepository.create(article);
        return Response.status(Response.Status.CREATED).build();
    }

    @GET
    public List<Article> listAll() {
        return articleRepository.listAll();
    }

    @PUT
    public Response update(Article updatedArticle) {
        articleRepository.update(updatedArticle);
        return Response.noContent().build();
    }

    @DELETE
    @Path("/{id}")
    public Response delete(@PathParam("id") String id) {
        articleRepository.delete(id);
        return Response.noContent().build();
    }
}
```

### 3.4 测试我们的API

为了确保我们的API正常工作，我们可以使用[curl](https://curl.se/)，这是一个使用各种网络协议传输数据的多功能命令行工具。

我们将向数据库添加一篇新文章，使用HTTP POST方法将代表文章的JSON有效负载发送到/articles端点：

```shell
curl -X POST http://localhost:8080/articles \
-H "Content-Type: application/json" \
-d '{"author":"John Doe","title":"Introduction to Quarkus","description":"A comprehensive guide to the Quarkus framework."}'
```

为了验证我们的文章是否成功存储，我们可以使用HTTP GET方法从数据库中获取所有文章：

```shell
curl -X GET http://localhost:8080/articles
```

通过运行此程序，我们将获得一个包含当前存储在数据库中的所有文章的JSON数组：

```json
[
    {
        "id": "66a8c65e8bd3a01e0a509f0a",
        "author": "John Doe",
        "title": "Introduction to Quarkus",
        "description": "A comprehensive guide to Quarkus framework."
    }
]
```

## 4. 将Panache与MongoDB结合使用

Quarkus提供了一个名为[Panache](https://quarkus.io/guides/mongodb-panache)的额外抽象层，它简化了数据库操作并减少了样板代码。**借助Panache，我们可以将更多精力放在业务逻辑上**，而较少关注数据访问代码，让我们看看如何使用Panache实现相同的CRUD操作。

### 4.1 Maven依赖

要将Panache与MongoDB一起使用，我们需要添加[quarkus-mongodb-panache](https://mvnrepository.com/artifact/io.quarkus/quarkus-mongodb-panache)依赖：

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-mongodb-panache</artifactId>
    <version>3.13.0</version>
</dependency>
```

### 4.2 定义实体

使用Panache时，我们的实体类将扩展PanacheMongoEntity，它为常见的数据库操作提供内置方法。我们还将使用@MongoEntity注解来定义我们的MongoDB集合：

```java
@MongoEntity(collection = "articles", database = "articles")
public class Article extends PanacheMongoEntityBase {
    private ObjectId id;
    private String author;
    private String title;
    private String description;

    // getters and setters
}
```

### 4.3 定义Repository

**使用Panache，我们通过扩展PanacheMongoRepository来创建Repository**。这为我们提供了CRUD操作，而无需编写样板代码：

```java
@ApplicationScoped
public class ArticleRepository implements PanacheMongoRepository<Article> {}
```

**扩展PanacheMongoRepository类时，可以使用几种常用方法来执行CRUD操作和管理MongoDB实体**。现在，我们可以开箱即用persist()、listAll()或findById等方法。

### 4.4 定义资源

现在，我们需要做的就是创建使用新Repository的新资源，而无需所有样板代码：

```java
@Path("/v2/articles")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public class ArticleResource
@Inject
ArticleRepository articleRepository;

@POST
public Response create(Article article) {
    articleRepository.persist(article);
    return Response.status(Response.Status.CREATED).build();
}

@GET
public List<Article> listAll() {
    return articleRepository.listAll();
}

@PUT
public Response update(Article updatedArticle) {
    articleRepository.update(updatedArticle);
    return Response.noContent().build();
}

@DELETE
@Path("/{id}")
public Response delete(@PathParam("id") String id) {
    articleRepository.delete(id);
    return Response.noContent().build();
}
}

```

### 4.5 测试API

我们还可以使用与其他示例中相同的curl命令来测试新的API，唯一要更改的是调用的端点，这次调用/v2/articles API。我们将创建新文章：

```shell
curl -X POST http://localhost:8080/v2/articles \
-H "Content-Type: application/json" \
-d '{"author":"John Doe","title":"Introduction to MongoDB","description":"A comprehensive guide to MongoDB."}'
```

检索现有的文章：

```shell
curl -X GET http://localhost:8080/v2/articles
```

## 5. 总结

在本文中，我们探讨了如何将MongoDB与Quarkus集成。我们配置了MongoDB并在Docker容器中运行它，为我们的应用程序设置了一个强大且可扩展的环境。我们演示了使用默认Mongo客户端实现CRUD操作。

此外，我们引入了Quarkus的对象文档映射器(ODM)Panache，它通过减少样板代码显著简化了数据访问。