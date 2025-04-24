---
layout: post
title:  使用Micronaut和MongoDB创建响应式API
category: micronaut
copyright: micronaut
excerpt: Micronaut
---

## 1. 概述

在本教程中，我们将探讨如何使用[Micronaut](https://www.baeldung.com/micronaut)和[MongoDB](https://www.baeldung.com/java-mongodb)创建响应式REST API。

Micronaut是一个在Java虚拟机(JVM)上构建微服务和Serverless应用程序的框架。

我们将研究如何使用Micronaut创建实体、Repository、服务和控制器。

## 2. 项目设置

对于我们的代码示例，我们将创建一个CRUD应用程序，用于从MongoDB数据库存储和检索书籍。首先，让我们使用Micronaut Launch创建一个Maven项目，设置依赖并配置数据库。

### 2.1 初始化项目

让我们首先使用[Micronaut Launch](https://micronaut.io/launch/)创建一个新项目，选择以下设置：

- 应用程序类型：Micronaut应用程序
- Java版本：17
- 构建工具：Maven
- 语言：Java
- 测试框架：JUnit

**此外，我们需要提供Micronaut版本、基础包和项目名称**，为了包含MongoDB和响应式支持，我们将添加以下功能：

- reactor：实现响应式支持
- mongo-reactive：启用MongoDB Reactive Streams支持
- data-mongodb-reactive：启用响应式MongoDB Repository

![](/assets/images/2025/micronaut/mongodbmicronautcreatereactiveapis01.png)

选择上述功能后，我们就可以生成并下载项目，然后，我们可以将项目导入到我们的IDE中。

### 2.2 MongoDB设置

设置MongoDB数据库的方法有很多种，例如，我们可以[在本地安装MongoDB](https://www.baeldung.com/java-redis-mongodb#mongoDBInstallation)，使用MongoDB Atlas等云服务，或者使用[Docker容器](https://www.baeldung.com/java-mongodb-case-insensitive-sorting#overview-1)。

之后，我们需要在已经生成的application.properties文件中配置连接详细信息：
```properties
mongodb.uri=mongodb://${MONGO_HOST:localhost}:${MONGO_PORT:27017}/someDb
```

在这里，我们分别添加了数据库的默认主机和端口为localhost和27017。

## 3. 实体

现在我们已经设置好了项目，让我们看看如何创建实体，我们将创建一个Book实体，它映射到数据库中的集合：
```java
@Serdeable
@MappedEntity
public class Book {
    @Id
    @Generated
    @Nullable
    private ObjectId id;
    private String title;
    private Author author;
    private int year;
}
```

**@Serdeable注解表示该类可以序列化和反序列化**，由于我们将在请求和响应中传递此实体，因此需要将其设置为可序列化，这与实现Serializable接口相同。

**要将类映射到数据库集合，我们使用@MappedEntity注解**。在写入或读取数据库时，Micronaut使用此类将数据库文档转换为Java对象，反之亦然。这与[Spring Data MongoDB](https://www.baeldung.com/spring-data-mongodb-tutorial#annotations)中的[@Document](https://www.baeldung.com/spring-data-mongodb-tutorial#annotations)注解类似。

 我们用@Id标注id字段，以表明它是实体的主键。此外，我们用@Generated标注它，以表明数据库生成该值。**@Nullable注解用于指示该字段可以为空，因为创建实体时id字段将为空**。

类似地，让我们创建一个Author实体：
```java
@Serdeable
public class Author {
    private String firstName;
    private String lastName;
}
```

我们不需要用@MappedEntity标注此类，因为它将嵌入到Book实体中。

## 4. Repository

接下来，让我们创建一个Repository来存储和检索MongoDB数据库中的书籍，Micronaut提供了几个预定义的接口来创建Repository。

我们将使用ReactorCrudRepository接口来创建一个响应式Repository，**此接口扩展了CrudRepository接口并添加了对响应式流的支持**。

此外，我们将使用@MongoRepository标注Repository，以表明它是一个MongoDB Repository，这还会指示Micronaut为此类创建一个Bean：
```java
@MongoRepository
public interface BookRepository extends ReactorCrudRepository<Book, ObjectId> {
    @MongoFindQuery("{year: {$gt: :year}}")
    Flux<Book> findByYearGreaterThan(int year);
}
```

我们扩展了ReactorCrudRepository接口并提供了Book实体和ID类型作为泛型参数。

**Micronaut在编译时生成接口的实现，它包含从数据库中保存、检索和删除书籍的方法**。我们添加了一个自定义方法来查找给定年份后出版的书籍，**@MongoFindQuery注解用于指定自定义查询**。

在我们的查询中，我们使用:year占位符来表示该值将在运行时提供，\$gt运算符类似于SQL中的“\>”运算符。

## 5. 服务

服务用于封装业务逻辑，通常注入到控制器中。此外，它们可能包含其他功能，例如验证、错误处理和日志记录。

我们将使用BookRepository创建一个BookService来存储和检索书籍：
```java
@Singleton
public class BookService {
    private final BookRepository bookRepository;

    public BookService(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }

    public ObjectId save(Book book) {
        Book savedBook = bookRepository.save(book).block();
        return null != savedBook ? savedBook.getId() : null;
    }

    public Book findById(String id) {
        return bookRepository.findById(new ObjectId(id)).block();
    }
    
    public ObjectId update(Book book) {
        Book updatedBook = bookRepository.update(book).block();
        return null != updatedBook ? updatedBook.getId() : null;
    }

    public Long deleteById(String id) {
        return bookRepository.deleteById(new ObjectId(id)).block();
    }

    
    public Flux<Book> findByYearGreaterThan(int year) {
        return bookRepository.findByYearGreaterThan(year);
    }
}
```

这里我们使用构造函数注入将BookRepository注入到构造函数中，**@Singleton注解表示只会创建该服务的一个实例**，这类似于Spring Boot的[@Component](https://www.baeldung.com/spring-component-repository-service#component)注解。

接下来，我们使用save()、findById()、update()和deleteById()方法来从数据库中保存、查找、更新和删除书籍。**block()方法会阻塞执行，直到结果可用**。

最后，我们有一个findByYearGreaterThan()方法来查找给定年份之后出版的书籍。

## 6. 控制器

控制器用于处理传入的请求并返回响应，在Micronaut中，我们可以使用注解来创建控制器并根据不同的路径和HTTP方法配置路由。

### 6.1 BookController

我们将创建一个BookController来处理与书籍相关的请求：
```java
@Controller("/books")
public class BookController {

    private final BookService bookService;

    public BookController(BookService bookService) {
        this.bookService = bookService;
    }

    @Post
    public String createBook(@Body Book book) {
        @Nullable ObjectId bookId = bookService.save(book);
        if (null == bookId) {
            return "Book not created";
        } else {
            return "Book created with id: " + bookId.getId();
        }
    }

    @Put
    public String updateBook(@Body Book book) {
        @Nullable ObjectId bookId = bookService.update(book);
        if (null == bookId) {
            return "Book not updated";
        } else {
            return "Book updated with id: " + bookId.getId();
        }
    }
}
```

我们用@Controller标注了该类以表明它是一个控制器，我们还将控制器的基本路径指定为/books。

让我们看一下控制器的一些重要部分：

- 首先，我们将BookService注入构造函数。
- 然后，我们用createBook()方法来创建一本新书，@Post注解表示该方法处理POST请求。
- **由于我们想要将传入的请求正文转换为Book对象，因此我们使用了@Body注解**。
- **当书籍保存成功时，将返回一个ObjectId**，我们使用了@Nullable注解来指示如果书籍未保存，则该值可以为null。
- 类似地，我们有一个updateBook()方法来更新现有书籍，我们使用@Put注解，因为该方法处理PUT请求。
- 这些方法返回一个字符串响应，表明该书是否已成功创建或更新。

### 6.2 路径变量

要从路径中提取值，我们可以使用路径变量。为了演示这一点，让我们添加一些方法，通过ID来查找和删除书籍：
```java
@Delete("/{id}")
public String deleteBook(String id) {
    Long bookId = bookService.deleteById(id);
    if (0 == bookId) {
        return "Book not deleted";
    } else {
        return "Book deleted with id: " + bookId;
    }
}

@Get("/{id}")
public Book findById(@PathVariable("id") String identifier) {
    return bookService.findById(identifier);
}
```

**路径变量在路径中使用花括号表示**，在此示例中，{id}是一个路径变量，它将从路径中提取并传递给方法。

默认情况下，路径变量的名称应与方法参数的名称匹配，deleteBook()方法就是这种情况。如果不匹配，**我们可以使用@PathVariable注解为路径变量指定不同的名称**，findById()方法就是这种情况。

### 6.3 查询参数

我们可以使用查询参数从查询字符串中提取值，让我们添加一个方法来查找给定年份之后出版的书籍：
```java
@Get("/published-after")
public Flux<Book> findByYearGreaterThan(@QueryValue("year") int year) {
    return bookService.findByYearGreaterThan(year);
}
```

**@QueryValue表示该值将作为查询参数提供，此外，我们需要将查询参数的名称指定为注解的值**。

当我们向此方法发出请求时，我们会在URL后附加一个year参数并提供其值。

## 7. 测试

我们可以使用curl或Postman之类的工具测试该应用程序，让我们使用[curl](https://www.baeldung.com/curl-rest)来测试该应用程序。

### 7.1 创建一本书

让我们使用POST请求创建一本书：
```shell
curl --request POST \
  --url http://localhost:8080/books \
  --header 'Content-Type: application/json' \
  --data '{
    "title": "1984",
    "year": 1949,
    "author": {
        "firstName": "George",
        "lastName": "Orwel"
    }
  }'
```

首先，我们使用-request POST选项来表明该请求是POST请求。然后我们使用-header选项提供标头，在这里，我们将内容类型设置为application/json。最后，我们使用-data选项来指定请求主体。

以下是一个示例响应：
```text
Book created with id: 650e86a7f0f1884234c80e3f
```

### 7.2 查找书籍

接下来我们查找刚刚创建的书：
```shell
curl --request GET \
  --url http://localhost:8080/books/650e86a7f0f1884234c80e3f
```

这将返回ID为650e86a7f0f1884234c80e3f的书籍。

### 7.3 更新书籍

接下来，让我们更新这本书。作者的lastName有一个拼写错误，让我们修复它：
```shell
curl --request PUT \
  --url http://localhost:8080/books \
  --header 'Content-Type: application/json' \
  --data '{
	"id": {
	    "$oid": "650e86a7f0f1884234c80e3f"
	},
	"title": "1984",
	"author": {
	    "firstName": "George",
	    "lastName": "Orwell"
	},
	"year": 1949
}'
```

如果我们再次尝试查找这本书，我们会发现作者的lastName现在是Orwell。

### 7.4 自定义查询

接下来，让我们查找1940年以后出版的所有书籍：
```shell
curl --request GET \
  --url 'http://localhost:8080/books/published-after?year=1940'
```

当我们执行此命令时，它会调用我们的API并以JSON数组的形式返回1940年后出版的所有书籍的列表：
```json
[
    {
        "id": {
            "$oid": "650e86a7f0f1884234c80e3f"
        },
        "title": "1984",
        "author": {
            "firstName": "George",
            "lastName": "Orwell"
        },
        "year": 1949
    }
]
```

类似地，如果我们尝试查找1950年以后出版的所有书籍，我们将得到一个空数组：
```shell
curl --request GET \
  --url 'http://localhost:8080/books/published-after?year=1950'
[]
```

## 8. 错误处理

接下来，我们来看看在应用程序中处理错误的几种方法，我们将讨论两种常见的情况：

- 尝试获取、更新或删除该书时未找到它
- 创建或更新书籍时提供了错误的输入

### 8.1 Bean验证

首先，我们来看看如何处理错误输入，为此，我们可以使用Java的[Bean Validation API](https://www.baeldung.com/java-validation)。

让我们向Book类添加一些约束：
```java
public class Book {
    @NotBlank
    private String title;
    @NotNull
    private Author author;
    // ...
}
```

@NotBlank注解表示title不能为空，同样，我们使用@NotNull注解表示author不能为空。

**然后，为了在我们的控制器中启用输入验证，我们需要使用@Valid注解**：
```java
@Post
public String createBook(@Valid @Body Book book) {
    // ...
}
```

当输入无效时，控制器将返回400 Bad Request响应，其JSON主体包含错误的详细信息：
```json
{
    "_links": {
        "self": [
            {
                "href": "/books",
                "templated": false
            }
        ]
    },
    "_embedded": {
        "errors": [
            {
                "message": "book.author: must not be null"
            },
            {
                "message": "book.title: must not be blank"
            }
        ]
    },
    "message": "Bad Request"
}
```

### 8.2 自定义错误处理程序

在上面的例子中，我们可以看到Micronaut默认如何处理错误。但是，如果我们想改变这种行为，我们可以创建一个自定义错误处理程序。

由于验证错误是ConstraintViolation类的实例，因此让我们创建一个处理ConstraintViolationException的自定义错误处理方法：
```java
@Error(exception = ConstraintViolationException.class)
public MutableHttpResponse<String> onSavedFailed(ConstraintViolationException ex) {
    return HttpResponse.badRequest(ex.getConstraintViolations().stream()
        .map(cv -> cv.getPropertyPath() + " " + cv.getMessage())
        .toList().toString());
}
```

当任何控制器抛出ConstraintViolationException时，Micronaut都会调用此方法，然后它返回400 Bad Request响应，其中包含错误详细信息的JSON主体：
```json
[
    "createBook.book.author must not be null",
    "createBook.book.title must not be blank"
]
```

### 8.3 自定义异常

接下来，我们来看看如何处理找不到书的情况，在这种情况下，我们可以创建一个自定义异常：
```java
public class BookNotFoundException extends RuntimeException {
    public BookNotFoundException(long id) {
        super("Book with id " + id + " not found");
    }
}
```

然后我们可以从控制器抛出这个异常：
```java
@Get("/{id}")
public Book findById(@PathVariable("id") String identifier) throws BookNotFoundException {
    Book book = bookService.findById(identifier);
    if (null == book) {
        throw new BookNotFoundException(identifier);
    } else {
        return book;
    }
}
```

当找不到该书时，控制器将抛出BookNotFoundException。

最后，我们可以创建一个处理BookNotFoundException的自定义错误处理方法：
```java
@Error(exception = BookNotFoundException.class)
public MutableHttpResponse<String> onBookNotFound(BookNotFoundException ex) {
    return HttpResponse.notFound(ex.getMessage());
}
```

当提供不存在的书籍ID时，控制器将返回404 Not Found响应，其JSON主体包含错误的详细信息：
```text
Book with id 650e86a7f0f1884234c80e3f not found
```

## 9. 总结

在本文中，我们研究了如何使用Micronaut和MongoDB创建REST API。首先，我们研究了如何创建MongoDB Repository、简单的控制器以及如何使用路径变量和查询参数。然后，我们使用curl测试了应用程序。最后，我们研究了如何处理控制器中的错误。