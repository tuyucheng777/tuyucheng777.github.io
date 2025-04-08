---
layout: post
title:  Ratpack与Groovy
category: webmodules
copyright: webmodules
excerpt: Ratpack
---

## 1. 概述

[Ratpack](https://www.baeldung.com/ratpack)是一组轻量级Java库，用于构建具有响应式、异步和非阻塞功能的可扩展HTTP应用程序。

此外，Ratpack还提供与[Google Guice](https://www.baeldung.com/ratpack-google-guice)、[Spring Boot](https://www.baeldung.com/ratpack-spring-boot)、[RxJava](https://www.baeldung.com/ratpack-rxjava)和[Hystrix](https://www.baeldung.com/ratpack-hystrix)等技术和框架的集成。

在本教程中，我们将探讨如何**将Ratpack与Groovy结合使用**。

## 2. 为什么选择Groovy？

[Groovy](https://www.baeldung.com/groovy-language)是一种在JVM中运行的强大的动态语言。

因此，Groovy使脚本和领域特定语言变得非常简单。借助Ratpack，这可以实现快速的Web应用程序开发。

**Ratpack通过ratpack-groovy和ratpack-groovy-test库轻松与Groovy集成**。

## 3. 使用Groovy脚本的Ratpack应用程序

[Ratpack Groovy API](https://github.com/ratpack/ratpack/tree/master/ratpack-groovy)以Java构建，因此可以轻松与Java和Groovy应用程序集成，它们位于ratpack.groovy包中。

实际上，结合Groovy的脚本功能和Grape依赖管理，我们只需几行代码就可以快速创建一个由Ratpack驱动的Web应用程序：

```groovy
@Grab('io.ratpack:ratpack-groovy:1.6.1')
import static ratpack.groovy.Groovy.ratpack

ratpack {
    handlers {
        get {
            render 'Hello World from Ratpack with Groovy!!'
        }    
    }
}
```

**这是我们的第一个处理程序**，用于处理GET请求，我们所要做的就是添加一些基本的DSL来启用Ratpack服务器。

现在让我们将其作为Groovy脚本运行以启动应用程序，默认情况下，该应用程序将在[http://localhost:5050](http://localhost:5050/)上可用：

```shell
$ curl -s localhost:5050
Hello World from Ratpack with Groovy!!
```

我们还可以使用ServerConfig配置端口：

```groovy
ratpack {
    serverConfig {
        port(5056)
    }
}
```

**Ratpack还提供了热重载功能**，这意味着我们可以更改Ratpack.groovy，然后在应用程序处理我们的下一个HTTP请求时看到更改。

## 4. Ratpack-Groovy依赖管理

有多种方法可以启用ratpack-groovy支持。

### 4.1 Grape

我们可以使用Groovy的嵌入式依赖管理器Grape。

这就像在我们的Ratpack.groovy脚本中添加注解一样简单：

```groovy
@Grab('io.ratpack:ratpack-groovy:1.6.1')
import static ratpack.groovy.Groovy.ratpack
```

### 4.2 Maven依赖

为了在Maven中构建，我们需要做的就是添加[ratpack-groovy库](https://mvnrepository.com/artifact/io.ratpack/ratpack-groovy)的依赖：

```xml
<dependency>
    <groupId>io.ratpack</groupId>
    <artifactId>ratpack-groovy</artifactId>
    <version>${ratpack.version}</version>
</dependency>
```

### 4.3 Gradle

我们可以通过在build.gradle中添加Ratpack的Groovy Gradle插件来启用ratpack-groovy集成：

```shell
plugins { 
  id 'io.ratpack.ratpack-groovy' version '1.6.1' 
}
```

## 5. Groovy中的Ratpack处理程序

**处理程序提供了一种处理Web请求和响应的方法**，可在此闭包中访问请求和响应对象。

我们可以使用GET和POST等HTTP方法处理Web请求：

```groovy
handlers { 
    get("greet/:name") { ctx ->
        render "Hello " + ctx.getPathTokens().get("name") + " !!!"
    }
}      
```

我们可以通过[http://localhost:5050/greet/](http://localhost:5050/greet/Norman)测试此Web请求：

```shell
$ curl -s localhost:5050/greet/Norman
Hello Norman!!!
```

在处理程序的代码中，ctx是Context注册表对象，它授予对路径变量、请求和响应对象的访问权限。

处理程序还支持通过Jackson处理JSON。

让我们返回从Groovy Map转换的JSON：

```groovy
get("data") {
    render Jackson.json([title: "Mr", name: "Norman", country: "USA"])
}
```

```shell
$ curl -s localhost:5050/data
{"title":"Mr","name":"Norman","country":"USA"}
```

这里使用Jackson.json进行转换。

## 6. Groovy中的Ratpack Promises

我们知道，Ratpack在应用程序中启用了异步和非阻塞功能，这是通过[Ratpack Promises](https://www.baeldung.com/ratpack-http-client)实现的。

Promise与JavaScript中使用的Promise类似，有点像Java中的[Future](https://www.baeldung.com/java-future)；我们可以将Promise视为将来可用的值的表示：

```groovy
post("user") {
    Promise<User> user = parse(Jackson.fromJson(User)) 
    user.then { u -> render u.name } 
}
```

这里的最后一个操作是then操作，它决定如何处理最终值。在本例中，我们将其作为对POST的响应返回。

让我们更详细地了解这段代码；在这里，Jackson.fromJson使用ObjectMapper User解析请求主体的JSON。然后，内置的Context.parse方法将其绑定到Promise对象。

Promise是异步操作的，当then操作最终执行时，将返回响应：

```shell
curl -X POST -H 'Content-type: application/json' --data \
'{"id":3,"title":"Mrs","name":"Jiney Weiber","country":"UK"}' \
http://localhost:5050/employee

Jiney Weiber
```

我们应该注意到Promise库非常丰富，允许我们使用map和flatMap等函数链接操作。

## 7. 与数据库集成

当我们的处理程序必须等待服务时，异步处理程序的优势最大，让我们通过将Ratpack应用程序与H2数据库集成来演示这一点。

我们可以使用Ratpack的HikariModule类(它是[HikariCP](https://www.baeldung.com/hikaricp) JDBC连接池的扩展)或Groovy [Sql](http://docs.groovy-lang.org/latest/html/api/groovy/sql/Sql.html)进行数据库集成。

### 7.1 HikariModule

要添加HikariCP支持，我们首先在pom.xml中添加以下[Hikari](https://mvnrepository.com/artifact/io.ratpack/ratpack-hikari)和[H2](https://mvnrepository.com/artifact/com.h2database/h2) Maven依赖：

```xml
<dependency>
    <groupId>io.ratpack</groupId>
    <artifactId>ratpack-hikari</artifactId>
    <version>${ratpack.version}</version>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>${h2.version}</version>
</dependency>
```

或者，我们可以将以下依赖添加到我们的build.gradle中：

```groovy
dependencies {
    compile ratpack.dependency('hikari')
    compile "com.h2database:h2:$h2.version"
}
```

现在，我们将在连接池的bindings闭包下声明HikariModule：

```groovy
import ratpack.hikari.HikariModule

ratpack {
    bindings {
        module(HikariModule) { config ->
            config.dataSourceClassName = 'org.h2.jdbcx.JdbcDataSource'
            config.addDataSourceProperty('URL', "jdbc:h2:mem:devDB;INIT=RUNSCRIPT FROM 'classpath:/User.sql'")
        }
    }
}
```

最后，我们可以使用Java的Connection和PreparedStatement进行简单的数据库操作：

```groovy
get('fetchUserName/:id') { Context ctx ->
    Connection connection = ctx.get(DataSource.class).getConnection()
    PreparedStatement queryStatement = connection.prepareStatement("SELECT NAME FROM USER WHERE ID=?")
    queryStatement.setInt(1, Integer.parseInt(ctx.getPathTokens().get("id")))
    ResultSet resultSet = queryStatement.executeQuery()
    resultSet.next()
    render resultSet.getString(1)  
}
```

让我们检查处理程序是否按预期工作：

```shell
$ curl -s localhost:5050/fetchUserName/1
Norman Potter
```

### 7.2 Groovy Sql类 

我们可以使用Groovy Sql通过rows和executeInsert等方法快速执行数据库操作：

```groovy
get('fetchUsers') {
    def db = [url:'jdbc:h2:mem:devDB']
    def sql = Sql.newInstance(db.url, db.user, db.password)
    def users = sql.rows("SELECT * FROM USER");
    render(Jackson.json(users))
}
```

```shell
$ curl -s localhost:5050/fetchUsers
[{"ID":1,"TITLE":"Mr","NAME":"Norman Potter","COUNTRY":"USA"},
{"ID":2,"TITLE":"Miss","NAME":"Ketty Smith","COUNTRY":"FRANCE"}]
```

让我们用Sql编写一个HTTP POST示例：

```groovy
post('addUser') {
    parse(Jackson.fromJson(User))
        .then { u ->
            def db = [url:'jdbc:h2:mem:devDB']
            Sql sql = Sql.newInstance(db.url, db.user, db.password)
            sql.executeInsert("INSERT INTO USER VALUES (?,?,?,?)", 
              [u.id, u.title, u.name, u.country])
            render "User $u.name inserted"
        }
}
```

```shell
$ curl -X POST -H 'Content-type: application/json' --data \
'{"id":3,"title":"Mrs","name":"Jiney Weiber","country":"UK"}' \
http://localhost:5050/addUser

User Jiney Weiber inserted
```

## 8. 单元测试

### 8.1 设置测试

如上所述，Ratpack还提供了[ratpack-groovy-test](https://mvnrepository.com/artifact/io.ratpack/ratpack-groovy-test)库来测试ratpack-groovy应用程序。

要使用它，我们可以在我们的pom.xml中添加它作为Maven依赖：

```xml
<dependency>
    <groupId>io.ratpack</groupId>
    <artifactId>ratpack-groovy-test</artifactId>
    <version>1.6.1</version>
</dependency>
```

或者，我们可以在build.gradle中添加Gradle依赖：

```groovy
testCompile ratpack.dependency('groovy-test')
```

然后我们需要创建一个Groovy主类RatpackGroovyApp.groovy来让我们测试Ratpack.groovy脚本。

```groovy
public class RatpackGroovyApp {
    public static void main(String[] args) {
        File file = new File("src/main/groovy/com/tuyucheng/Ratpack.groovy");
        def shell = new GroovyShell()  
        shell.evaluate(file)
    }
}
```

当将Groovy测试作为JUnit测试运行时，该类将**使用GroovyShell调用Ratpack.groovy脚本**。反过来，它将启动Ratpack服务器进行测试。

现在，让我们编写Groovy测试类RatpackGroovySpec.groovy以及通过RatpackGroovyApp启动Ratpack服务器的代码：

```groovy
class RatpackGroovySpec {
    ServerBackedApplicationUnderTest ratpackGroovyApp = new MainClassApplicationUnderTest(RatpackGroovyApp.class)
    @Delegate TestHttpClient client = TestHttpClient.testHttpClient(ratpackGroovyApp)
}
```

Ratpack提供MainClassApplicationUnderTest来Mock启动服务器的应用程序类。

### 8.2 编写测试

让我们编写测试，从一个非常基本的测试开始，检查应用程序是否可以启动：

```groovy
@Test
void "test if app is started"() {
    when:
    get("")

    then:
    assert response.statusCode == 200
    assert response.body.text == "Hello World from Ratpack with Groovy!!"
}
```

现在让我们编写另一个测试来验证fetchUsers获取处理程序的响应：

```groovy
@Test
void "test fetchUsers"() {
    when:
    get("fetchUsers")
        
    then:
    assert response.statusCode == 200
    assert response.body.text == '[{"ID":1,"TITLE":"Mr","NAME":"Norman Potter","COUNTRY":"USA"},{"ID":2,"TITLE":"Miss","NAME":"Ketty Smith","COUNTRY":"FRANCE"}]'
}
```

Ratpack测试框架负责为我们启动和停止服务器。

## 9. 总结

在本文中，我们了解了使用Groovy为Ratpack编写HTTP处理程序的几种方法，我们还探讨了Promises和数据库集成。

我们了解了Groovy闭包、DSL和Groovy Sql如何使我们的代码简洁、高效且易读。同时，Groovy的测试支持使单元测试和集成测试变得简单易用。

利用这些技术，我们可以使用Groovy的动态语言特性和富有表现力的API，通过Ratpack快速开发高性能、可扩展的HTTP应用程序。