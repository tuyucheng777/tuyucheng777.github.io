---
layout: post
title:  从Servlet返回JSON响应
category: webmodules
copyright: webmodules
excerpt: Jakarta EE
---

## 1. 简介

在本快速教程中，我们将创建一个小型Web应用程序并探讨如何从Servlet返回JSON响应。

## 2. Maven

对于我们的Web应用程序，我们将在pom.xml中包含jakarta.servlet-api和Gson依赖：

```xml
<dependency>
    <groupId>jakarta.servlet</groupId>
    <artifactId>jakarta.servlet-api</artifactId>
    <version>${jakarta.servlet.version}</version>
</dependency>
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>${gson.version}</version>
</dependency>

```

依赖的最新版本可以在这里找到：[jakarta.servlet-api](https://mvnrepository.com/artifact/jakarta.servlet/jakarta.servlet-api)和[gson](https://mvnrepository.com/artifact/com.google.code.gson/gson)。

我们还需要配置一个Servlet容器来部署我们的应用程序，[这篇文章](https://www.baeldung.com/tomcat-deploy-war)是了解如何在Tomcat上部署WAR的好起点。

## 3. 创建实体

让我们创建一个Employee实体，稍后它将以JSON格式从Servlet返回：

```java
public class Employee {
	
    private int id;
    
    private String name;
    
    private String department;
   
    private long salary;

    // constructors
    // standard getters and setters.
}
```

## 4. 实体转JSON

要从Servlet发送JSON响应，我们首先需要**将Employee对象转换为其JSON表示形式**。

有许多Java库可用于将对象转换为JSON表示形式，其中最突出的是Gson和Jackson库。要了解GSON和Jackson之间的区别，请查看[本文](https://www.baeldung.com/jackson-vs-gson)。

使用Gson将对象转换为JSON表示的一个快速示例是：

```java
String employeeJsonString = new Gson().toJson(employee);
```

## 5. 响应和内容类型

对于HTTP Servlet，填充响应的正确过程是：

1. 从响应中检索输出流
2. 填写响应头
3. 将内容写入输出流
4. 提交响应

在响应中，Content-Type标头告诉客户端返回内容的实际内容类型。

为了生成JSON响应，内容类型应该是application/json：

```java
PrintWriter out = response.getWriter();
response.setContentType("application/json");
response.setCharacterEncoding("UTF-8");
out.print(employeeJsonString);
out.flush();
```

**响应标头必须始终在提交响应之前设置，Web容器将忽略提交响应后设置或添加标头的任何尝试**。

在PrintWriter上调用flush()提交响应。

## 6. 示例Servlet

现在让我们看一个返回JSON响应的示例Servlet：

```java
@WebServlet(name = "EmployeeServlet", urlPatterns = "/employeeServlet")
public class EmployeeServlet extends HttpServlet {

    private Gson gson = new Gson();

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        Employee employee = new Employee(1, "Karan", "IT", 5000);
        String employeeJsonString = this.gson.toJson(employee);

        PrintWriter out = response.getWriter();
        response.setContentType("application/json");
        response.setCharacterEncoding("UTF-8");
        out.print(employeeJsonString);
        out.flush();   
    }
}
```

## 7. 总结

本文展示了如何从Servlet返回JSON响应，这对于使用Servlet实现REST服务的Web应用程序很有帮助。