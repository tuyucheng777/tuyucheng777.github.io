---
layout: post
title:  使用Spring AI和PGVector实现语义搜索
category: springai
copyright: springai
excerpt: Spring AI
---

## 1. 概述

搜索是软件中的一个基本概念，旨在从大量数据中查找相关信息，它涉及在一组元素中找到特定元素。

在本教程中，我们将探讨如何使用[Spring AI](https://www.baeldung.com/spring-ai)、[PGVector](https://www.baeldung.com/cs/vector-databases)和[Ollama](https://www.baeldung.com/spring-ai-ollama-chatgpt-like-chatbot)实现语义搜索。

## 2. 背景

语义搜索是一种高级搜索技术，它利用单词的含义来查找最相关的结果。要构建语义搜索应用程序，我们需要了解一些关键概念：

- **词嵌入**：词嵌入是一种词语表示，允许具有相似含义的单词具有相似的表示，词嵌入将单词转换为可用于机器学习模型的数字向量。
- **语义相似度**：语义相似度是衡量两段文本在含义上的相似程度的指标，它用于比较单词、句子或文档的含义。
- **向量空间模型**：向量空间模型是一种将文本文档表示为高维空间中的向量的数学模型。在该模型中，每个单词都表示为一个向量，两个单词之间的相似度通过它们向量之间的距离来计算。
- **余弦相似度**：余弦相似度是内积空间中两个非零向量之间的相似度度量，测量它们之间夹角的余弦。它计算向量空间模型中两个向量之间的相似度。

现在让我们构建一个应用程序来演示这一点。

## 3. 先决条件

首先，我们应该在我们的机器上安装[Docker](https://www.baeldung.com/ops/docker-guide)来运行PGVector和Ollama。

然后，我们的Spring应用程序中需要[Spring AI](https://mvnrepository.com/artifact/org.springframework.ai) Ollama和PGVector依赖：

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-ollama-spring-boot-starter</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-pgvector-store-spring-boot-starter</artifactId>
</dependency>
```

我们还将添加[Spring Boot的Docker Compose](https://www.baeldung.com/docker-compose-support-spring-boot)支持来管理Ollama和PGVector Docker容器：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-docker-compose</artifactId>
    <version>3.1.1</version>
</dependency>
```

除了[依赖](https://mvnrepository.com/artifact/org.springframework.ai)之外，我们还将通过在docker-compose.yml文件中描述这两个服务将它们放在一起：

```yaml
services:
    postgres:
        image: pgvector/pgvector:pg17
        environment:
            POSTGRES_DB: vectordb
            POSTGRES_USER: postgres
            POSTGRES_PASSWORD: postgres
        ports:
            - "5434:5432"
        healthcheck:
            test: [ "CMD-SHELL", "pg_isready -U postgres" ]
            interval: 10s
            timeout: 5s
            retries: 5

    ollama:
        image: ollama/ollama:latest
        ports:
            - "11435:11434"
        volumes:
            - ollama_data:/root/.ollama
        healthcheck:
            test: [ "CMD", "curl", "-f", "http://localhost:11435/api/health" ]
            interval: 10s
            timeout: 5s
            retries: 10

volumes:
    ollama_data:
```

## 4. 配置应用程序

接下来，我们需要配置Spring Boot应用程序以使用Ollama和PGVector服务。在application.yml文件中，我们定义了几个属性。让我们特别注意为ollama和vectorstore属性选择的内容：

```yaml
spring:
    ai:
        ollama:
            init:
                pull-model-strategy: when_missing
                chat:
                    include: true
            embedding:
                options:
                    model: nomic-embed-text
        vectorstore:
            pgvector:
                initialize-schema: true
                dimensions: 768
                index-type: hnsw
    docker:
        compose:
            file: docker-compose.yml
            enabled: true
    datasource:
        url: jdbc:postgresql://localhost:5434/vectordb
        username: postgres
        password: postgres
        driver-class-name: org.postgresql.Driver
    jpa:
        database-platform: org.hibernate.dialect.PostgreSQLDialect
```

我们为Ollama模型选择了[nomic-embed-text](https://ollama.com/library/nomic-embed-text)，如果没有下载，Spring AI会帮我们提取它。

PGVector设置通过初始化数据库模式(initialize-schema: true)、将向量维度与常见嵌入大小对齐(dimensions: 768)以及使用分层可导航小世界(HNSW)索引(index-type: hnsw)优化搜索效率来确保正确的向量存储设置，以实现快速近似最近邻搜索。

## 5. 执行语义搜索

现在我们的基础设施已经准备就绪，我们可以实现一个简单的语义搜索应用程序。我们的用例将是一个智能图书搜索引擎，它允许用户根据图书内容搜索图书。

首先，我们将使用PGVector构建一个简单的搜索功能，然后，我们将使用Ollama增强它，以提供更多上下文感知响应。

让我们定义一个代表书籍实体的Book类：

```java
public record Book(String title, String author, String description) {
}
```

在搜索书籍之前，我们需要将书籍数据导入PGVector存储。以下方法添加了一些示例书籍数据：

```java
void run() {
    var books = List.of(
            new Book("The Great Gatsby", "F. Scott Fitzgerald", "The Great Gatsby is a 1925 novel by American writer F. Scott Fitzgerald. Set in the Jazz Age on Long Island, near New York City, the novel depicts first-person narrator Nick Carraway's interactions with mysterious millionaire Jay Gatsby and Gatsby's obsession to reunite with his former lover, Daisy Buchanan."),
            new Book("To Kill a Mockingbird", "Harper Lee", "To Kill a Mockingbird is a novel by the American author Harper Lee. It was published in 1960 and was instantly successful. In the United States, it is widely read in high schools and middle schools."),
            new Book("1984", "George Orwell", "Nineteen Eighty-Four: A Novel, often referred to as 1984, is a dystopian social science fiction novel by the English novelist George Orwell. It was published on 8 June 1949 by Secker & Warburg as Orwell's ninth and final book completed in his lifetime."),
            new Book("The Catcher in the Rye", "J. D. Salinger", "The Catcher in the Rye is a novel by J. D. Salinger, partially published in serial form in 1945–1946 and as a novel in 1951. It was originally intended for adults but is often read by adolescents for its themes of angst, alienation, and as a critique on superficiality in society."),
            new Book("Lord of the Flies", "William Golding", "Lord of the Flies is a 1954 novel by Nobel Prize-winning British author William Golding. The book focuses on a group of British")
    );

    List<Document> documents = books.stream()
            .map(book -> new Document(book.toString()))
            .toList();

    vectorStore.add(documents);
}
```

现在我们已将示例书籍数据添加到PGVector存储中，我们可以实现语义搜索功能。

### 5.1 语义搜索

我们的目标是实现一个语义搜索API，让用户能够根据内容查找书籍。让我们定义一个与PGVector交互以执行相似性搜索的控制器：

```java
@RequestMapping("/books")
class BookSearchController {
    final VectorStore vectorStore;
    final ChatClient chatClient;

    BookSearchController(VectorStore vectorStore, ChatClient.Builder chatClientBuilder) {
        this.vectorStore = vectorStore;
        this.chatClient = chatClientBuilder.build();
    }
    // ...
}
```

接下来，我们将创建一个POST /search端点，接收来自用户的搜索条件并返回匹配的书籍列表：

```java
@PostMapping("/search")
List<String> semanticSearch(@RequestBody String query) {
    return vectorStore.similaritySearch(SearchRequest.builder()
            .query(query)
            .topK(3)
            .build())
            .stream()
        .map(Document::getText)
        .toList();
}
```

请注意，我们使用了VectorStore#similaritySearch，这会对我们之前提取的书籍进行语义搜索。

启动应用程序后，我们就可以执行搜索了。让我们使用[cURL](https://www.baeldung.com/linux/curl-guide)搜索1984年的实例：

```shell
curl -X POST --data "1984" http://localhost:8080/books/search
```

响应包含三本书：一本完全匹配，两本部分匹配：

```json
[
    "Book[title=1984, author=George Orwell, description=Nineteen Eighty-Four: A Novel, often referred to as 1984, is a dystopian social science fiction novel by the English novelist George Orwell.]",
    "Book[title=The Catcher in the Rye, author=J. D. Salinger, description=The Catcher in the Rye is a novel by J. D. Salinger, partially published in serial form in 1945–1946 and as a novel in 1951.]",
    "Book[title=To Kill a Mockingbird, author=Harper Lee, description=To Kill a Mockingbird is a novel by the American author Harper Lee.]"
]
```

### 5.2 使用Ollama增强语义搜索

我们可以整合Ollama来生成释义响应，提供额外的上下文来改善语义搜索结果，具体步骤如下：

1. 从搜索查询中检索最匹配的三本书籍描述
2. 将这些描述输入到Ollama中以生成更自然、更具情境感知的响应
3. 提供包含总结和释义信息的回应，提供更清晰、更相关的见解

让我们在BookSearchController中创建一个新方法，使用Ollama生成查询的释义：

```java
@PostMapping("/enhanced-search")
String enhancedSearch(@RequestBody String query) {
    String context = vectorStore.similaritySearch(SearchRequest.builder()
            .query(query)
            .topK(3)
            .build())
        .stream()
        .map(Document::getText)
        .reduce("", (a, b) -> a + b + "\n");

    return chatClient.prompt()
        .system(context)
        .user(query)
        .call()
        .content();
}
```

现在让我们通过向/books/enhanced-search端点发送POST请求来测试增强语义搜索功能：

```shell
curl -X POST --data "1984" http://localhost:8080/books/enhanced-search

1984 is a classic dystopian novel written by George Orwell. Here's an excerpt from the book:

"He loved Big Brother. He even admired him. After all, who wouldn't? Big Brother was all-powerful, all-knowing, and infinitely charming. And now that he had given up all his money in bank accounts with his names on them, and his credit cards, and his deposit slips, he felt free."

This excerpt sets the tone for the novel, which depicts a totalitarian society where the government exercises total control over its citizens. The protagonist, Winston Smith, is a low-ranking member of the ruling Party who begins to question the morality of their regime.

Would you like to know more about the book or its themes?
```

Ollama不会像简单的语义搜索那样返回三个单独的图书描述，而是会综合搜索结果中最相关的信息。在本例中，1984是最相关的匹配项，因此Ollama专注于提供详细的摘要，而不是列出不相关的图书。这模仿了类似人类的搜索帮助，使结果更具吸引力和洞察力。

## 6. 总结

在本文中，我们探讨了如何使用Spring AI、PGVector和Ollama实现语义搜索。我们比较了两个端点；一个端点对我们的图书目录执行语义搜索，另一个端点使用Ollama LLM提供并增强该搜索结果。