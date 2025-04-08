---
layout: post
title:  Ratpack HTTP客户端
category: webmodules
copyright: webmodules
excerpt: Ratpack
---

## 1. 简介

在过去的几年中，我们见证了使用Java创建应用程序的函数式和响应式方式的兴起，Ratpack提供了一种以相同方式创建HTTP应用程序的方法。

由于它使用Netty来满足网络需求，因此**它完全是异步和非阻塞的**；Ratpack还通过提供配套测试库来支持测试。

在本教程中，我们将介绍Ratpack HTTP客户端和相关组件的使用。

在此过程中，我们将尝试从[Ratpack入门教程](https://www.baeldung.com/ratpack)结束时的要点出发，进一步加深理解。

## 2. Maven依赖

首先，让我们添加所需的[Ratpack依赖](https://mvnrepository.com/search?q=io.ratpack)：

```xml
<dependency>
    <groupId>io.ratpack</groupId>
    <artifactId>ratpack-core</artifactId>
    <version>1.5.4</version>
</dependency>
<dependency>
    <groupId>io.ratpack</groupId>
    <artifactId>ratpack-test</artifactId>
    <version>1.5.4</version>
    <scope>test</scope>
</dependency>
```

始终可以选择使用其他Ratpack库来添加和扩展。

## 3. 背景

在深入研究之前，让我们先了解一下Ratpack应用程序中的完成方式。

### 3.1 基于处理程序的方法

Ratpack使用基于处理程序的方法来处理请求，这个想法本身很简单。

最简单的形式是，我们可以让每个处理程序在每个特定路径上处理请求：

```java
public class FooHandler implements Handler {
    @Override
    public void handle(Context ctx) throws Exception {
        ctx.getResponse().send("Hello Foo!");
    }
}
```

### 3.2 链、注册表和上下文

**处理程序使用Context对象与传入请求进行交互**，通过它，我们可以访问HTTP请求和响应，以及委托给其他处理程序的功能。

以下面的处理程序为例：

```java
Handler allHandler = context -> {
    Long id = Long.valueOf(context.getPathTokens().get("id"));
    Employee employee = new Employee(id, "Mr", "NY");
    context.next(Registry.single(Employee.class, employee));
};
```

该处理程序负责进行一些预处理，将结果放入Registry，然后将请求委托给其他处理程序。

**通过使用Registry，我们可以实现处理程序之间的通信**。以下处理程序使用对象类型从Registry查询先前计算的结果：

```java
Handler empNameHandler = ctx -> {
    Employee employee = ctx.get(Employee.class);
    ctx.getResponse()
        .send("Name of employee with ID " + employee.getId() + " is " + employee.getName());
};
```

我们应该记住，在生产应用程序中，我们会将这些处理程序作为单独的类，以便更好地抽象、调试和开发复杂的业务逻辑。

**现在我们可以在Chain中使用这些处理程序来创建复杂的自定义请求处理管道**。

例如：

```java
Action<Chain> chainAction = chain -> chain.prefix("employee/:id", empChain -> {
    empChain.all(allHandler)
        .get("name", empNameHandler)
        .get("title", empTitleHandler);
});
```

我们可以进一步采用这种方法，通过使用Chain中的insert(...)方法将多个链组合在一起，并让每个链负责不同的关注点。

以下测试用例展示了这些结构的使用：

```java
@Test
public void givenAnyUri_GetEmployeeFromSameRegistry() throws Exception {
    EmbeddedApp.fromHandlers(chainAction)
            .test(testHttpClient -> {
                assertEquals("Name of employee with ID 1 is NY", testHttpClient.get("employee/1/name")
                        .getBody()
                        .getText());
                assertEquals("Title of employee with ID 1 is Mr", testHttpClient.get("employee/1/title")
                        .getBody()
                        .getText());
            });
}
```

在这里，我们使用Ratpack的测试库来单独测试我们的功能，而无需启动实际的服务器。

## 4. HTTP与Ratpack

### 4.1 努力实现异步

HTTP协议本质上是同步的；因此，Web应用程序通常是同步的，导致会阻塞。这是一种极其耗费资源的方法，因为我们会为每个传入请求创建一个线程。

我们宁愿创建非阻塞和异步应用程序，这将确保我们只需要使用一小部分线程来处理请求。

### 4.2 回调函数

**处理异步API时，我们通常会向接收方提供回调函数，以便将数据返回给调用方。在Java中，这通常采用匿名内部类和Lambda表达式的形式**。但是随着我们的应用程序扩展，或者有多个嵌套的异步调用，这样的解决方案将难以维护且更难调试。

Ratpack以Promise的形式提供了一种优雅的解决方案来处理这种复杂性。

### 4.3 Ratpack Promise

Ratpack Promise可以被视为类似于Java Future对象，**它本质上是稍后可用的值的表示**。

我们可以指定值可用时要经过的一系列操作，每个操作都会返回一个新的Promise对象，即前一个Promise对象的转换版本。

正如我们所料，这会导致线程之间的上下文切换很少，从而使我们的应用程序更加高效。

以下是使用Promise的处理程序实现：

```java
public class EmployeeHandler implements Handler {
    @Override
    public void handle(Context ctx) throws Exception {
        EmployeeRepository repository = ctx.get(EmployeeRepository.class);
        Long id = Long.valueOf(ctx.getPathTokens().get("id"));
        Promise<Employee> employeePromise = repository.findEmployeeById(id);
        employeePromise.map(employee -> employee.getName())
                .then(name -> ctx.getResponse()
                        .send(name));
    }
}
```

我们需要记住，**当我们定义如何处理最终值时，Promise特别有用**，我们可以通过在其上调用终端操作then(Action)来实现这一点。

如果我们需要发回一个Promise但数据源是同步的，我们仍然可以这样做：

```java
@Test
public void givenSyncDataSource_GetDataFromPromise() throws Exception {
    String value = ExecHarness.yieldSingle(execution -> Promise.sync(() -> "Foo"))
        .getValueOrThrow();
    assertEquals("Foo", value);
}
```

### 4.4 HTTP客户端

Ratpack提供了一个异步HTTP客户端，可以从服务器注册表中检索该客户端的实例。但是，**我们鼓励创建和使用替代实例，因为默认实例不使用连接池，并且默认值相当保守**。

我们可以使用of(Action)方法创建一个实例，该方法以HttpClientSpec类型的Action作为参数。

利用这一点，我们可以根据自己的偏好调整客户端：

```java
HttpClient httpClient = HttpClient.of(httpClientSpec -> {
    httpClientSpec.poolSize(10)
        .connectTimeout(Duration.of(60, ChronoUnit.SECONDS))
        .maxContentLength(ServerConfig.DEFAULT_MAX_CONTENT_LENGTH)
        .responseMaxChunkSize(16384)
        .readTimeout(Duration.of(60, ChronoUnit.SECONDS))
        .byteBufAllocator(PooledByteBufAllocator.DEFAULT);
});
```

我们可能已经猜到了它的异步特性，HttpClient返回一个Promise对象。因此，我们可以以非阻塞方式拥有复杂的操作管道。

为了说明，让我们让客户端使用这个HttpClient调用我们的EmployeeHandler：

```java
public class RedirectHandler implements Handler {

    @Override
    public void handle(Context ctx) throws Exception {
        HttpClient client = ctx.get(HttpClient.class);
        URI uri = URI.create("http://localhost:5050/employee/1");
        Promise<ReceivedResponse> responsePromise = client.get(uri);
        responsePromise.map(response -> response.getBody()
                        .getText()
                        .toUpperCase())
                .then(responseText -> ctx.getResponse()
                        .send(responseText));
    }
}
```

使用[cURL](https://www.baeldung.com/curl-rest)调用可以确认我们得到了预期的响应：

```shell
curl http://localhost:5050/redirect
JANE DOE
```

## 5. 总结

在本文中，我们介绍了Ratpack中可用的主要库构造，这些构造使我们能够开发非阻塞和异步Web应用程序。

我们了解了Ratpack HttpClient和随附的Promise类，该类代表Ratpack中的所有异步操作；我们还了解了如何使用随附的TestHttpClient轻松测试我们的HTTP应用程序。