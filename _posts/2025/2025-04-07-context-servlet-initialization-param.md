---
layout: post
title:  上下文和Servlet初始化参数
category: webmodules
copyright: webmodules
excerpt: Jakarta EE
---

## 1. 概述

[Servlet](https://en.wikipedia.org/wiki/Java_Servlet)是在Servlet容器中运行的普通Java类。

HTTP Servlet(一种特定类型的Servlet)是Java Web应用程序中的头等公民，**HTTP Servlet的API旨在通过典型的请求-处理-响应循环来处理HTTP请求，该循环在客户端-服务器协议中实现**。

此外，Servlet可以使用请求/响应参数形式的键值对来控制客户端(通常是Web浏览器)和服务器之间的交互。

这些参数可以初始化并绑定到应用程序范围的作用域(上下文参数)和Servlet特定作用域(Servlet参数)。

在本教程中，**我们将学习如何定义和访问上下文和Servlet初始化参数**。

## 2. 初始化Servlet参数

我们可以使用注解和标准部署描述符-“web.xml”文件来定义和初始化Servlet参数，值得注意的是，这两个选项并不互相排斥。

让我们深入探讨每一个选项。

### 2.1 使用注解

**使用注解初始化Servlet参数允许我们将配置和源代码保存在同一个地方**。

在本节中，我们将演示如何使用注解定义和访问绑定到特定Servlet的初始化参数。

为此，我们将实现一个基本的UserServlet类，通过纯HTML表单收集用户数据。

首先，让我们看一下呈现表单的JSP文件：

```html
<!DOCTYPE html>
<html>
<head>
    <title>Context and Initialization Servlet Parameters</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
</head>
<body>
<h2>Please fill the form below:</h2>
<form action="${pageContext.request.contextPath}/userServlet" method="post">
    <label for="name"><strong>Name:</strong></label>
    <input type="text" name="name" id="name">
    <label for="email"><strong>Email:</strong></label>
    <input type="text" name="email" id="email">
    <input type="submit" value="Send">
</form>
</body>
</html>
```

请注意，我们使用[EL(表达式语言)](https://mvnrepository.com/artifact/javax.el/el-api/2.2)对表单的action属性进行了编码。这样，无论应用程序文件在服务器中的位置如何，它都可以始终指向“/userServlet”路径。

**“${pageContext.request.contextPath}”表达式为表单设置一个动态URL，该URL始终相对于应用程序的上下文路径**。

以下是我们的初始Servlet实现：

```java
@WebServlet(name = "UserServlet", urlPatterns = "/userServlet", initParams={
        @WebInitParam(name="name", value="Not provided"),
        @WebInitParam(name="email", value="Not provided")})
public class UserServlet extends HttpServlet {
    // ...    

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response)
            throws ServletException, IOException {
        processRequest(request, response);
        forwardRequest(request, response, "/WEB-INF/jsp/result.jsp");
    }

    protected void processRequest(
            HttpServletRequest request,
            HttpServletResponse response)
            throws ServletException, IOException {

        request.setAttribute("name", getRequestParameter(request, "name"));
        request.setAttribute("email", getRequestParameter(request, "email"));
    }

    protected void forwardRequest(
            HttpServletRequest request,
            HttpServletResponse response,
            String path)
            throws ServletException, IOException {
        request.getRequestDispatcher(path).forward(request, response);
    }

    protected String getRequestParameter(
            HttpServletRequest request,
            String name) {
        String param = request.getParameter(name);
        return !param.isEmpty() ? param : getInitParameter(name);
    }
}
```

在本例中，我们**使用initParams和@WebInitParam注解**定义了两个Servlet初始化参数，name和email。

请注意，我们使用了HttpServletRequest的getParameter()方法从HTML表单中检索数据，并使用getInitParameter()方法来访问Servlet初始化参数。

getRequestParameter()方法检查name和email请求参数是否为空字符串。

如果它们是空字符串，那么它们会被赋予匹配的初始化参数的默认值。

doPost()方法首先检索用户在HTML表单中输入的姓名和电子邮件(如果有)，然后，它处理请求参数并将请求转发到“result.jsp”文件：

```html
<!DOCTYPE html>
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>User Data</title>
    </head>
    <body>
        <h2>User Information</h2>
        <p><strong>Name:</strong> ${name}</p>
        <p><strong>Email:</strong> ${email}</p>
    </body>
</html>
```

如果我们将示例Web应用程序部署到应用程序服务器(例如[Apache Tomcat](http://tomcat.apache.org/)、[Oracle GlassFish](http://www.oracle.com/technetwork/middleware/glassfish/overview/index.html)或[JBoss WidlFly](http://www.wildfly.org/))并运行它，它应该首先显示HTML表单页面。

一旦用户填写了姓名和电子邮件字段并提交表单，它将输出数据：

```text
User Information
Name: the user's name
Email: the user's email 
```

如果表单只是空白，它将显示Servlet初始化参数：

```text
User Information 
Name: Not provided 
Email: Not provided
```

在这个例子中，**我们展示了如何使用注解定义Servlet初始化参数，以及如何使用getInitParameter()方法访问它们**。

### 2.2 使用标准部署描述符

**这种方法与使用注解的方法不同，因为它允许我们保持配置和源代码彼此隔离**。

为了展示如何使用“web.xml”文件定义初始化Servlet参数，我们首先从UserServlet类中删除initParam和@WebInitParam注解：

```java
@WebServlet(name = "UserServlet", urlPatterns = {"/userServlet"}) 
public class UserServlet extends HttpServlet { ... }
```

接下来，我们在“web.xml”文件中定义Servlet初始化参数：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app
        xmlns="http://xmlns.jcp.org/xml/ns/javaee"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd" version="3.1">
    <servlet>
        <display-name>UserServlet</display-name>
        <servlet-name>UserServlet</servlet-name>
        <init-param>
            <param-name>name</param-name>
            <param-value>Not provided</param-value>
        </init-param>
        <init-param>
            <param-name>email</param-name>
            <param-value>Not provided</param-value>
        </init-param>
    </servlet>
</web-app>
```

如上所示，使用“web.xml”文件定义Servlet初始化参数实际上就是使用<init-param\>、<param-name\>和<param-value\>标签。

此外，只要我们遵循上述标准结构，就可以根据需要定义任意数量的Servlet参数。

当我们将应用程序重新部署到服务器并重新运行它时，它的行为应该与使用注解的版本相同。

## 3. 初始化上下文参数

有时我们需要定义一些不可变的数据，这些数据必须在Web应用程序中全局共享和访问。

由于数据的全局性，**我们应该使用应用程序作用域的上下文初始化参数来存储数据，而不是诉诸于Servlet对应部分**。

尽管无法使用注解定义上下文初始化参数，但我们可以在“web.xml”文件中执行此操作。

假设我们想要为应用程序运行所在的国家和省份提供一些默认的全局值。

我们可以使用几个上下文参数来实现这一点。

让我们相应地重构“web.xml”文件：

```xml
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd" version="3.1">
    <context-param>
        <param-name>province</param-name>
        <param-value>Mendoza</param-value>
    </context-param>
    <context-param>
        <param-name>country</param-name>
        <param-value>Argentina</param-value>
    </context-param>
    <!-- Servlet initialization parameters -->
</web-app>
```

这次，我们使用了<context-param\>、<param-name\>和<param-value\>标签来定义province和country上下文参数。

当然，我们需要重构UserServlet类，以便它可以获取这些参数并将它们传递给结果页面。

以下是Servlet的相关部分：

```java
@WebServlet(name = "UserServlet", urlPatterns = {"/userServlet"})
public class UserServlet extends HttpServlet {
    // ...

    protected void processRequest(
            HttpServletRequest request,
            HttpServletResponse response)
            throws ServletException, IOException {

        request.setAttribute("name", getRequestParameter(request, "name"));
        request.setAttribute("email", getRequestParameter(request, "email"));
        request.setAttribute("province", getContextParameter("province"));
        request.setAttribute("country", getContextParameter("country"));
    }

    protected String getContextParameter(String name) {-
        return getServletContext().getInitParameter(name);
    }
}
```

请注意getContextParameter()方法的实现，它**首先通过getServletContext()获取Servlet上下文，然后使用getInitParameter()方法获取上下文参数**。

接下来，我们需要重构“result.jsp”文件，以便它可以显示上下文参数以及Servlet特定的参数：

```html
<p><strong>Name:</strong> ${name}</p>
<p><strong>Email:</strong> ${email}</p>
<p><strong>Province:</strong> ${province}</p>
<p><strong>Country:</strong> ${country}</p>
```

最后，我们可以重新部署应用程序并再次执行它。

如果用户在HTML表单中填写姓名和电子邮件，那么它将与上下文参数一起显示这些数据：

```text
User Information 
Name: the user's name 
Email: the user's email
Province: Mendoza
Country: Argentina
```

否则，它将输出Servlet和上下文初始化参数：

```text
User Information 
Name: Not provided 
Email: Not provided
Province: Mendoza 
Country: Argentina
```

虽然这个例子是人为的，但它展示了**如何使用上下文初始化参数来存储不可变的全局数据**。

由于数据绑定到应用程序上下文而不是特定的Servlet，我们可以使用getServletContext()和getInitParameter()方法从一个或多个Servlet访问它们。

## 4. 总结

在本文中，**我们学习了上下文和Servlet初始化参数的关键概念以及如何使用注解和“web.xml”文件定义和访问它们**。

很长一段时间以来，Java中一直存在着一种强烈的趋势，即尽可能摆脱XML配置文件并迁移到注解。

举几个例子，[CDI](https://docs.oracle.com/javaee/6/tutorial/doc/giwhl.html)、[Spring](https://spring.io/)、[Hibernate](http://hibernate.org/)就是这方面的明显例子。

尽管如此，使用“web.xml”文件来定义上下文和Servlet初始化参数本质上没有任何问题。

尽管[Servlet API](https://mvnrepository.com/search?q=Servlet-api)已经朝着这个趋势快速发展，但我们仍然需要使用部署描述符来定义上下文初始化参数。
