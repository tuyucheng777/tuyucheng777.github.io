---
layout: post
title:  使用Servlet和JSP的MVC示例
category: webmodules
copyright: webmodules
excerpt: Jakarta EE
---

## 1. 概述

在这篇简短的文章中，我们将使用基本的Servlet和JSP创建一个实现模型-视图-控制器(MVC)设计模式的小型Web应用程序。

在进行实现之前，我们将稍微探讨一下MVC的工作原理及其主要特性。

## 2. MVC简介

模型-视图-控制器(MVC)是软件工程中用于将应用程序逻辑与用户界面分离的一种模式，顾名思义，MVC模式有三层。

**模型定义应用程序的业务层，控制器管理应用程序的流程，视图定义应用程序的表示层**。

尽管MVC模式并非特定于Web应用程序，但它非常适合此类应用程序。在Java上下文中，模型由简单的Java类组成，控制器由Servlet组成，视图由JSP页面组成。

以下是该模式的一些主要特征：

- 它将表示层与业务层分开
- 控制器执行调用模型并向视图发送数据的操作
- 模型甚至不知道它被某个Web应用程序或桌面应用程序使用

### 2.1 模型层

这是包含系统业务逻辑的数据层，也代表应用程序的状态。

它独立于表示层，控制器从模型层获取数据并将其发送到视图层。

### 2.2 控制层

控制器层充当视图和模型之间的接口，它接收来自视图层的请求并进行处理，包括必要的验证。

请求进一步发送到模型层进行数据处理，处理完成后，数据被发送回控制器，然后显示在视图上。

### 2.3 视图层

这一层代表应用程序的输出，通常是某种形式的UI，表示层用于显示由控制器获取的模型数据。

## 3. 使用Servlet和JSP的MVC

为了实现基于MVC设计模式的Web应用程序，我们将创建Student和StudentService类-它们将充当我们的模型层。

StudentServlet类将充当控制器，对于表示层，我们将创建student-record.jsp页面。

现在，让我们逐一编写这些层，并从Student类开始：

```java
public class Student {
    private int id;
    private String firstName;
    private String lastName;
	
    // constructors, getters and setters goes here
}
```

现在让我们编写处理业务逻辑的StudentService：

```java
public class StudentService {

    public Optional<Student> getStudent(int id) {
        switch (id) {
            case 1:
                return Optional.of(new Student(1, "John", "Doe"));
            case 2:
                return Optional.of(new Student(2, "Jane", "Goodall"));
            case 3:
                return Optional.of(new Student(3, "Max", "Born"));
            default:
                return Optional.empty();
        }
    }
}
```

现在让我们创建控制器类StudentServlet：

```java
@WebServlet(
        name = "StudentServlet",
        urlPatterns = "/student-record")
public class StudentServlet extends HttpServlet {

    private StudentService studentService = new StudentService();

    private void processRequest(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String studentID = request.getParameter("id");
        if (studentID != null) {
            int id = Integer.parseInt(studentID);
            studentService.getStudent(id)
                    .ifPresent(s -> request.setAttribute("studentRecord", s));
        }

        RequestDispatcher dispatcher = request.getRequestDispatcher(
                "/WEB-INF/jsp/student-record.jsp");
        dispatcher.forward(request, response);
    }

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        processRequest(request, response);
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        processRequest(request, response);
    }
}
```

这个Servlet是我们的Web应用程序的控制器。

首先从请求中读取一个参数id，如果提交了这个id，就从业务层获取一个Student对象。

一旦它从模型中检索到必要的数据，它就会使用setAttribute()方法将该数据放入请求中。

最后，控制器将请求和响应对象转发给JSP，即应用程序的视图。

接下来，让我们编写我们的表示层student-record.jsp：

```html
<html>
    <head>
        <title>Student Record</title>
    </head>
    <body>
    <% 
        if (request.getAttribute("studentRecord") != null) {
            Student student = (Student) request.getAttribute("studentRecord");
    %>
 
    <h1>Student Record</h1>
    <div>ID: <%= student.getId()%></div>
    <div>First Name: <%= student.getFirstName()%></div>
    <div>Last Name: <%= student.getLastName()%></div>
        
    <% 
        } else { 
    %>

    <h1>No student record found.</h1>
         
    <% } %>	
    </body>
</html>
```

当然，JSP是应用程序的视图；它从控制器接收所需的所有信息，不需要直接与业务层交互。

## 4. 总结

在本教程中，我们学习了MVC即模型-视图-控制器架构，并重点介绍了如何实现一个简单的示例。