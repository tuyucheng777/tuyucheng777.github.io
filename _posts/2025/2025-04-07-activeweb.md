---
layout: post
title:  ActiveWeb简介
category: webmodules
copyright: webmodules
excerpt: ActiveWeb
---

## 1. 概述

在本文中，我们将介绍[Activeweb](http://javalite.io/activeweb)(JavaLite推出的全栈Web框架)，它提供了开发动态Web应用程序或RESTful Web服务所需的一切。

## 2. 基本概念和原则

Activeweb利用“约定优于配置”-这意味着它是可配置的，但具有合理的默认值，不需要额外的配置。我们只需要遵循一些预定义的约定，例如以某种预定义的格式命名类、方法和字段。

它还通过重新编译并将源重新加载到正在运行的容器(默认为Jetty)来简化开发。

对于依赖管理，它使用Google Guice作为DI框架；要了解有关Guice的更多信息，请查看[此处](https://www.baeldung.com/guice)的指南。

## 3. Maven设置

首先，我们来添加必要的依赖：

```xml
<dependency>
    <groupId>org.javalite</groupId>
    <artifactId>activeweb</artifactId>
    <version>1.15</version>
</dependency>
```

最新版本可以在[这里](https://mvnrepository.com/artifact/org.javalite/activeweb)找到。

此外，为了测试应用程序，我们需要activeweb-testing依赖：

```xml
<dependency>
    <groupId>org.javalite</groupId>
    <artifactId>activeweb-testing</artifactId>
    <version>1.15</version>
    <scope>test</scope>
</dependency>
```

[此处](https://mvnrepository.com/artifact/org.javalite/activeweb-testing)查看最新版本。

## 4. 应用程序结构

正如我们所讨论的，应用程序结构需要遵循一定的惯例；典型的MVC应用程序如下所示：

![](/assets/images/2025/webmodules/activeweb01.png)

我们可以看到，controllers、service、config和models应该位于app包中它们自己的子包中。

视图应位于WEB-INF/views目录中，每个视图都根据控制器名称拥有自己的子目录。例如，app.controllers.ArticleController应该有一个article/子目录，其中包含该控制器的所有视图文件。

部署描述符或web.xml通常应包含<filter\>和相应的<filter-mapping\>，由于框架是Servlet过滤器，因此有一个过滤器配置，而不是<servlet\>配置：

```xml
<!--...-->
<filter>
    <filter-name>dispatcher</filter-name>
    <filter-class>org.javalite.activeweb.RequestDispatcher</filter-class>
<!--...-->
</filter>
<!--...-->
```

我们还需要一个<init-param\>root_controller来定义应用程序的默认控制器-类似于主控制器：

```xml
<!--...-->
<init-param>
    <param-name>root_controller</param-name>
    <param-value>home</param-value>
</init-param>
<!--...-->
```

## 5. 控制器

控制器是ActiveWeb应用程序的主要组件；并且，如前所述，所有控制器都应位于app.controllers包内：

```java
public class ArticleController extends AppController {
    // ...
}
```

请注意，控制器扩展org.javalite.activeweb.AppController。

### 5.1 控制器URL映射

控制器会根据约定自动映射到URL，例如，ArticleController将映射到：

```text
http://host:port/contextroot/article
```

现在，这将把它们映射到控制器中的默认操作。操作只不过是控制器内部的方法，将默认方法命名为index()：

```java
public class ArticleController extends AppController {
    // ...
    public void index() {
        render("articles");    
    }
    // ...
}
```

对于其他方法或操作，请将方法名称附加到URL：

```java
public class ArticleController extends AppController {
    // ...
    
    public void search() {
        render("search");
    }
}
```

URL：

```text
http://host:port/contextroot/article/search
```

我们甚至可以基于HTTP方法执行控制器操作，只需使用@POST、@PUT、@DELETE、@GET或@HEAD之一标注该方法即可。如果我们不标注某个操作，则默认情况下它将被视为GET。

### 5.2 控制器URL解析

框架使用控制器名称和子包名称来生成控制器URL，例如app.controllers.ArticleController.java的URL：

```text
http://host:port/contextroot/article
```

如果控制器位于子包内，则URL简单地变为：

```text
http://host:port/contextroot/tuyucheng/article
```

对于包含多个单词的控制器名称(例如app.controllers.PublishedArticleController.java)，URL将使用下划线分隔：

```text
http://host:port/contextroot/published_article
```

### 5.3 获取请求参数

在控制器内部，我们使用AppController类中的param()或params()方法访问请求参数。第一个方法接收一个String参数-要检索的参数的名称：

```java
public void search() {

    String keyword = param("key");  
    view("search",articleService.search(keyword));
}
```

如果需要的话，我们可以使用后者来获取所有参数：

```java
public void search() {
        
    Map<String, String[]> criterion = params();
    // ...
}
```

## 6. 视图

在ActiveWeb术语中，视图通常被称为模板；这主要是因为它使用[Apache FreeMarker](https://freemarker.apache.org/)模板引擎而不是JSP。

将模板放在WEB-INF/views目录中，每个控制器都应该有一个同名的子目录，用于存放其所需的所有模板。

### 6.1 控制器视图映射

当控制器被访问时，将执行默认操作index()，并且框架将从views目录中为该控制器选择WEB-INF/views/article/index.ftl模板。同样，对于任何其他操作，将根据操作名称选择视图。

这并不总是我们想要的，有时我们可能希望根据内部业务逻辑返回一些视图。在这种情况下，**我们可以使用父类org.javalite.activeweb.AppController中的render()方法控制该过程**：

```java
public void index() {
    render("articles");    
}
```

请注意，自定义视图的位置也应位于该控制器的同一视图目录中。如果不是这种情况，请在模板名称前加上模板所在目录的名称作为前缀，并将其传递给render()方法：

```java
render("/common/error");
```

### 6.3 带有数据的视图

为了将数据发送到视图，org.javalite.activeweb.AppController提供了view()方法：

```java
view("articles", articleService.getArticles());
```

这需要两个参数，首先是用于访问模板中的对象的对象名称，其次是包含数据的对象。

我们还可以使用assign()方法将数据传递给视图，view()和assign()方法之间没有任何区别-我们可以选择其中任何一个：

```java
assign("article", articleService.search(keyword));
```

让我们映射模板中的数据：

```html
<@content for="title">Articles</@content>
...
<#list articles as article>
    <tr>
        <td>${article.title}</td>
        <td>${article.author}</td>
        <td>${article.words}</td>
        <td>${article.date}</td>
    </tr>
</#list>
</table>
```

## 7. 管理依赖关系

为了管理对象和实例，ActiveWeb使用Google Guice作为依赖管理框架。

假设我们的应用程序中需要一个服务类；这会将业务逻辑与控制器分开。

我们首先创建一个服务接口：

```java
public interface ArticleService {
    
    List<Article> getArticles();   
    Article search(String keyword);
}
```

实现如下：

```java
public class ArticleServiceImpl implements ArticleService {

    public List<Article> getArticles() {
        return fetchArticles();
    }

    public Article search(String keyword) {
        Article ar = new Article();
        ar.set("title", "Article with "+keyword);
        ar.set("author", "tuyucheng");
        ar.set("words", "1250");
        ar.setDate("date", Instant.now());
        return ar;
    }
}
```

现在，让我们将这个服务绑定为Guice模块：

```java
public class ArticleServiceModule extends AbstractModule {

    @Override
    protected void configure() {
        bind(ArticleService.class).to(ArticleServiceImpl.class)
            .asEagerSingleton();
    }
}
```

最后，根据需要在应用程序上下文中注册它并将其注入到控制器中：

```java
public class AppBootstrap extends Bootstrap {

    public void init(AppContext context) {
    }

    public Injector getInjector() {
        return Guice.createInjector(new ArticleServiceModule());
    }
}
```

注意，这个配置类名必须是AppBootstrap，并且位于app.config包中。

最后，下面是我们将其注入控制器的方法：

```java
@Inject
private ArticleService articleService;
```

## 8. 测试

ActiveWeb应用程序的单元测试是使用JavaLite中的[JSpec](http://javalite.io/jspec)库编写的。

我们将使用JSpec中的org.javalite.activeweb.ControllerSpec类来测试我们的控制器，并且我们将按照类似的约定命名测试类：

```java
public class ArticleControllerSpec extends ControllerSpec {
    // ...
}
```

请注意，该名称与其正在测试的控制器相似，末尾带有“Spec”。

这是测试用例：

```java
@Test
public void whenReturnedArticlesThenCorrect() {
    request().get("index");
    a(responseContent())
        .shouldContain("<td>Introduction to Mule</td>");
}
```

请注意，request()方法模拟对控制器的调用，相应的HTTP方法get()将操作名称作为参数。

我们还可以使用params()方法将参数传递给控制器：

```java
@Test
public void givenKeywordWhenFoundArticleThenCorrect() {
    request().param("key", "Java").get("search");
    a(responseContent())
        .shouldContain("<td>Article with Java</td>");
}
```

为了传递多个参数，我们也可以使用此流式的API链接方法。

## 9. 部署应用程序

可以将应用程序部署在任何servlet容器中，例如Tomcat、WildFly或Jetty。当然，最简单的部署和测试方法是使用Maven Jetty插件：

```xml
<!--...-->
<plugin>
    <groupId>org.eclipse.jetty</groupId>
    <artifactId>jetty-maven-plugin</artifactId>
    <version>9.4.8.v20171121</version>
    <configuration>
        <reload>manual</reload>
        <scanIntervalSeconds>10000</scanIntervalSeconds>
    </configuration>
</plugin>
<!--...-->
```

该插件的最新版本在[这里](https://mvnrepository.com/artifact/org.eclipse.jetty/jetty-maven-plugin)可以找到。

现在，我们可以启动它了：

```shell
mvn jetty:run
```

## 10. 总结

在本文中，我们了解了ActiveWeb框架的基本概念和约定。除此之外，该框架还具有比我们在此处讨论的更多的功能和配置。

请参阅官方[文档](http://javalite.io/documentation#activeweb)了解更多详细信息。