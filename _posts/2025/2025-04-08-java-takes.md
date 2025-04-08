---
layout: post
title:  Takes简介
category: webmodules
copyright: webmodules
excerpt: Takes
---

## 1. 概述

Java生态系统中有许多Web框架，例如[Spring](https://www.baeldung.com/spring-tutorial)、[Play](https://www.baeldung.com/java-intro-to-the-play-framework)和[Grails](https://www.baeldung.com/grails-gorm-tutorial)。但是，没有一个框架可以声称是完全不可变和面向对象的。

在本教程中，我们将探索Takes框架并使用其常见功能(如路由、请求/响应处理和单元测试)创建一个简单的Web应用程序。

## 2. Takes

**Takes是一个不可变的Java 8 Web框架，既不使用null也不使用公共静态方法**。

此外，**该框架不支持可变类、强制类型转换或反射**。因此，它是一个真正的面向对象框架。

Takes不需要配置文件即可进行设置。除此之外，它还提供JSON/XML响应和模板等内置功能。

## 3. 设置

首先，我们将最新的Maven[依赖](https://mvnrepository.com/artifact/org.takes/takes)添加到pom.xml中：

```xml
<dependency>
    <groupId>org.takes</groupId>
    <artifactId>takes</artifactId>
    <version>1.19</version>
</dependency>
```

然后，让我们创建实现[Take](https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/Take.html)接口的TakesHelloWorld类：

```java
public class TakesHelloWorld implements Take {
    @Override
    public Response act(Request req) {
        return new RsText("Hello, world!");
    }
}
```

Take接口提供了框架的基本功能，每个Take都充当请求处理程序，通过act方法返回响应。

在这里，当向TakesHelloWorld发出请求时，我们使用[RsText](https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/rs/RsText.html)类来呈现纯文本“Hello,world!”作为响应。

接下来，我们将创建TakesApp类来启动Web应用程序：

```java
public class TakesApp {
    public static void main(String... args) {
        new FtBasic(new TakesHelloWorld()).start(Exit.NEVER);
    }
}
```

在这里，我们使用了提供[Front](https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/http/Front.html)接口基本实现的[FtBasic](https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/http/FtBasic.html)类来启动Web服务器并将请求转发给TakesHelloWorld。

Takes使用[ServerSocket](https://www.baeldung.com/a-guide-to-java-sockets)类实现了自己的无状态Web服务器，默认情况下，它在端口80上启动服务器。但是，我们可以在代码中定义端口：

```java
new FtBasic(new TakesHelloWorld(), 6060).start(Exit.NEVER);
```

或者，**我们可以使用命令行参数–port传递端口号**。

然后，让我们使用Maven命令编译类：

```shell
mvn clean package
```

现在，我们准备在IDE中将TakesApp类作为简单的Java应用程序运行。

## 4. 运行

我们还可以将TakesApp类作为单独的Web服务器应用程序运行。

### 4.1 Java命令行

首先，编译我们的类：

```shell
javac -cp "takes.jar:." cn.tuyucheng.taketoday.takes.*
```

然后，我们可以使用Java命令行运行该应用程序：

```shell
java -cp "takes.jar:." cn.tuyucheng.taketoday.takes.TakesApp --port=6060
```

### 4.2 Maven

或者，我们**可以使用[exec-maven-plugin](https://mvnrepository.com/artifact/org.codehaus.mojo/exec-maven-plugin)插件通过Maven运行它**：

```xml
<profiles>
    <profile>
        <id>reload</id>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.codehaus.mojo</groupId>
                    <artifactId>exec-maven-plugin</artifactId>
                    <version>3.1.0</version>
                    <executions>
                        <execution>
                            <id>start-server</id>
                            <phase>pre-integration-test</phase>
                            <goals>
                                <goal>java</goal>
                            </goals>
                        </execution>
                    </executions>
                    <configuration>
                        <mainClass>cn.tuyucheng.taketoday.takes.TakesApp</mainClass>
                        <cleanupDaemonThreads>false</cleanupDaemonThreads>
                        <arguments>
                            <argument>--port=${port}</argument>
                        </arguments>
                    </configuration>
                </plugin>
            </plugins>
        </build>
    </profile>
</profiles>
```

现在，我们可以使用Maven命令运行我们的应用程序：

```shell
mvn clean integration-test -Preload -Dport=6060
```

## 5. 路由

该框架提供[TkFork](https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/facets/fork/TkFork.html)类来将请求路由到不同的take。

例如，让我们向我们的应用程序添加一些路由：

```java
public static void main(String... args) {
    new FtBasic(
        new TkFork(
            new FkRegex("/", new TakesHelloWorld()),
            new FkRegex("/contact", new TakesContact())
        ), 6060
    ).start(Exit.NEVER);
}
```

这里，我们使用了[FkRegex](https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/facets/fork/FkRegex.html)类来匹配请求路径。

## 6. 请求处理

框架在org.takes.rq包中提供了一些[装饰器类](https://www.baeldung.com/java-decorator-pattern)来处理HTTP请求。

例如，我们可以使用[RqMethod](https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/rq/RqMethod.html)接口来提取HTTP方法：

```java
public class TakesHelloWorld implements Take { 
    @Override
    public Response act(Request req) throws IOException {
        String requestMethod = new RqMethod.Base(req).method(); 
        return new RsText("Hello, world!"); 
    }
}
```

类似地，[RqHeaders](https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/rq/RqHeaders.html)接口可用于获取请求标头：

```java
Iterable<String> requestHeaders = new RqHeaders.Base(req).head();
```

我们可以使用[RqPrint](https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/rq/RqPrint.html)类来获取请求的主体：

```java
String body = new RqPrint(req).printBody();
```

同样，我们可以使用[RqFormSmart](https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/rq/form/RqFormSmart.html)类来访问表单参数：

```java
String username = new RqFormSmart(req).single("username");
```

## 7. 响应处理

Takes还在org.takes.rs包中提供了许多有用的装饰器来处理HTTP响应。

响应装饰器实现了[Response](https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/Response.html)接口的head和body方法。

例如，[RsWithStatus](https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/rs/RsWithStatus.html)类使用状态码呈现响应：

```java
Response resp = new RsWithStatus(200);
```

可以使用head方法验证响应的输出：

```java
assertEquals("[HTTP/1.1 200 OK], ", resp.head().toString());
```

类似地，[RsWithType](https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/rs/RsWithType.html)类使用内容类型呈现响应：

```java
Response resp = new RsWithType(new RsEmpty(), "text/html");
```

这里，[RsEmpty](https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/rs/RsEmpty.html)类呈现空响应。

同样，我们可以使用[RsWithBody](https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/rs/RsWithBody.html)类来呈现带有主体的响应。

因此，让我们创建TakesContact类并使用讨论的装饰器来呈现响应：

```java
public class TakesContact implements Take {
    @Override
    public Response act(Request req) throws IOException {
        return new RsWithStatus(
                new RsWithType(
                        new RsWithBody("Contact us at https://www.tuyucheng.com"),
                        "text/html"), 200);
    }
}
```

类似地，我们可以使用[RsJson](https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/rs/RsJson.html)类来呈现JSON响应：

```java
@Override 
public Response act(Request req) { 
    JsonStructure json = Json.createObjectBuilder() 
        .add("id", rs.getInt("id")) 
        .add("user", rs.getString("user")) 
        .build(); 
    return new RsJson(json); 
}
```

## 8. 异常处理

**该框架包含[Fallback](https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/facets/fallback/Fallback.html)接口来处理异常情况**，它还提供了一些实现来处理回退场景。

例如，让我们使用[TkFallback](https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/facets/fallback/TkFallback.html)类来处理HTTP 404并向用户显示一条消息：

```java
public static void main(String... args) throws IOException, SQLException {
    new FtBasic(
            new TkFallback(
                    new TkFork(
                            new FkRegex("/", new TakesHelloWorld()),
                            // ...
                    ),
                    new FbStatus(404, new RsText("Page Not Found"))), 6060
    ).start(Exit.NEVER);
}
```

在这里，我们使用了[FbStatus](https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/facets/fallback/FbStatus.html)类来处理定义的状态码的回退。

类似地，我们可以使用[FbChain](https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/facets/fallback/FbChain.html)类来定义回退组合：

```java
new TkFallback(
    new TkFork(
        // ...
        ),
    new FbChain(
        new FbStatus(404, new RsText("Page Not Found")),
        new FbStatus(405, new RsText("Method Not Allowed"))
        )
    ), 6060
).start(Exit.NEVER);
```

另外，我们可以实现Fallback接口来处理异常：

```java
new FbChain(
    new FbStatus(404, new RsText("Page Not Found")),
    new FbStatus(405, new RsText("Method Not Allowed")),
    new Fallback() {
        @Override
        public Opt<Response> route(RqFallback req) {
          return new Opt.Single<Response>(new RsText(req.throwable().getMessage()));
        }
    }
)
```

## 9. 模板

让我们将[Apache Velocity](https://www.baeldung.com/apache-velocity)与我们的Takes Web应用程序集成以提供一些模板功能。

首先，我们将添加[velocity-engine-core](https://mvnrepository.com/artifact/org.apache.velocity/velocity-engine-core) Maven依赖：

```xml
<dependency>
    <groupId>org.apache.velocity</groupId>
    <artifactId>velocity-engine-core</artifactId>
    <version>2.2</version>
</dependency>
```

然后，我们将使用[RsVelocity](https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/rs/RsVelocity.html)类来定义模板字符串和act方法中的绑定参数：

```java
public class TakesIndex implements Take {
    @Override
    public Response act(Request req) throws IOException {
        return new RsHtml(
            new RsVelocity("${username}", new RsVelocity.Pair("username", "Tuyucheng")));
        );
    }
}
```

在这里，我们使用[RsHtml](https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/rs/RsHtml.html)类来呈现HTML响应。

另外，我们可以使用带有RsVelocity类的速度模板：

```java
new RsVelocity(this.getClass().getResource("/templates/index.vm"), 
    new RsVelocity.Pair("username", username))
);
```

## 10. 单元测试

该框架通过提供创建虚假请求的[RqFake](https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/rq/RqFake.html)类来支持任何Take的单元测试：

例如，让我们使用[JUnit](https://www.baeldung.com/junit-5)为TakesContact类编写一个单元测试：

```java
String resp = new RsPrint(new TakesContact().act(new RqFake())).printBody();
assertEquals("Contact us at https://www.tuyucheng.com", resp);
```

## 11. 集成测试

我们可以使用JUnit和任何HTTP客户端测试整个应用程序。

该框架提供了[FtRemote](https://www.javadoc.io/doc/org.takes/takes/latest/org/takes/http/FtRemote.html)类，可以在随机端口上启动服务器，并为Take的执行提供远程控制。

例如，让我们编写一个集成测试并验证TakesContact类的响应：

```java
new FtRemote(new TakesContact()).exec(
    new FtRemote.Script() {
        @Override
        public void exec(URI home) throws IOException {
            HttpClient client = HttpClientBuilder.create().build();    
            HttpResponse response = client.execute(new HttpGet(home));
            int statusCode = response.getStatusLine().getStatusCode();
            HttpEntity entity = response.getEntity();
            String result = EntityUtils.toString(entity);
            
            assertEquals(200, statusCode);
            assertEquals("Contact us at https://www.tuyucheng.com", result);
        }
    });
```

在这里，我们使用[Apache HttpClient](https://www.baeldung.com/httpclient-status-code)向服务器发出请求并验证响应。

## 12. 总结

在本教程中，我们通过创建一个简单的Web应用程序探索了Takes框架。

首先，我们看到了在Maven项目中快速设置框架并运行应用程序的一种方法。

然后，我们研究了一些常见的功能，如路由、请求/响应处理和单元测试。

最后，我们探讨了框架提供的单元和集成测试的支持。