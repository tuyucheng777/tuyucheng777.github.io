---
layout: post
title:  为所有Spring Boot控制器添加前缀
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 简介

在Spring Boot应用程序中，每个控制器都可以有自己的URL映射，这使得单个应用程序可以轻松地在多个位置提供Web端点。例如，我们可以将API端点分组为内部和外部等逻辑组。

但是，有时我们可能希望所有端点都使用一个通用前缀。**在本教程中，我们将介绍为所有Spring Boot控制器使用通用前缀的不同方法**。

## 2. Servlet上下文

在Spring应用程序中，负责处理Web请求的主要组件是[DispatcherServlet](https://www.baeldung.com/spring-dispatcherservlet)。通过自定义此组件，我们可以对请求的路由方式进行相当大的控制。

让我们看一下定制DispatcherServlet的两种不同方法，这将使我们所有的应用程序端点都可以在一个通用的URL前缀上使用。

### 2.1 Spring Bean

第一种方式是引入一个新的Spring Bean：

```java
@Configuration
public class DispatcherServletCustomConfiguration {

    @Bean
    public DispatcherServlet dispatcherServlet() {
        return new DispatcherServlet();
    }

    @Bean
    public ServletRegistrationBean dispatcherServletRegistration() {
        ServletRegistrationBean registration = new ServletRegistrationBean(dispatcherServlet(), "/api/");
        registration.setName(DispatcherServletAutoConfiguration.DEFAULT_DISPATCHER_SERVLET_REGISTRATION_BEAN_NAME);
        return registration;
    }
}
```

在这里，我们创建一个包装DispatcherServlet Bean的ServletRegistrationBean。请注意，我们提供了一个显式基本URL /api/，**这意味着我们所有的端点都必须在该基础URL前缀处访问**。

### 2.2 应用程序属性

我们也可以仅通过使用应用程序属性来实现相同的结果。在Spring Boot 2.0.0之后的版本中，我们将以下内容添加到application.properties文件中：

```properties
server.servlet.contextPath=/api
```

在此版本之前，属性名称略有不同：

```properties
server.contextPath=/api
```

这种方法的一个好处是它只使用普通的Spring属性，**这意味着我们可以使用Profile或外部属性绑定等标准机制轻松更改或覆盖我们的通用前缀**。

### 2.3 优点和缺点

这两种方法的主要优点也是主要缺点：它们会影响应用程序中的每个端点。

对于某些应用程序来说，这可能完全没问题。但是，某些应用程序可能需要使用标准端点映射来与第三方服务进行交互-例如OAuth交换。在这些情况下，像这样的全局解决方案可能不太合适。

## 3. 注解

另一种方法是使用注解来为Spring应用程序中的所有控制器添加前缀。下面，我们将介绍两种不同的方法。

### 3.1 SpEL

第一种方法是使用带有标准@RequestMapping注解的[Spring Expression Language](https://www.baeldung.com/spring-expression-language)(SpEL)。使用这种方法，我们只需向每个想要添加前缀的控制器添加一个属性：

```java
@Controller
@RequestMapping(path = "${apiPrefix}/users")
public class UserController {
} 
```

然后，我们只需在application.properties中指定属性值：

```properties
apiPrefix=/api
```

### 3.2 自定义注解

实现此目的的另一种方法是创建我们自己的注解：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
@RequestMapping("/api/")
public @interface ApiPrefixController {
    @AliasFor(annotation = Component.class)
    String value() default "";
}
```

然后，我们只需要将注解应用到我们想要添加前缀的每个控制器上：

```java
@Controller
@ApiPrefixController
public class SomeController {
    @RequestMapping("/users")
    @ReponseBody
    public String getAll(){
        // ...
    }
}
```

### 3.3 优点和缺点

这两种方法解决了前一种方法的主要问题：**它们都提供了对哪些控制器获取前缀的细粒度控制**。我们可以仅将注解应用于特定的控制器，而不是影响应用程序中的所有端点。

## 4. 服务器端转发

我们将要研究的最后一种方法是使用[服务器端转发](https://www.baeldung.com/spring-redirect-and-forward)。**与重定向不同，转发不涉及返回客户端的响应**，这意味着我们的应用程序可以在端点之间传递请求而不会影响客户端。

首先，让我们编写一个带有两个端点的简单控制器：

```java
@Controller
class EndpointController {
    @GetMapping("/endpoint1")
    @ResponseBody
    public String endpoint1() {
        return "Hello from endpoint 1";
    }

    @GetMapping("/endpoint2")
    @ResponseBody
    public String endpoint2() {
        return "Hello from endpoint 2";
    }
}
```

接下来，我们根据想要的前缀创建一个新的控制器：

```java
@Controller
@RequestMapping("/api/endpoint")
public class ApiPrefixController {

    @GetMapping
    public ModelAndView route(ModelMap model) {
        if(new Random().nextBoolean()) {
            return new ModelAndView("forward:/endpoint1", model);
        }
        else {
            return new ModelAndView("forward:/endpoint2", model);
        }
    }
}
```

**该控制器有一个充当路由器的端点**，在这种情况下，它本质上是抛硬币，将原始请求转发到我们的另外两个端点之一。

我们可以通过发送几个连续的请求来验证它是否正常工作：

```shell
> curl http://localhost:8080/api/endpoint
Hello from endpoint 2
> curl http://localhost:8080/api/endpoint
Hello from endpoint 1
> curl http://localhost:8080/api/endpoint
Hello from endpoint 1
> curl http://localhost:8080/api/endpoint
Hello from endpoint 2
> curl http://localhost:8080/api/endpoint
Hello from endpoint 2
```

这种方法的主要好处是功能非常强大，我们可以应用任何我们想要的逻辑来确定如何转发请求：URL路径、HTTP方法、HTTP标头等等。

## 5. 总结

在本文中，我们了解了几种将通用前缀应用于Spring应用程序中的每个控制器的方法。与大多数决策一样，每种方法都有其优缺点，在实施之前应仔细考虑。