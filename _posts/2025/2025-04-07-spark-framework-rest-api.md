---
layout: post
title:  使用Spark Java框架构建API
category: webmodules
copyright: webmodules
excerpt: Spark
---

## 1. 简介

在本文中，我们将简要介绍[Spark框架](http://sparkjava.com/)。Spark框架是一个快速开发的Web框架，灵感来自Ruby的Sinatra框架，并围绕Java 8 Lambda表达式哲学构建，因此比用其他Java框架编写的大多数应用程序更简洁。

如果你想在使用Java开发Web API或微服务时获得类似Node.js的体验，那么Spark是一个不错的选择。借助Spark，你只需不到十行代码即可准备好REST API来提供JSON服务。

我们将通过“Hello World”示例快速入门，然后是简单的REST API。

## 2. Maven依赖

### 2.1 Spark框架

在你的pom.xml中包含以下Maven依赖：

```xml
<dependency>
    <groupId>com.sparkjava</groupId>
    <artifactId>spark-core</artifactId>
    <version>2.5.4</version>
</dependency>
```

你可以在[Maven Central](https://mvnrepository.com/artifact/com.sparkjava/spark-core)上找到最新版本的Spark。

### 2.2 Gson库

在示例的各个地方，我们将使用Gson库进行JSON操作。要将Gson包含在你的项目中，请在你的pom.xml中包含此依赖：

```xml
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.10.1</version>
</dependency>
```

你可以在[Maven Central](https://mvnrepository.com/artifact/com.google.code.gson/gson)上找到最新版本的Gson。

## 3. Spark框架入门

让我们看一下Spark应用程序的基本构建块并演示一个快速的Web服务。

### 3.1 路由

Spark Java中的Web服务基于路由及其处理程序构建，路由是Spark中的基本元素，根据[文档](http://sparkjava.com/documentation.html#routes)，每个路由由3个简单部分组成-动词、路径和回调。

1. 动词是与HTTP方法对应的方法，动词方法包括：get、post、put、delete、head、trace、connect和options
2. 路径(也称为路由模式)决定路由应该监听哪些URI并提供响应
3. 回调是一个处理函数，针对给定的动词和路径调用该函数，以便生成并返回对相应HTTP请求的响应，回调以请求对象和响应对象作为参数

这里我们展示了使用get动词的路由的基本结构：

```java
get("/your-route-path/", (request, response) -> {
    // your callback code
});
```

### 3.2 Hello World API

让我们创建一个简单的Web服务，它有两个GET请求路由，并返回“Hello”消息作为响应。这些路由使用get方法，它是从类spark.Spark静态导入的：

```java
import static spark.Spark.*;

public class HelloWorldService {
    public static void main(String[] args) {
        get("/hello", (req, res)->"Hello, world");
        
        get("/hello/:name", (req,res)->{
            return "Hello, "+ req.params(":name");
        });
    }
}
```

get方法的第一个参数是路由的路径，第一个路由包含一个仅表示单个URI(“/hello”)的静态路径。

第二个路由的路径(“/hello/:name”)包含“name”参数的占位符，通过在参数前面加上冒号(“:”) 来表示。此路由将在响应对URI(例如“/hello/Joe”和“/hello/Mary”)的GET请求时调用。

get方法的第二个参数是一个[Lambda表达式](https://www.baeldung.com/java-8-lambda-expressions-tips)，它为该框架提供了函数编程风格。

Lambda表达式具有请求和响应作为参数，并有助于返回响应。我们将把控制器逻辑放在REST API路由的Lambda表达式中，正如我们将在本教程后面看到的那样。

### 3.3 测试Hello World API

将HelloWorldService类作为普通Java类运行后，你将能够使用上面get方法定义的路由在其默认端口4567上访问该服务。

让我们看看第一条路线的请求和响应：

请求：

```text
GET http://localhost:4567/hello
```

响应：

```text
Hello, world
```

让我们测试第二条路线，在其路径中传递name参数：

请求：

```text
GET http://localhost:4567/hello/tuyucheng
```

响应：

```text
Hello, tuyucheng
```

看看URI中文本“tuyucheng”的位置是如何与路由模式“/hello/:name”匹配的-从而导致第二个路由的回调处理函数被调用。

## 4. 设计RESTful服务

在本节中，我们将为以下用户实体设计一个简单的REST Web服务：

```java
public class User {
    private String id;
    private String firstName;
    private String lastName;
    private String email;

    // constructors, getters and setters
}
```

### 4.1 路由

让我们列出组成API的路由：

-GET/users：获取所有用户列表
-GET/users/:id：获取指定id的用户
-POST/users/:id：添加用户
-PUT/users/:id：编辑特定用户
-OPTIONS/users/:id：检查给定id的用户是否存在
-DELETE/users/:id：删除特定用户

### 4.2 用户服务

下面是UserService接口，声明了User实体的CRUD操作：

```java
public interface UserService {

    public void addUser (User user);

    public Collection<User> getUsers ();
    
    public User getUser (String id);

    public User editUser (User user) throws UserException;

    public void deleteUser (String id);

    public boolean userExist (String id);
}
```

为了演示目的，我们在GitHub代码中提供了此UserService接口的Map实现来模拟持久层，**你可以使用自己选择的数据库和持久层提供自己的实现**。

### 4.3 JSON响应结构

以下是我们的REST服务中使用的响应的JSON结构：

```json
{
    status: <STATUS>
    message: <TEXT-MESSAGE>
    data: <JSON-OBJECT>
}
```

status字段值可以是SUCCESS或ERROR，data字段将包含返回数据的JSON表示形式，例如User或Users集合。

当没有返回数据或状态为ERROR时，我们将填充message字段以传达错误或缺少返回数据的原因。

让我们使用Java类来表示上述JSON结构：

```java
public class StandardResponse {

    private StatusResponse status;
    private String message;
    private JsonElement data;

    public StandardResponse(StatusResponse status) {
        // ...
    }
    public StandardResponse(StatusResponse status, String message) {
        // ...
    }
    public StandardResponse(StatusResponse status, JsonElement data) {
        // ...
    }

    // getters and setters
}
```

其中StatusResponse是一个枚举，定义如下：

```java
public enum StatusResponse {
    SUCCESS ("Success"),
    ERROR ("Error");
 
    private String status;       
    // constructors, getters
}
```

## 5. 实现RESTful服务

现在让我们实现REST API的路由和处理程序。

### 5.1 创建控制器

以下Java类包含我们的API的路由，包括动词和路径以及每个路由的处理程序的概要：

```java
public class SparkRestExample {
    public static void main(String[] args) {
        post("/users", (request, response) -> {
            //...
        });
        get("/users", (request, response) -> {
            //...
        });
        get("/users/:id", (request, response) -> {
            //...
        });
        put("/users/:id", (request, response) -> {
            //...
        });
        delete("/users/:id", (request, response) -> {
            //...
        });
        options("/users/:id", (request, response) -> {
            //...
        });
    }
}
```

我们将在以下小节中展示每个路由处理程序的完整实现。

### 5.2 添加用户

下面是添加用户的post方法响应处理程序：

```java
post("/users", (request, response) -> {
    response.type("application/json");
    User user = new Gson().fromJson(request.body(), User.class);
    userService.addUser(user);

    return new Gson()
        .toJson(new StandardResponse(StatusResponse.SUCCESS));
});
```

注意：在此示例中，User对象的JSON表示形式作为POST请求的原始主体传递。

让我们测试一下路由：

请求：

```shell
POST http://localhost:4567/users
{
    "id": "1012", 
    "email": "your-email@your-domain.com", 
    "firstName": "Mac",
    "lastName": "Mason1"
}
```

响应：

```json
{
    "status":"SUCCESS"
}
```

### 5.3 获取所有用户

下面是从UserService返回所有用户的get方法响应处理程序：

```java
get("/users", (request, response) -> {
    response.type("application/json");
    return new Gson().toJson(new StandardResponse(StatusResponse.SUCCESS,new Gson()
        .toJsonTree(userService.getUsers())));
});
```

现在让我们测试一下路由：

请求：

```shell
GET http://localhost:4567/users
```

响应：

```json
{
    "status":"SUCCESS",
    "data":[
        {
            "id":"1014",
            "firstName":"John",
            "lastName":"Miller",
            "email":"your-email@your-domain.com"
        },
        {
            "id":"1012",
            "firstName":"Mac",
            "lastName":"Mason1",
            "email":"your-email@your-domain.com"
        }
    ]
}
```

### 5.4 根据ID获取用户

下面是get方法响应处理程序，它返回具有给定id的用户：

```java
get("/users/:id", (request, response) -> {
    response.type("application/json");
    return new Gson().toJson(new StandardResponse(StatusResponse.SUCCESS,new Gson()
        .toJsonTree(userService.getUser(request.params(":id")))));
});
```

现在让我们测试一下路由：

请求：

```shell
GET http://localhost:4567/users/1012
```

响应：

```json
{
    "status":"SUCCESS",
    "data":{
        "id":"1012",
        "firstName":"Mac",
        "lastName":"Mason1",
        "email":"your-email@your-domain.com"
    }
}
```

### 5.5 编辑用户

下面是put方法响应处理程序，它修改路由模式中提供的id的用户：

```java
put("/users/:id", (request, response) -> {
    response.type("application/json");
    User toEdit = new Gson().fromJson(request.body(), User.class);
    User editedUser = userService.editUser(toEdit);
            
    if (editedUser != null) {
        return new Gson().toJson(new StandardResponse(StatusResponse.SUCCESS,new Gson()
            .toJsonTree(editedUser)));
    } else {
        return new Gson().toJson(new StandardResponse(StatusResponse.ERROR,new Gson()
            .toJson("User not found or error in edit")));
    }
});
```

注意：在此示例中，数据作为JSON对象在POST请求的原始主体中传递，其属性名称与要编辑的User对象的字段匹配。

让我们测试一下路由：

请求：

```shell
PUT http://localhost:4567/users/1012
{
    "lastName": "Mason"
}
```

响应：

```json
{
    "status":"SUCCESS",
    "data":{
        "id":"1012",
        "firstName":"Mac",
        "lastName":"Mason",
        "email":"your-email@your-domain.com"
    }
}
```

### 5.6 删除用户

下面是删除方法响应处理程序，它将删除具有给定id的用户：

```java
delete("/users/:id", (request, response) -> {
    response.type("application/json");
    userService.deleteUser(request.params(":id"));
    return new Gson().toJson(new StandardResponse(StatusResponse.SUCCESS, "user deleted"));
});
```

现在，让我们测试一下路由：

请求：

```shell
DELETE http://localhost:4567/users/1012
```

响应：

```json
{
    "status":"SUCCESS",
    "message":"user deleted"
}
```

### 5.7 检查用户是否存在

options方法是条件检查的不错选择。下面是options方法响应处理程序，它将检查是否存在具有给定id的用户：

```java
options("/users/:id", (request, response) -> {
    response.type("application/json");
    return new Gson().toJson(
      new StandardResponse(StatusResponse.SUCCESS,(userService.userExist(request.params(":id"))) ? "User exists" : "User does not exists" ));
});
```

现在让我们测试一下路由：

请求：

```shell
OPTIONS http://localhost:4567/users/1012
```

响应：

```json
{
    "status":"SUCCESS",
    "message":"User exists"
}
```

## 6. 总结

在本文中，我们对用于快速Web开发的Spark框架进行了简单介绍。

该框架主要用于用Java生成微服务，具有Java知识并希望利用基于JVM库构建的库的Node.js开发人员应该可以轻松使用此框架。