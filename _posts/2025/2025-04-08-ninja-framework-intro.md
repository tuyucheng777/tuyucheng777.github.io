---
layout: post
title:  Ninja框架简介
category: webmodules
copyright: webmodules
excerpt: Ninja
---

## 1. 概述

如今，有许多基于JEE的框架可用于Web应用程序开发，例如[Spring](https://www.baeldung.com/spring-tutorial)、[Play](https://www.baeldung.com/java-intro-to-the-play-framework)和[Grails](https://www.baeldung.com/grails-gorm-tutorial)。

我们可能有理由选择其中一种，但是，我们的选择还取决于用例和我们要解决的问题。

在本入门教程中，我们将探索Ninja Web框架并创建一个简单的Web应用程序。同时，我们将研究它提供的一些基本功能。

## 2. Ninja

[Ninja](https://www.ninjaframework.org/)是一个全栈但轻量级的Web框架，它利用现有的Java库来完成工作。

它具有从HTML到JSON渲染、持久层到测试的功能，是构建可扩展Web应用程序的一站式解决方案。

它遵循**约定优于配置**范式，并将代码分类到models、controllers和services等包中。

Ninja使用流行的Java库实现主要功能，例如**用于JSON/XML渲染的[Jackson](https://www.baeldung.com/jackson)、用于依赖管理的[Guice](https://www.baeldung.com/guice)、用于持久层的[Hibernate](https://www.baeldung.com/hibernate-5-bootstrapping-api)以及用于数据库迁移的[Flyway](https://www.baeldung.com/database-migrations-with-flyway)**。

为了快速开发，它提供了[SuperDevMode](https://www.ninjaframework.org/documentation/basic_concepts/super_dev_mode.html)来实现代码的热重载；因此，它允许我们在开发环境中即时看到更改。

## 3. 设置

Ninja需要一套标准工具来创建Web应用程序：

- Java 1.8或更高版本
- Maven 3或更高版本
- IDE(Eclipse或IntelliJ)

我们将使用[Maven原型](https://www.baeldung.com/maven-archetype#creating-archetype)来快速设置Ninja项目，它会提示我们提供组ID、工件ID和版本号，然后提供项目名称：

```shell
mvn archetype:generate -DarchetypeGroupId=org.ninjaframework \
  -DarchetypeArtifactId=ninja-servlet-archetype-simple
```

或者，对于现有的Maven项目，我们可以将最新的[ninja-core](https://mvnrepository.com/artifact/org.ninjaframework/ninja-core)依赖添加到pom.xml：

```xml
<dependency>
    <groupId>org.ninjaframework</groupId>
    <artifactId>ninja-core</artifactId>
    <version>6.5.0</version>
</dependency>
```

然后，我们将运行Maven命令来首次编译文件：

```shell
mvn clean install
```

最后，让我们使用Ninja提供的Maven命令运行该应用程序：

```shell
mvn ninja:run
```

应用程序已启动后，可通过localhost:8080访问：

![](/assets/images/2025/webmodules/ninjaframeworkintro01.png)

## 4. 项目结构

我们来看一下Ninja创建的类似Maven的项目结构：

![](/assets/images/2025/webmodules/ninjaframeworkintro02.png)

该框架根据约定创建了一些包。

**Java类分为src/main/java中的conf、controllers、models和services目录**。

同样，src/test/java保存相应的单元测试类。

src/main/java下的views目录包含HTML文件；此外，src/main/java/assets目录包含图像、样式表和JavaScript文件等资源。

## 5. 控制器

我们现在开始讨论该框架的一些基本功能，控制器是一个接收请求并返回具有特定结果的响应的类。

首先，让我们讨论一下要遵循的一些惯例：

- 在controllers包中创建一个类，并在名称后面加上Controller
- 处理请求的方法必须返回[Result](http://www.ninjaframework.org/apidocs/ninja/Result.html)类的对象

让我们创建ApplicationController类并使用简单的方法来呈现HTML：

```java
@Singleton
public class ApplicationController {
    public Result index() {
        return Results.html();
    }
}
```

这里，index方法将通过调用[Results](http://www.ninjaframework.org/apidocs/ninja/Results.html)类的html方法来呈现HTML，Result对象包含呈现内容所需的所有内容，如响应码、标头和cookie。

注意：**Guice的[@Singleton](https://google.github.io/guice/api-docs/latest/javadoc/index.html?com/google/inject/Singleton.html)注解允许整个应用程序中只有一个控制器实例**。

## 6. 视图

对于index方法，Ninja将在views/ApplicationController目录下查找HTML文件-index.ftl.html。

**Ninja使用[Freemarker](https://www.baeldung.com/freemarker-operations)模板引擎进行HTML渲染**，因此，views下的所有文件都应具有.ftl.html扩展名。

让我们为index方法创建index.ftl.html文件：

```html
<html>  
<head>
    <title>Ninja: Index</title>
</head>
<body>
    <h1>${i18n("helloMsg")}</h1>
    <a href="/userJson">User Json</a>
</body>
</html>
```

这里，我们使用了Ninja提供的i18n标签从message.properties文件中获取helloMsg属性，稍后我们将在国际化部分进一步讨论这一点。

## 7. 路由

接下来，我们将定义请求到达index方法的路由。

Ninja使用conf包中的Routes类将URL映射到控制器的特定方法。

让我们添加一个路由来访问ApplicationController的index方法：

```java
public class Routes implements ApplicationRoutes {
    @Override
    public void init(Router router) {          
        router.GET().route("/index").with(ApplicationController::index);
    }
}
```

就这样，我们准备好访问localhost:8080/index处的index页面：

![](/assets/images/2025/webmodules/ninjaframeworkintro03.png)

## 8. JSON渲染

如前所述，Ninja使用Jackson进行JSON渲染。要渲染JSON内容，我们可以使用Results类的json方法。

我们在ApplicationController类中添加userJson方法，并以JSON格式呈现简单HashMap的内容：

```java
public Result userJson() {
    HashMap<String, String> userMap = new HashMap<>();
    userMap.put("name", "Norman Lewis");
    userMap.put("email", "norman@email.com");    
    return Results.json().render(user);
}
```

然后，我们将添加访问userJson所需的路由：

```java
router.GET().route("/userJson").with(ApplicationController::userJson);
```

现在，我们可以使用localhost:8080/userJson呈现JSON：

![](/assets/images/2025/webmodules/ninjaframeworkintro04.png)

## 9. 服务

我们可以创建一个服务，将业务逻辑与控制器分开，并在需要的地方注入我们的服务。

首先，让我们创建一个简单的UserService接口来定义抽象：

```java
public interface UserService {
    HashMap<String, String> getUserMap();
}
```

然后，我们在UserServiceImpl类中实现UserService接口并重写getUserMap方法：

```java
public class UserServiceImpl implements UserService {
    @Override
    public HashMap<String, String> getUserMap() {
        HashMap<String, String> userMap = new HashMap<>();
        userMap.put("name", "Norman Lewis");
        userMap.put("email", "norman@email.com");
        return userMap;
    }
}
```

然后，我们将使用Guice提供的Ninja依赖注入功能将UserService接口与UserServiceImpl类绑定。

让我们在conf包中的Module类中添加绑定：

```java
@Singleton
public class Module extends AbstractModule {
    protected void configure() {
        bind(UserService.class).to(UserServiceImpl.class);
    }
}
```

最后，我们将使用[@Inject](https://google.github.io/guice/api-docs/latest/javadoc/index.html?com/google/inject/Inject.html)注解在ApplicationController类中注入UserService依赖：

```java
public class ApplicationController {
    @Inject
    UserService userService;
    
    // ...
}
```

因此，我们可以在ApplicationController中使用UserService的getUserMap方法：

```java
public Result userJson() {
    HashMap<String, String> userMap = userService.getUserMap();
    return Results.json().render(userMap);
}
```

## 10. Flash Scope

Ninja通过其名为Flash Scope的功能提供了一种简单而有效的方法来处理来自请求的成功和错误消息。

为了在控制器中使用它，我们将FlashScope参数添加到方法中：

```java
public Result showFlashMsg(FlashScope flashScope) {
    flashScope.success("Success message");
    flashScope.error("Error message");
    return Results.redirect("/home");
}
```

注意：Results类的redirect方法将目标重定向到提供的URL。

然后，我们将路由/flash添加到showFlashMsg方法并修改视图以显示flash消息：

```html
<#if (flash.error)??>
    <div class="alert alert-danger">
        ${flash.error}
    </div>
</#if>
<#if (flash.success)??>
    <div class="alert alert-success">
        ${flash.success}
    </div>
</#if>
```

现在，我们可以在localhost:8080/flash上看到FlashScope的运行：

![](/assets/images/2025/webmodules/ninjaframeworkintro05.png)

## 11. 国际化

Ninja提供了易于配置的内置国际化功能。

首先，我们将在application.conf文件中定义支持的语言列表：

```properties
application.languages=fr,en
```

然后，我们将创建默认属性文件-英语的messages.properties，其中包含消息的键值对：

```properties
header.home=Home!
helloMsg=Hello, welcome to Ninja Framework!
```

类似地，我们可以在特定语言的属性文件的文件名中添加语言代码-例如，法语的message_fr.properties文件：

```properties
header.home=Accueil!
helloMsg=Bonjour, bienvenue dans Ninja Framework!
```

配置完成后，我们就可以轻松地在ApplicationController类中启用国际化。

我们有两种方法，要么使用[Lang](https://www.ninjaframework.org/apidocs/ninja/i18n/Lang.html)类，要么使用[Messages](https://www.ninjaframework.org/apidocs/ninja/i18n/Messages.html)类：

```java
@Singleton
public class ApplicationController {
    @Inject
    Lang lang;

    @Inject
    Messages msg;
    
    // ...
}
```

然后，使用Lang类，我们可以设置结果的语言：

```java
Result result = Results.html();
lang.setLanguage("fr", result);
```

类似地，使用Messages类，我们可以获得特定语言的消息：

```java
Optional<String> language = Optional.of("fr");        
String helloMsg = msg.get("helloMsg", language).get();
```

## 12. 持久层

Ninja支持JPA 2.0并利用Hibernate在Web应用程序中实现持久层，此外，它还提供内置的H2数据库支持以实现快速开发。

### 12.1 模型

我们需要一个实体类来连接数据库中的表；为此，Ninja遵循在models包中查找实体类的惯例。因此，我们将在那里创建User实体类：

```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    Long id;
    public String firstName;
    public String email;  
}
```

然后，我们将配置Hibernate并设置数据库连接的详细信息。

### 12.2 配置

对于Hibernate配置，Ninja期望persistence.xml文件位于src/main/java/META-INF目录中：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://java.sun.com/xml/ns/persistence"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://java.sun.com/xml/ns/persistence/persistence_2_0.xsd"
             version="2.0">

    <!-- Database settings for development -->
    <persistence-unit name="dev_unit"
                      transaction-type="RESOURCE_LOCAL">
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
        <properties>
            <property name="hibernate.connection.driver_class" value="org.h2.Driver" />
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect" />
            <property name="hibernate.show_sql" value="true" />
            <property name="hibernate.format_sql" value="true" />
            <property name="hibernate.hbm2ddl.auto" value="update" />
            <property name="hibernate.connection.autocommit" value="true" />
        </properties>
    </persistence-unit>
</persistence>
```

然后，我们将数据库连接详细信息添加到application.conf：

```properties
ninja.jpa.persistence_unit_name=dev_unit
db.connection.url=jdbc:h2:./devDb
db.connection.username=sa
db.connection.password=
```

### 12.3 EntityManager

最后，我们将使用Guice的[Provider](https://google.github.io/guice/api-docs/latest/javadoc/index.html?com/google/inject/Provider.html)类在ApplicationController中注入[EntityManager](https://www.baeldung.com/hibernate-entitymanager)的实例：

```java
public class ApplicationController {
    @Inject 
    Provider<EntityManager> entityManagerProvider;

    // ...
}
```

因此，我们可以使用EntityManager来持久保存User对象：

```java
@Transactional
public Result insertUser(User user) {
    EntityManager entityManager = entityManagerProvider.get();
    entityManager.persist(user);
    entityManager.flush();
    return Results.redirect("/home");
}
```

类似地，我们可以使用EntityManager从DB中读取User对象：

```java
@UnitOfWork
public Result fetchUsers() {
    EntityManager entityManager = entityManagerProvider.get();
    Query q = entityManager.createQuery("SELECT x FROM User x");
    List<User> users = (List<User>) q.getResultList();
    return Results.json().render(users);
}
```

这里，Ninja的[@UnitOfWork](https://www.ninjaframework.org/apidocs/ninja/jpa/UnitOfWork.html)注解将处理有关数据库连接的所有内容，但不包括事务。因此，它对于只读查询非常有用，因为通常我们不需要事务。

## 13. 校验

**Ninja遵循JSR303规范，提供对Bean Validation的内置支持**。

让我们通过使用[@NotNull](https://docs.oracle.com/javaee/7/api/javax/validation/constraints/NotNull.html)注解来标注User实体中的属性来检查该功能：

```java
public class User {
    // ...
    
    @NotNull
    public String firstName;
}
```

然后，我们将修改ApplicationController中已经讨论过的insertUser方法以启用校验：

```java
@Transactional
public Result insertUser(FlashScope flashScope, @JSR303Validation User user, Validation validation) {
    if (validation.getViolations().size() > 0) {
        flashScope.error("Validation Error: User can't be created");
    } else {
        EntityManager entityManager = entitiyManagerProvider.get();
        entityManager.persist(user);
        entityManager.flush();
        flashScope.success("User '" + user + "' is created successfully");
    }
    return Results.redirect("/home");
}
```

我们使用Ninja的[@JSR303Validation](https://www.ninjaframework.org/apidocs/ninja/validation/JSR303Validation.html)注解来启用User对象的校验；然后，我们添加了[Validation](https://www.ninjaframework.org/apidocs/ninja/validation/Validation.html)参数，以便通过hasViolations、getViolations和addViolation等方法进行验证。

最后，使用FlashScope对象在屏幕上显示校验错误。

注意：Ninja遵循JSR303规范来进行Bean校验。但是，JSR380规范([Bean Validation 2.0](https://www.baeldung.com/javax-validation))是新标准。

## 14. 总结

在本文中，我们探讨了Ninja Web框架-一个使用流行Java库提供便捷功能的全栈框架。

首先，我们使用控制器、模型和服务创建了一个简单的Web应用程序。然后，我们在应用程序中启用了JPA支持以实现持久化。

同时，我们看到了一些基本功能，如路由、JSON渲染、国际化和FlashScope。

最后，我们探讨了框架提供的验证支持。