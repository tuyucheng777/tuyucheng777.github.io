---
layout: post
title:  Jersey MVC支持
category: webmodules
copyright: webmodules
excerpt: webmodules
---

## 1. 概述

**[Jersey](https://jersey.github.io/)是一个用于开发RESTFul Web服务的开源框架**。

除了作为JAX-RS参考实现之外，它还包括许多扩展，以进一步简化Web应用程序开发。

在本教程中，**我们将创建一个使用Jersey提供的模型-视图-控制器(MVC)扩展的小型示例应用程序**。

要了解如何使用Jersey创建API，请查看[此处](https://www.baeldung.com/jersey-rest-api-with-spring)的文章。

## 2. Jersey MVC

**Jersey包含一个扩展来支持模型-视图-控制器(MVC)设计模式**。

首先，在Jersey组件上下文中，MVC模式中的Controller对应于资源类或方法。

同样，View对应于绑定到资源类或方法的模板。最后，模型表示从资源方法(Controller)返回的Java对象。

**为了在我们的应用程序中使用Jersey MVC的功能，我们首先需要注册我们希望使用的MVC模块扩展**。

在示例中，我们将使用流行的Java模板引擎[Freemarker](https://freemarker.apache.org/)。这是Jersey开箱即用的渲染引擎之一，与[Mustache](https://github.com/spullara/mustache.java)和标准Java Server Pages(JSP)一起支持。

## 3. 应用程序设置

在本节中，我们将首先在pom.xml中配置必要的Maven依赖。

然后，我们将了解如何使用简单的嵌入式[Grizzly](https://javaee.github.io/grizzly/)服务器配置和运行我们的服务器。

### 3.1 Maven依赖

让我们首先添加Jersey MVC Freemarker扩展。

可以从[Maven Central](https://mvnrepository.com/artifact/org.glassfish.jersey.ext/jersey-mvc-freemarker)获取最新版本：

```xml
<dependency>
    <groupId>org.glassfish.jersey.ext</groupId>
    <artifactId>jersey-mvc-freemarker</artifactId>
    <version>3.1.1</version>
</dependency>
```

我们还需要Grizzly Servlet容器。

可以在[Maven Central](https://mvnrepository.com/artifact/org.glassfish.jersey.containers/jersey-container-grizzly2-servlet)中找到最新版本：

```xml
<dependency>
    <groupId>org.glassfish.jersey.containers</groupId>
    <artifactId>jersey-container-grizzly2-servlet</artifactId>
    <version>3.1.1</version>
</dependency>
```

### 3.2 配置服务器

为了在我们的应用程序中使用Jersey MVC模板支持，**我们需要注册MVC模块提供的特定JAX-RS功能**。

考虑到这一点，我们定义一个自定义资源配置：

```java
public class ViewApplicationConfig extends ResourceConfig {    
    public ViewApplicationConfig() {
        packages("cn.tuyucheng.taketoday.jersey.server");
        property(FreemarkerMvcFeature.TEMPLATE_BASE_PATH, "templates/freemarker");
        register(FreemarkerMvcFeature.class);
    }
}
```

在上面的例子中我们配置了3项：

- 首先，我们使用packages方法告诉Jersey扫描cn.tuyucheng.taketoday.jersey.server包，查找用@Path标注的类，这将注册我们的FruitResource
- 接下来，我们配置基本路径以解析模板，这告诉Jersey在/src/main/resources/templates/freemarker中查找Freemarker模板
- 最后，我们通过FreemarkerMvcFeature类注册处理Freemarker渲染的功能 

### 3.3 运行应用程序

现在让我们看看如何运行Web应用程序，我们将使用[exec-maven-plugin](https://www.mojohaus.org/exec-maven-plugin/)配置pom.xml来执行我们的嵌入式Web服务器：

```xml
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>exec-maven-plugin</artifactId>
    <configuration>                
        <mainClass>cn.tuyucheng.taketoday.jersey.server.http.EmbeddedHttpServer</mainClass>
    </configuration>
</plugin>
```

现在让我们使用Maven编译并运行应用程序：

```shell
mvn clean compile exec:java
...
Jul 28, 2018 6:21:08 PM org.glassfish.grizzly.http.server.HttpServer start
INFO: [HttpServer] Started.
Application started.
Try out http://localhost:8082/fruit
Stop the application using CTRL+C
```

转到浏览器URL-http://localhost:8080/fruit，可以看到显示了“Welcome Fruit Index Page!”。

## 4. MVC模板

**在Jersey中，MVC API由两个类组成，用于将模型绑定到视图，即Viewable和@Template**。 

在本节中，我们将解释将模板链接到视图的三种不同方法：

- 使用Viewable类
- 使用@Template注解
- 使用MVC处理错误并将其传递给特定模板

### 4.1 在资源类中使用Viewable

我们先来看一下Viewable：

```java
@Path("/fruit")
public class FruitResource {
    @GET
    public Viewable get() {
        return new Viewable("/index.ftl", "Fruit Index Page");
    }
}
```

**在这个例子中，FruitResource JAX-RS资源类是控制器**；Viewable实例封装了引用的数据模型，它是一个简单的字符串。

此外，我们还包括对相关视图模板的命名引用-index.ftl。

### 4.2 在资源方法上使用@Template

**每次我们想要将模型绑定到模板时，没有必要使用Viewable**。

在下一个示例中，我们将简单地用@Template标注我们的资源方法：

```java
@GET
@Template(name = "/all.ftl")
@Path("/all")
@Produces(MediaType.TEXT_HTML)
public Map<String, Object> getAllFruit() {
    List<Fruit> fruits = new ArrayList<>();
    fruits.add(new Fruit("banana", "yellow"));
    fruits.add(new Fruit("apple", "red"));
    fruits.add(new Fruit("kiwi", "green"));

    Map<String, Object> model = new HashMap<>();
    model.put("items", fruits);
    return model;
}
```

在此示例中，我们使用了@Template注解，这避免了通过Viewable将模型直接包装在模板引用中，并使我们的资源方法更具可读性。

该模型现在由我们标注的资源方法的返回值表示-Map<String, Object\>，它直接传递给模板all.ftl，该模板仅显示我们的水果列表。

### 4.3 使用MVC处理错误

现在让我们看看如何使用@ErrorTemplate注解处理错误：

```java
@GET
@ErrorTemplate(name = "/error.ftl")
@Template(name = "/named.ftl")
@Path("{name}")
@Produces(MediaType.TEXT_HTML)
public String getFruitByName(@PathParam("name") String name) {
    if (!"banana".equalsIgnoreCase(name)) {
        throw new IllegalArgumentException("Fruit not found: " + name);
    }
    return name;
}
```

**一般来说，@ErrorTemplate注解的目的是将模型绑定到错误视图**，当在处理请求期间抛出异常时，此错误处理程序将负责呈现响应。

在我们简单的Fruit API示例中，如果处理过程中没有发生错误，则使用named.ftl模板呈现页面。否则，如果出现异常，则向用户显示error.ftl模板。

**在这种情况下，模型就是抛出的异常本身**，这意味着我们可以从模板内部直接调用异常对象上的方法。

让我们快速浏览一下error.ftl模板中的一段代码来强调这一点：

```html
<body>
    <h1>Error - ${model.message}!</h1>
</body>
```

在最后的例子中，我们将看一个简单的单元测试：

```java
@Test
public void givenGetFruitByName_whenFruitUnknown_thenErrorTemplateInvoked() {
    String response = target("/fruit/orange").request()
        .get(String.class);
    assertThat(response, containsString("Error -  Fruit not found: orange!"));
}
```

在上面的例子中，我们使用了水果资源的响应，检查响应是否包含抛出的IllegalArgumentException消息。

## 5. 总结

在本文中，我们探讨了Jersey框架MVC扩展。

我们首先介绍了Jersey中的MVC工作原理；接下来，我们了解了如何配置、运行和设置示例Web应用程序。

最后，我们研究了使用Jersey和Freemarker的MVC模板的3种方法以及如何处理错误。