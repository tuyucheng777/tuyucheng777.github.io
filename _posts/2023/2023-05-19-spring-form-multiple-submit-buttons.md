---
layout: post
title:  表单上的多个提交按钮
category: springweb
copyright: springweb
excerpt: Spring Web
---

## 1. 概述

在本快速教程中，我们将以[Spring MVC中的表单入门](https://www.baeldung.com/spring-mvc-form-tutorial)为基础，向JSP表单添加一个按钮，映射到相同的URI。

## 2. 简短回顾

早些时候，我们创建了一个小型Web应用程序来输入员工的详细信息并将它们保存在内存中。

首先，我们编写了一个模型[Employee](https://www.baeldung.com/spring-mvc-form-tutorial#the-model)来绑定实体，然后是一个[EmployeeController](https://www.baeldung.com/spring-mvc-form-tutorial#the-controller)来处理流程和映射，最后，一个名为[employeeHome](https://www.baeldung.com/spring-mvc-form-tutorial#the-view)的视图描述了用户键入输入值的表单。

这个表单有一个按钮Submit，它映射到控制器的[RequestMapping](https://www.baeldung.com/spring-requestmapping)调用addEmployee以使用模型将用户输入的详细信息添加到内存数据库。

在接下来的几节中，我们将看到如何将另一个按钮Cancel添加到控制器中具有相同RequestMapping路径的相同表单。

## 3. 表单

首先，让我们向表单employeeHome.jsp添加一个新按钮：

```html
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form"%>
...
<body>
<h3>Welcome, Enter The Employee Details</h3>
<h4>${message}</h4>
<form:form method="POST" action="${pageContext.request.contextPath}/addEmployee"
           modelAttribute="employee">
    <table>
        ...
        <tr>
            <td><input type="submit" name="submit" value="Submit"/></td>
            <td><input type="submit" name="cancel" value="Cancel"/></td>
        </tr>
        ...
```

如我们所见，我们向现有的提交按钮添加了一个属性名称，并添加了另一个名称设置为cancel的取消按钮。

我们还在页面顶部添加了一条模型属性消息，如果单击取消，该消息将显示。

## 4. 控制器

接下来，让我们修改[控制器](https://www.baeldung.com/spring-controller-vs-restcontroller)，为RequestMapping添加一个新的属性参数，以区分两次按钮点击：

```java
@RequestMapping(value = "/addEmployee", method = RequestMethod.POST, params = "submit")
public String submit(@Valid @ModelAttribute("employee") final Employee employee, final BindingResult result, final ModelMap model) {
    // same code as before
}

@RequestMapping(value = "/addEmployee", method = RequestMethod.POST, params = "cancel")
public String cancel(@Valid @ModelAttribute("employee") final Employee employee, final BindingResult result, final ModelMap model) {
    model.addAttribute("message", "You clicked cancel, please re-enter employee details:");
    return "employeeHome";
}
```

在这里，我们在现有方法submit中添加了一个新参数params。值得注意的是，它的值与表单中指定的名称相同。

然后我们添加了另一个具有类似签名的方法cancel，唯一的区别是参数params指定为cancel。和以前一样，该值与JSP表单中取消按钮的名称完全相同。

## 5. 测试

为了进行测试，我们将在Tomcat等Web容器上部署该项目。

在访问URL http://localhost:8080/spring-mvc-forms/employee 时，我们将看到：

![](/assets/images/2023/springweb/springformmultiplesubmitbuttons01.png)

点击Cancel后，我们将看到：

![](/assets/images/2023/springweb/springformmultiplesubmitbuttons02.png)

在这里，我们看到了我们在控制器的方法cancel中编写的消息。

点击Submit，我们会看到[像以前一样](https://www.baeldung.com/spring-mvc-form-tutorial#testing-the-application)输入的员工信息：

![](/assets/images/2023/springweb/springformmultiplesubmitbuttons03.png)

## 6. 总结

在本教程中，我们学习了如何将另一个按钮添加到[Spring MVC](https://www.baeldung.com/category/spring-mvc/)应用程序中的相同表单，该表单映射到控制器上的相同RequestMapping。

如果需要，我们可以使用代码片段中演示的相同技术添加更多按钮。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-web-modules)上获得。