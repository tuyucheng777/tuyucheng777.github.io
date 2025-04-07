---
layout: post
title:  Jooby简介
category: webmodules
copyright: webmodules
excerpt: Jooby
---

## 1. 概述

[Jooby](https://jooby.io/)是一个可扩展且快速的微Web框架，建立在最常用的NIO Web服务器之上。它非常简单且模块化，专为现代Web架构而设计，还支持Javascript和Kotlin。

默认情况下，Jooby对Netty、Jetty和Undertow提供了强大的支持。

在本文中，我们将了解Jooby的整体项目结构以及如何使用Jooby构建一个简单的Web应用程序。

## 2. 应用程序架构

一个简单的Jooby应用程序结构如下所示：

```text
├── public
|   └── welcome.html
├── conf
|   ├── application.conf
|   └── logback.xml
└── src
|   ├── main
|   |   └── java
|   |       └── com
|   |           └── tuyucheng
|   |               └── jooby
|   |                   └── App.java
|   └── test
|       └── java
|           └── com
|               └── baeldung
|                   └── jooby
|                       └── AppTest.java
├── pom.xml
```

这里要注意的一点是，在public目录中，我们可以放置静态文件，例如css/js/html等。在conf目录中，我们可以放置应用程序需要的任何配置文件，例如logback.xml或application.conf等。

## 3. Maven依赖

我们可以通过在pom.xml中添加以下依赖来创建一个简单的Jooby应用程序：

```xml
<dependency>
    <groupId>io.jooby</groupId>
    <artifactId>jooby-netty</artifactId>
    <version>2.16.1</version>
</dependency>
```

如果我们要选择Jetty或Undertow，我们可以使用以下依赖：

```xml
<dependency>
    <groupId>io.jooby</groupId>
    <artifactId>jooby-jetty</artifactId>
    <version>2.16.1</version>
</dependency>
<dependency>
    <groupId>io.jooby</groupId>
    <artifactId>jooby-undertow</artifactId>
    <version>2.16.1</version>
</dependency>
```

你可以在[中央Maven仓库](https://mvnrepository.com/search?q=jooby)中查看Jooby项目的最新版本。

## 4. 构建应用程序

### 4.1 启动服务器

要启动嵌入式服务器，我们需要使用以下代码片段：

```java
public class App extends Jooby {
    public static void main(String[] args) {
        run(args, App::new);
    }
}
```

一旦启动，服务器将在默认端口8080上运行。

我们还可以使用自定义端口和自定义HTTPS端口配置后端服务器：

```java
{
    setServerOptions(new ServerOptions().
        .setPort(8080)
        .setSecurePort(8433));
}
```

### 4.2 实现路由

在Jooby中创建基于路径的路由非常简单，例如，我们可以按照以下方式为路径“/login”创建路由：

```java
{
    get( "/login", ctx -> "Hello from Tuyucheng");
}
```

类似地，如果我们想要处理其他HTTP方法(如POST、PUT等)，我们可以使用以下代码片段：

```java
{
    post("/save", ctx -> {
        String userId = ctx.query("id").value();
        return userId;
    });
}
```

这里，我们从请求中获取请求参数名称id。有几种参数类型：header、cookie、path、query、form、multipart、session和flash，它们都共享一个统一/类型安全的API来访问和操作它们的值。

我们可以通过以下方式检查任何URI路径参数：

```java
{
    get("/user/{id}", ctx -> "Hello user : " + ctx.path("id").value());
    get("/user/:id", ctx -> "Hello user: " + ctx.path("id").value());
}
```

我们可以使用上述任何一种方法，也可以查找以固定内容开头的URI路径参数。例如，我们可以按以下方式查找以“uid:”开头的URI路径参数：

```java
{
    get("/uid:{id}", ctx -> "Hello User with id : uid = " + ctx.path("id").value());
}
```

### 4.3 实现MVC模式控制器

对于企业级应用程序，Jooby带有MVC API，与任何其他MVC框架(如Spring MVC)非常相似。

例如，我们可以处理名为'/hello'的路径：

```java
@Path("/submit")
public class PostController {

    @POST
    public String hello() {
        return "Submit Tuyucheng";
    }
}
```

非常相似，我们可以创建一个处理程序来使用[@POST](https://javadoc.io/static/org.jooby/jooby/1.2.3/org/jooby/mvc/POST.html)、[@PUT](https://javadoc.io/static/org.jooby/jooby/1.2.3/org/jooby/mvc/PUT.html)、[@DELETE](https://javadoc.io/static/org.jooby/jooby/1.2.3/org/jooby/mvc/DELETE.html)等注解来处理其他HTTP方法。

### 4.4 处理静态内容

为了提供任何静态内容(如HTML、Javascript、CSS、图像等)，我们需要将这些文件放在public目录中。

放置完成后，我们可以从路由将任何URL映射到这些资源：

```java
{
    assets("/employee", "public/form.html");
}
```

### 4.5 处理表单

Jooby的[Request](https://javadoc.io/static/org.jooby/jooby/1.2.3/org/jooby/Request.html)接口默认处理任何表单对象，而无需使用任何手动类型转换。

假设我们需要通过表单提交员工详细信息，第一步，我们需要创建一个用于保存数据的Employee Bean对象：

```java
public class Employee {
    String id;
    String name;
    String email;

    // standard constructors, getters and setters
}
```

现在，我们需要创建一个页面来创建表单：

```html
<form enctype="application/x-www-form-urlencoded" action="/submitForm" method="post">
    <label>Employed id:</label><br>
    <input name="id"/><br>
    <label>Name:</label><br>
    <input name="name"/><br>
    <label>Email:</label><br>
    <input name="email"/><br><br>
    <input type="submit" value="Submit"/>
</form>
```

接下来，我们将创建一个POST处理程序来处理此表单并获取提交的数据：

```java
post("/submitForm", ctx -> {
    Employee employee = ctx.path(Employee.class);
    // ...
    return "employee data saved successfully";
});
```

这里需要注意的是，我们必须将表单enctype声明为application/x-www-form-urlencoded，才能支持动态表单绑定。

### 4.6 会话

Jooby有3种类型的会话实现：内存会话、签名会话和带存储的内存会话。

#### 内存会话

实现内存中的会话管理非常简单，在这个过程中，Jooby将会话数据存储在内存中，并使用cookie/header来读取/保存会话ID。

```java
{
    get("/sessionInMemory", ctx -> {
        Session session = ctx.session();
        session.put("token", "value");
        return session.get("token").value();
    });
}
```

#### 签名会话

签名会话是一个无状态会话存储，需要在每个请求中找到一个会话令牌。在这种情况下，会话不保留任何状态。

让我们举个例子：

```java
{
    String secret = "super secret token";

    setSessionStore(SessionStore.signed(secret));

    get("/signedSession", ctx -> {
        Session session = ctx.session();
        session.put("token", "value");
        return session.get("token").value();
    });
}
```

在上面的例子中，我们使用secret作为密钥来签署数据，并使用该secret创建了一个cookie会话存储。

#### 存储会话

除了内置内存存储外，Jooby还提供以下功能：

- Caffeine：使用Caffeine缓存的内存会话存储
- JWT：JSON Web Token会话存储
- Redis：Redis会话存储

例如，为了实现基于Redis的会话存储，我们需要添加以下Maven依赖：

```xml
<dependency>
    <groupId>io.jooby</groupId>
    <artifactId>jooby-redis</artifactId>
    <version>2.16.1</version>
</dependency>
```

现在我们可以使用下面的代码片段来启用会话管理：

```java
{
    install(new RedisModule("redis"));
    setSessionStore(new RedisSessionStore(require(RedisClient.class)));
    get("/redisSession", ctx -> {
        Session session = ctx.session();
        session.put("token", "value");
        return session.get("token");
    });
}
```

这里要注意的一点是，我们可以通过添加以下行将Redis url配置为application.conf中的“db”属性：

```properties
redis = "redis://localhost:6379"
```

有关更多示例，请参阅[Jooby文档](https://jooby.io/#session-signed-session)

## 5. 测试

测试MVC路由确实很容易，因为路由与某些类的策略绑定，MockRouter允许你执行路由功能。

在测试之前我们需要添加Jooby测试依赖：

```xml
<dependency>
    <groupId>io.jooby</groupId>
    <artifactId>jooby-test</artifactId>
    <version>2.16.1</version>
</dependency>
```

例如，我们可以快速为默认URL创建一个测试用例：

```java
public class AppUnitTest {

    @Test
    public void given_defaultUrl_with_mockrouter_expect_fixedString() {
        MockRouter router = new MockRouter(new App());
        assertEquals("Hello World!", router.get("/")
            .value());
    }
}
```

在上面的例子中，MockRouter返回路由处理程序生成的值。

Jooby通过JUnit 5支持集成测试。

我们来看看下面的例子：

```java
@JoobyTest(value = App.class, port = 8080)
public class AppLiveTest {

    static OkHttpClient client = new OkHttpClient();

    @Test
    public void given_defaultUrl_expect_fixedString() {
        Request request = new Request.Builder().url("http://localhost:8080").build();

        try (Response response = client.newCall(request).execute()) {
            assertEquals("Hello World!", response.body().string());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

@JoobyTest注解负责启动和停止应用程序，默认情况下，端口为8911。

你可以使用port()方法来改变它：

```java
@JoobyTest(value = App.class, port = 8080)
```

在上面的例子中，我们在类级别添加了@JoobyTest注解，也可以在方法级别使用此注解。有关更多示例，请查看Jooby[官方文档](https://jooby.io/#testing-integration-testing)。

## 6. 总结

在本教程中，我们探索了Jooby项目及其基本功能。
