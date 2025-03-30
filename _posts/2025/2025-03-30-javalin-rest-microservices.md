---
layout: post
title:  使用Javalin创建REST微服务
category: libraries
copyright: libraries
excerpt: Javalin
---

## 1. 简介

[Javalin](https://javalin.io/)是一个为Java和Kotlin编写的轻量级Web框架，它基于Jetty Web服务器编写，因此性能极佳。Javalin的模型与[koa.js](http://koajs.com/)非常相似，这意味着它是从头开始编写的，易于理解和构建。

在本教程中，我们将介绍使用此轻量级框架构建基本REST微服务的步骤。

## 2. 添加依赖

要创建一个基本的应用程序，我们只需要一个依赖-Javalin本身：

```xml
<dependency>
    <groupId>io.javalin</groupId>
    <artifactId>javalin</artifactId>
    <version>1.6.1</version>
</dependency>
```

当前版本可以在[这里](https://mvnrepository.com/artifact/io.javalin/javalin)找到。

## 3. 设置Javalin

Javalin使设置基本应用程序变得简单，我们将首先定义主类并设置一个简单的“Hello World”应用程序。

让我们在基础包中创建一个名为JavalinApp.java的新文件。

在这个文件中，我们创建一个main方法并添加以下内容来设置一个基本应用程序：

```java
Javalin app = Javalin.create()
    .port(7000)
    .start();
app.get("/hello", ctx -> ctx.html("Hello, Javalin!"));
```

我们创建Javalin的新实例，使其监听端口7000，然后启动应用程序。

我们还设置了第一个端点，用于在/hello端点监听GET请求。

让我们运行这个应用程序并访问[http://localhost:7000/hello](http://localhost:7000/hello)来查看结果。

## 4. 创建UserController

“Hello World”示例非常适合介绍主题，但对于实际应用来说却没有帮助。现在让我们来看看Javalin的一个更现实的用例。

首先，我们需要创建要使用的对象模型。首先 在根项目下创建一个名为user的包。

然后，我们添加一个新的User类：

```java
public class User {
    public final int id;
    public final String name;

    // constructors
}
```

此外，我们还需要设置数据访问对象(DAO)。在本例中，我们将使用内存对象来存储用户。

我们在user包中创建一个名为UserDao.java的新类：

```java
class UserDao {

    private List<User> users = Arrays.asList(
            new User(0, "Steve Rogers"),
            new User(1, "Tony Stark"),
            new User(2, "Carol Danvers")
    );

    private static UserDao userDao = null;

    private UserDao() {
    }

    static UserDao instance() {
        if (userDao == null) {
            userDao = new UserDao();
        }
        return userDao;
    }

    Optional<User> getUserById(int id) {
        return users.stream()
                .filter(u -> u.id == id)
                .findAny();
    }

    Iterable<String> getAllUsernames() {
        return users.stream()
                .map(user -> user.name)
                .collect(Collectors.toList());
    }
}
```

将我们的DAO实现为单例使其在示例中更易于使用，我们还可以将其声明为主类的静态成员，或者使用Guice等库中的依赖注入(如果我们愿意的话)。

最后，我们要创建控制器类。Javalin允许我们在声明路由处理程序时非常灵活，因此这只是定义它们的一种方式。

我们在user包中创建一个名为UserController.java的新类：

```java
public class UserController {
    public static Handler fetchAllUsernames = ctx -> {
        UserDao dao = UserDao.instance();
        Iterable<String> allUsers = dao.getAllUsernames();
        ctx.json(allUsers);
    };

    public static Handler fetchById = ctx -> {
        int id = Integer.parseInt(Objects.requireNonNull(ctx.param("id")));
        UserDao dao = UserDao.instance();
        Optional<User> user = dao.getUserById(id);
        if (user.isPresent()) {
            ctx.json(user);
        } else {
            ctx.html("Not Found");
        }
    };
}
```

通过将处理程序声明为静态，我们确保控制器本身不保存任何状态。但是在更复杂的应用程序中，我们可能希望在请求之间存储状态，在这种情况下，我们需要删除static修饰符。

还要注意，使用静态方法进行单元测试更加困难，所以如果我们想要那种级别的测试，我们将需要使用非静态方法。

## 5. 添加路由

现在，我们有多种方法从模型中获取数据。最后一步是通过REST端点公开这些数据，我们需要在主应用程序中注册两个新路由。

让我们将它们添加到我们的主要应用程序类中：

```java
app.get("/users", UserController.fetchAllUsernames);
app.get("/users/:id", UserController.fetchById);
```

编译并运行应用程序后，我们可以向每个新端点发出请求。调用“http://localhost:7000/users”将列出所有用户，调用“http://localhost:7000/users/0”将获取ID为0的单个用户JSON对象，我们现在有一个允许我们检索用户数据的微服务。

## 6. 扩展路由

检索数据是大多数微服务的一项重要任务。

但是，我们还需要能够将数据存储在我们的数据存储中，Javalin提供了构建服务所需的全套路径处理程序。

我们上面看到了GET的示例，但PATCH、POST、DELETE和PUT也是可能的。

此外，如果我们将Jackson作为依赖，我们可以自动将JSON请求主体解析到我们的模型类中。例如：

```java
app.post("/") { ctx ->
    User user = ctx.bodyAsClass(User.class);
}
```

允许我们从请求正文中获取JSON User对象并将其转换为User模型对象。

## 7. 总结

我们可以结合所有这些技术来创建我们的微服务。

在本文中，我们了解了如何设置Javalin并构建一个简单的应用程序。我们还讨论了如何使用不同的HTTP方法类型让客户端与我们的服务进行交互。

有关如何使用Javalin的更多高级示例，请务必查看[文档](https://javalin.io/documentation)。