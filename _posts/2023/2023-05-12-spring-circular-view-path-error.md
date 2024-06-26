---
layout: post
title:  循环视图路径错误
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 简介

在本教程中，我们将了解如何在Spring MVC应用程序中获取和解决循环视图路径错误。

## 2. 依赖关系

为了演示这一点，让我们创建一个简单的Spring Boot Web项目。首先，我们需要在我们的Maven项目文件中添加[Spring Boot web starter](https://search.maven.org/artifact/org.springframework.boot/spring-boot-starter-web/2.3.0.RELEASE/jar)依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

## 3. 重现问题

然后，让我们创建一个简单的Spring Boot应用程序，其中包含一个解析为一个路径的控制器：

```java
@Controller
public class CircularViewPathController {

    @GetMapping("/path")
    public String path() {
        return "path";
    }
}
```

返回值是将产生响应数据的视图名称，在我们的例子中，返回值是与path.html模板关联的路径：

```html
<html>
<head>
    <title>path.html</title>
</head>
<body>
<p>path.html</p>
</body>
</html>
```

启动服务器后，我们可以通过向http://localhost:8080/path发出GET请求来重现错误，结果将是Circular View Path错误：

```bash
{"timestamp":"2023-01-01T16:47:42.173+0000","status":500,"error":"Internal Server Error",
"message":"Circular view path [path]: would dispatch back to the current handler URL [/path] 
again. Check your ViewResolver setup! (Hint: This may be the result of an unspecified view, 
due to default view name generation.)","path":"/path"}
```

## 4. 解决方案

默认情况下，Spring MVC框架应用[InternalResourceView](https://docs.spring.io/spring/docs/3.0.x/javadoc-api/org/springframework/web/servlet/view/InternalResourceView.html)类作为视图解析器，因此，**如果@GetMapping值与视图名称相同**，请求将失败并出现Circular View path错误。

一种可能的解决方案是重命名视图并更改控制器方法中的返回值。

```java
@Controller
public class CircularViewPathController {
    @GetMapping("/path")
    public String path() {
        return "path2";
    }
}
```

如果我们不想重命名视图并更改控制器方法中的返回值，那么另一种解决方案是为项目选择另一个视图处理器。

对于最常见的情况，我们可以选择[Thymeleaf Java模板引擎](https://www.thymeleaf.org/)，让我们将[spring-boot-starter-thymeleaf](https://search.maven.org/artifact/org.springframework.boot/spring-boot-starter-thymeleaf/2.3.0.RELEASE/jar)依赖添加到项目中：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

重新构建项目后我们再次运行，请求成功。在这种情况下，Thymeleaf替换了InternalResourceView类。

## 5. 总结

在本教程中，我们了解了循环视图路径错误、发生原因以及解决问题的方法。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-boot-modules/spring-boot-mvc-3)上获得。