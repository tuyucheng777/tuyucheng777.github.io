---
layout: post
title:  Jakarta EE MVC/Eclipse Krazo简介
category: webmodules
copyright: webmodules
excerpt: Jakarta EE
---

## 1. 简介

模型-视图-控制器(MVC)是构建Web应用程序的流行设计模式，多年来，它一直是构建现代Web应用程序的实际设计原则。

在本教程中，让我们学习如何使用带有网页和REST API的Jakarta EE MVC 2.0构建Web应用程序。

## 2. JSR-371

**Jakarta MVC 2.0(以前称为[JSR 371](https://jcp.org/en/jsr/detail?id=371) MVC 1.0)是一个基于[Jakarta RESTful Web Services](https://jakarta.ee/specifications/restful-ws/)或JAX-RS(以前称为[RESTful Web Services的Java API](https://www.baeldung.com/jax-rs-spec-and-implementations))构建的基于操作的Web框架**，JSR-371通过附加注解对JAX-RS进行了补充，使构建Web应用程序更加方便。

JSR 371或Jakarta MVC规范了我们用Java开发Web应用程序的方式；此外，主要目标是利用现有的[CDI(上下文和依赖注入)](https://jcp.org/en/jsr/detail?id=346)和[Bean Validation](https://jcp.org/en/jsr/detail?id=349)，并支持[JSP](https://www.baeldung.com/jsp)和[Facelets](https://javaee.github.io/tutorial/jsf-facelets.html)作为视图技术。

目前，Jakarta MVC 2.1规范工作正在进行中，可能会与Jakarta EE 10一起发布。

## 3. JSR-371注解

JSR-371除了JAX-RS注解之外还定义了一些注解，**所有这些注解都是jakarta.mvc.\*包的一部分**。

### 3.1 jakarta.mvc.Controller

@Controller注解将资源标记为MVC控制器，当用于类时，类中的所有资源方法都将成为控制器。同样，在资源方法上使用此注解会使该方法成为控制器。通常，如果我们想在同一个类中定义MVC控制器和REST API，在方法上定义@Controller会很有帮助。

例如，我们定义一个控制器：

```java
@Path("user")
public class UserController {
    @GET
    @Produces("text/html")
    @Controller
    public String showUserForm(){
        return "user.jsp";
    }
    @GET
    @Produces("application/json")
    public String getUserDetails(){
        return getUserDetails();
    }
}
```

此类有一个@Controller，用于呈现用户表单(showUserForm)和一个返回用户详细信息JSON(getUserDetails)的REST API。

### 3.2 jakarta.mvc.View

与@Controller类似，我们可以使用@View注解标记资源类或资源方法。通常，返回void的资源方法应该具有@View。带有@View的类表示具有void类型的类中控制器的默认视图。

例如，让我们用@View定义一个控制器：

```java
@Controller
@Path("user")
@View("defaultModal.jsp")
public class UserController {
    @GET
    @Path("void")
    @View("userForm.jsp")
    @Produces("text/html")
    public void showForm() {
        getInitFormData();
    }

    @GET
    @Path("string")
    @Produces("text/html")
    public void showModal() {
        getModalData();
    }
}
```

这里，资源类和资源方法都有@View注解，控制器showForm呈现视图userForm.jsp。类似地，showModal控制器呈现在资源类上定义的defaultModal.jsp。

### 3.3 jakarta.mvc.binding.MvcBinding

Jakarta RESTful Webservices拒绝存在绑定和验证错误的请求，类似的设置可能不适用于与网页交互的用户。幸运的是，即使发生绑定和验证错误，Jakarta MVC也会调用控制器。通常，用户应该充分了解数据绑定错误。

控制器注入[BindingResult](https://jakarta.ee/specifications/mvc/2.0/apidocs/jakarta/mvc/binding/bindingresult)以向用户呈现人性化验证和绑定错误消息。例如，让我们定义一个带有@MvcBinding的控制器：

```java
@Controller
@Path("user")
public class UserController {
    @MvcBinding
    @FormParam("age")
    @Min(18)
    private int age;
    @Inject
    private BindingResult bindingResult;
    @Inject
    private Models models;
    @POST
    public String processForm() {
        if (bindingResult.isFailed()) {
            models.put("errors", bindingResult.getAllMessages());
            return "user.jsp";
        }
    }
}
```

这里，如果用户输入的age小于18岁，用户将被送回包含绑定错误的同一页面。user.jsp页面使用表达式语言(EL)，可以检索请求属性errors并将其显示在页面上。

### 3.4 jakarta.mvc.RedirectScoped

考虑一个表单，用户在其中填写并提交数据(HTTP POST)，服务器处理数据并将用户重定向到成功页面(HTTP GET)，这种模式被广泛称为[PRG(Post-Redirect-Get)模式](https://en.wikipedia.org/wiki/Post/Redirect/Get)。在某些情况下，我们喜欢在POST和GET之间保存数据；在这些情况下，模型/Bean的范围超出了单个请求。

当一个Bean被@RedirectScoped标注时，该Bean的状态超出了单个请求的范围。然而，在POST、重定向和Get完成后，该状态将被销毁。用@RedirectScoped划定的Bean在POST、重定向和GET完成后将被销毁。

例如，假设Bean User有注解@RedirectScoped：

```java
@RedirectScoped
public class User {
    private String id;
    private String name;
    // getters and setters
}
```

接下来，将该Bean注入到控制器中：

```java
@Controller
@Path("user")
public class UserController {
    @Inject
    private User user;
    @POST
    public String post() {
        user.setName("John Doe");
        return "redirect:/submit";
    }
    @GET
    public String get() {
        return "success.jsp";
    }
}
```

此处，Bean User可用于POST以及后续的重定向和GET。因此，success.jsp可以使用EL访问Bean的name属性。

### 3.5 jakarta.mvc.UriRef

我们只能对资源方法使用@UriRef注解，@UriRef使我们能够为资源方法提供名称，我们可以使用这些名称在视图中调用我们的控制器，而不是使用控制器路径URI。

假设有一个带有href的用户表单：

```html
<a href="/app/user">Click Here</a>
```

单击“Click Here”将调用映射到GET /app/user的控制器。

```java
@GET
@UriRef("user-details")
public String getUserDetails(String userId) {
    userService.getUserDetails(userId);
}
```

在这里，我们用user-details命名我们的控制器。现在，我们可以在视图中引用此名称，而不是URI：

```html
<a href="${mvc.uri('user-details')}">Click Here</a>
```

### 3.6 jakarta.mvc.security.CsrfProtected

此注解强制要求调用资源方法时必须进行CSRF验证，如果CSRF令牌无效，客户端将收到ForbiddenException(HTTP 403)异常。只有资源方法可以具有此注解。

考虑一个控制器：

```java
@POST
@Path("user")
@CsrfProtected
public String saveUser(User user) {
    service.saveUser(user);
}
```

鉴于控制器具有@CsrfProtected注解，只有当请求包含有效的CSRF令牌时，请求才会到达控制器。

## 4. 构建MVC应用程序

接下来，让我们使用REST API和控制器构建一个Web应用程序。最后，让我们在最新版本的[Eclipse Glassfish](https://projects.eclipse.org/projects/ee4j.glassfish)中部署我们的Web应用程序。

### 4.1 生成项目

首先，我们使用Maven archetype:generate生成Jakarta MVC 2.0项目：

```shell
mvn archetype:generate 
  -DarchetypeGroupId=org.eclipse.krazo
  -DarchetypeArtifactId=krazo-jakartaee9-archetype
  -DarchetypeVersion=2.0.0 -DgroupId=cn.tuyucheng.taketoday
  -DartifactId=krazo -DkrazoImpl=jersey
```

上述原型生成一个具有所需工件的maven项目，类似于：

![](/assets/images/2025/webmodules/javaeemvceclipsekrazo01.png)

此外，生成的pom.xml包含[jakarta.platform](https://mvnrepository.com/search?q=jakarta.jakartaee-web-api)、[jakarta.mvc](https://mvnrepository.com/search?q=jakarta.mvc)和[org.eclipse.krazo](https://mvnrepository.com/artifact/org.eclipse.krazo/krazo-jersey)依赖：

```xml
<dependency>
    <groupId>jakarta.platform</groupId>
    <artifactId>jakarta.jakartaee-web-api</artifactId>
    <version>9.1.0</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>jakarta.mvc</groupId>
    <artifactId>jakarta.mvc-api</artifactId>
    <version>2.0.0</version>
</dependency>
<dependency>
    <groupId>org.eclipse.krazo</groupId>
    <artifactId>krazo-jersey</artifactId>
    <version>2.0.0</version>
</dependency>
```

### 4.2 控制器

接下来，让我们定义用于显示表单、保存用户详细信息的控制器以及用于获取用户详细信息的API。但首先，让我们定义我们的应用程序路径：

```java
@ApplicationPath("/app")
public class UserApplication extends Application {
}
```

应用程序路径定义为/app，接下来，让我们定义将用户转发到用户详细信息表单的控制器：

```java
@Path("users")
public class UserController {
    @GET
    @Controller
    public String showForm() {
        return "user.jsp";
    }
}
```

接下来，在WEB-INF/views下，我们可以创建一个视图user.jsp，并构建和部署应用程序：

```shell
mvn clean install glassfish:deploy
```

此[Glassfish Maven](https://mvnrepository.com/artifact/org.glassfish.maven.plugin/maven-glassfish-plugin)插件在端口8080上构建、部署并运行。成功部署后，我们可以打开浏览器并点击URL：

http://localhost:8080/mvc-2.0/app/users：


![](/assets/images/2025/webmodules/javaeemvceclipsekrazo02.png)

接下来，让我们定义一个处理表单提交操作的HTTP POST：

```java
@POST
@Controller
public String saveUser(@Valid @BeanParam User user) {   
    return "redirect:users/success";
}
```

现在，当用户单击“Create”按钮时，控制器将处理POST请求并将用户重定向到成功页面：

![](/assets/images/2025/webmodules/javaeemvceclipsekrazo03.png)

让我们利用Jakarta Validations、CDI和@MvcBinding来提供表单校验：

```java
@Named("user")
public class User implements Serializable {

    @MvcBinding
    @Null
    private String id;

    @MvcBinding
    @NotNull
    @Size(min = 1, message = "Name cannot be blank")
    @FormParam("name")
    private String name;
    // other validations with getters and setters 
}
```

一旦我们有了表单校验，我们就可以检查绑定错误。如果有任何绑定错误，我们必须向用户显示验证消息。为此，让我们注入BindingResult来处理无效的表单参数；让我们更新saveUser方法：

```java
@Inject
private BindingResult bindingResult;

public String saveUser(@Valid @BeanParam User user) {
    if (bindingResult.isFailed()) {
        models.put("errors", bindingResult.getAllErrors());
        return "user.jsp";
    }  
    return "redirect:users/success";
}
```

经过验证后，如果用户提交的表单没有包含必填参数，我们会显示验证错误：

![](/assets/images/2025/webmodules/javaeemvceclipsekrazo04.png)

接下来，让我们使用@CsrfProtected保护我们的POST方法免受CSRF攻击，将@CsrfProtected添加到方法saveUser：

```java
@POST
@Controller
@CsrfProtected
public String saveUser(@Valid @BeanParam User user) {
}
```

接下来我们尝试点击Create按钮：

![](/assets/images/2025/webmodules/javaeemvceclipsekrazo05.png)

当控制器受到保护以免受CSRF攻击时，客户端应始终传递CSRF令牌。因此，让我们在user.jsp中添加一个隐藏字段，该字段在每个请求上添加一个CSRF令牌：

```html
<input type="hidden" name="${mvc.csrf.name}" value="${mvc.csrf.token}"/>
```

同样的，现在我们来开发一个[REST API](https://www.baeldung.com/jax-rs-spec-and-implementations)：

```java
@GET
@Produces(MediaType.APPLICATION_JSON)
public List<User> getUsers() {
    return users;
}
```

此HTTP GET API返回用户列表。

## 5. 总结

在本文中，我们了解了Jakarta MVC 2.0以及如何使用Eclipse Krazo开发Web应用程序和REST API。