---
layout: post
title:  将Java Spring Boot连接到Db2数据库
category: springdata
copyright: springdata
excerpt: Spring Boot
---

## 1. 概述

**IBM Db2是一种云原生关系型数据库系统，也支持JSON和XML等半结构化数据**，它旨在以低延迟处理大量数据。

在本教程中，我们将使用[docker-compose.yml](https://www.baeldung.com/ops/docker-compose)文件在[Docker容器](https://www.baeldung.com/ops/docker-install-windows-linux-mac)内设置Db2数据库服务器的社区版。然后，我们将在Spring Boot应用程序中使用其JDBC驱动程序连接到它。

## 2. Db2数据库

Db2数据库以高效处理海量数据和高吞吐量而闻名，它可以处理结构化和半结构化数据，从而能够灵活地适应不同的数据模型。它既适用于微服务应用程序，也适用于单体应用。

此外，该数据库系统需要订阅才能访问其全部功能。**但是，可以使用免费的社区版，但其最大存储容量限制为16GiB和4个CPU核心**，此版本非常适合测试用途或资源需求有限的小型应用程序。

## 3. 设置

首先，让我们为Spring Boot应用程序设置依赖并在Docker容器内配置Db2数据库服务器。

### 3.1 Maven依赖

让我们通过向pom.xml添加[spring-boot-starter-web](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web)和[spring-boot-starter-data-jpa](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-jpa)依赖来引导Spring Boot应用程序：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>3.4.3</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <version>3.4.3</version>
</dependency>
```

spring-boot-starter-web依赖允许我们创建一个Web应用程序，包括RESTful API和MVC应用程序。

spring-boot-starter-data-jpa提供了一个[对象关系映射(ORM)](https://www.baeldung.com/cs/object-relational-mapping)工具，通过抽象原生查询来简化数据库交互，除非明确定义。

另外，让我们将[com.ibm.db2](https://mvnrepository.com/artifact/com.ibm.db2/jcc)依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>com.ibm.db2</groupId>
    <artifactId>jcc</artifactId>
    <version>12.1.0.0</version>
</dependency>
```

com.ibm.db2依赖提供了Db2 JDBC驱动程序，使我们能够建立与Db2数据库服务器的连接。

### 3.2 Db2 Docker Compose文件

此外，让我们在项目根目录中定义一个docker-compose.yml文件，以在Docker容器中运行数据库服务器：

```yaml
services:
    db2:
        image: icr.io/db2_community/db2
        container_name: db2server
        hostname: db2server
        privileged: true
        restart: unless-stopped
        ports:
            - "50000:50000"
        environment:
            LICENSE: accept
            DB2INST1_PASSWORD: mypassword
            DBNAME: testdb
            BLU: "false"
            ENABLE_ORACLE_COMPATIBILITY: "false"
            UPDATEAVAIL: "NO"
            TO_CREATE_SAMPLEDB: "false"
        volumes:
            - db2_data:/database
        healthcheck:
            test: ["CMD", "su", "-", "db2inst1", "-c", "db2 connect to testdb || exit 1"]
            interval: 30s
            retries: 5
            start_period: 60s
            timeout: 10s

volumes:
    db2_data:
        driver: local
```

上述配置会拉取最新的Db2社区镜像，此外，它还会创建用户、设置密码并初始化数据库。在本例中，用户名为db2inst1，密码为mypassword，数据库名称为testdb。

接下来，我们运行docker compose up命令来启动数据库服务器。

## 4. 在application.properties中定义连接凭据

现在我们的数据库服务器已经启动，让我们在application.properties文件中定义连接详细信息：

```properties
spring.datasource.url=jdbc:db2://localhost:50000/testdb
spring.datasource.username=db2inst1
spring.datasource.password=mypassword
spring.datasource.driver-class-name=com.ibm.db2.jcc.DB2Driver
spring.jpa.database-platform=org.hibernate.dialect.DB2Dialect
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

在这里，我们定义数据库URL，其中包括主机、端口和要连接的数据库的名称。此外，**URL以jdbc:db2开头，这是一个JDBC子协议，表示连接到的是Db2数据库**。

接下来，我们按照docker-compose.yml文件中的配置指定用户名和密码。

重要的是，设置spring.datasource.driver-class-name属性可确保Spring Boot加载正确的Db2 JDBC驱动程序。

## 5. 测试连接

接下来，让我们通过将记录持久保存到testdb来确认与数据库的连接。

### 5.1 实体类

首先，让我们创建一个名为Article的实体类：

```java
@Entity
public class Article {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String title;
    private String body;
    private String author;

    // standard constructor, getters and setters
}
```

实体类映射到数据库表及其列。

接下来，让我们定义一个Repository来处理数据库操作：

```java
public interface ArticleRepository extends JpaRepository<Article, Long> {
}
```

现在，我们可以通过将Repository注入到控制器中来将实体持久保存到数据库中。

### 5.2 控制器类

此外，让我们创建一个控制器类：

```java
@RestController
public class ArticleController {
    private final ArticleRepository articleRepository ;

    public ArticleController(ArticleRepository articleRepository) {
        this.articleRepository = articleRepository;
    }
}
```

在上面的代码中，我们使用[构造函数注入](https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring#constructor-based-dependency-injection)将Repository注入到控制器类中，这使我们能够执行各种数据库操作，例如保存和检索文章。

然后，让我们实现一个POST端点来插入记录：

```java
@PostMapping("/create-article")
private ResponseEntity<Article> createArticle(@RequestBody Article article, UriComponentsBuilder ucb) {
    Article newArticle = new Article();
    newArticle.setAuthor(article.getAuthor());
    newArticle.setBody(article.getBody());
    newArticle.setTitle(article.getTitle());
    Article savedArticle = articleRepository.save(newArticle);
    URI location = ucb.path("/articles/{id}").buildAndExpand(savedArticle.getId()).toUri();

    return ResponseEntity.created(location).body(savedArticle);
}
```

在上面的代码中，我们创建了一个端点，它接收Article对象作为请求主体并将其保存到数据库。此外，我们返回HTTP 201(Created)状态码。

### 5.3 集成测试

最后，让我们使用[RestTemplate](https://www.baeldung.com/rest-template)类编写一个[集成测试](https://www.baeldung.com/spring-boot-testing#integration-testing-with-springboottest)来验证POST请求的行为。

首先，让我们创建一个集成测试类：

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class ArticleApplicationIntegrationTest {
}
```

接下来，我们注入RestTemplate实例：

```java
@Autowired
TestRestTemplate restTemplate;
```

然后，我们编写一个测试方法来发送POST请求并验证响应：

```java
@Test
void givenNewArticleObject_whenMakingAPostRequest_thenReturnCreated() {
    Article article = new Article();
    article.setTitle("Introduction to Java");
    article.setAuthor("Tuyucheng");
    article.setBody("Java is a programming language created by James Gosling");
    ResponseEntity<Article> createResponse = restTemplate.postForEntity("/create-article", article, Article.class);
    assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.CREATED);

    URI locationOfNewArticle = createResponse.getHeaders().getLocation();
    ResponseEntity<String> getResponse = restTemplate.getForEntity(locationOfNewArticle, String.class);
    assertThat(getResponse.getStatusCode()).isEqualTo(HttpStatus.OK);
}
```

在上面的代码中，我们断言记录已成功创建，并且通过id检索它会返回HTTP 200响应。

## 6. 总结

在本文中，我们学习了如何设置IBM Db2数据库并使用在Spring Boot应用程序的application.properties文件中配置的Db2驱动程序连接到该数据库。此外，我们还通过将记录持久保存到数据库来测试连接。