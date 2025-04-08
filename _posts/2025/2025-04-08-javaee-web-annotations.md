---
layout: post
title:  Java EE Web相关注解指南
category: webmodules
copyright: webmodules
excerpt: Jakarta EE
---

## 1. 概述

Java EE注解使开发人员的工作变得更加轻松，因为他们可以指定应用程序组件在容器中的行为方式。这些是XML描述符的现代替代品，基本上可以避免样板代码。

在本文中，我们将重点介绍Java EE 7中Servlet API 3.1引入的注解，我们将研究它们的用途并了解它们的用法。

## 2. Web注解

Servlet API 3.1引入了一组可以在Servlet类中使用的新注解类型：

- @WebServlet
- @WebInitParam
- @WebFilter
- @WebListener
- @ServletSecurity
- @HttpConstraint
- @HttpMethodConstraint
- @MultipartConfig

我们将在下一节中详细讨论它们。

## 3. @WebServlet

简而言之，此注解允许我们将Java类声明为Servlet：

```java
@WebServlet("/account")
public class AccountServlet extends javax.servlet.http.HttpServlet {

    public void doGet(HttpServletRequest request, HttpServletResponse response)
            throws IOException {
        // ...
    }

    public void doPost(HttpServletRequest request, HttpServletResponse response)
            throws IOException {
        // ...
    }
}
```

### 3.1 使用@WebServlet注解的属性

@WebServlet有一组属性允许我们自定义Servlet：

- name
- description
- urlPatterns
- initParams

我们可以按照以下示例使用它们：

```java
@WebServlet(
        name = "BankAccountServlet",
        description = "Represents a Bank Account and it's transactions",
        urlPatterns = {"/account", "/bankAccount" },
        initParams = { @WebInitParam(name = "type", value = "savings")})
public class AccountServlet extends javax.servlet.http.HttpServlet {

    String accountType = null;

    public void init(ServletConfig config) throws ServletException {
        // ...
    }

    public void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        // ... 
    }

    public void doPost(HttpServletRequest request, HttpServletResponse response) throws IOException {
        // ...  
    }
}
```

name属性会覆盖默认的Servlet名称，默认情况下，默认名称是完全限定的类名。如果我们想提供Servlet功能的描述，可以使用description属性。

urlPatterns属性用于指定Servlet可用的URL(如代码示例所示，可以为此属性提供多个值)。

## 4. @WebInitParam

该注解与@WebServlet注解的initParams属性以及Servlet的初始化参数一起使用。

在此示例中，我们将Servlet初始化参数类型设置为“savings”的值：

```java
@WebServlet(
        name = "BankAccountServlet",
        description = "Represents a Bank Account and it's transactions",
        urlPatterns = {"/account", "/bankAccount" },
        initParams = { @WebInitParam(name = "type", value = "savings")})
public class AccountServlet extends javax.servlet.http.HttpServlet {

    String accountType = null;

    public void init(ServletConfig config) throws ServletException {
        accountType = config.getInitParameter("type");
    }

    public void doPost(HttpServletRequest request, HttpServletResponse response) throws IOException {
        // ...
    }
}
```

## 5. @WebFilter

如果我们想改变Servlet的请求和响应而不触及其内部逻辑，我们可以使用WebFilter注解，我们可以通过指定URL模式将过滤器与Servlet或一组Servlet和静态内容关联起来。

在下面的示例中，我们使用@WebFilter注解将任何未经授权的访问重定向到登录页面：

```java
@WebFilter(
        urlPatterns = "/account/*",
        filterName = "LoggingFilter",
        description = "Filter all account transaction URLs")
public class LogInFilter implements javax.servlet.Filter {

    public void init(FilterConfig filterConfig) throws ServletException {
    }

    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest req = (HttpServletRequest) request;
        HttpServletResponse res = (HttpServletResponse) response;

        res.sendRedirect(req.getContextPath() + "/login.jsp");
        chain.doFilter(request, response);
    }

    public void destroy() {
    }
}
```

## 6. @WebListener

如果我们想要了解或控制Servlet及其请求的初始化或更改方式和时间，我们可以使用@WebListener注解。

要编写Web监听器，我们需要扩展以下一个或多个接口：

-ServletContextListener：用于有关ServletContext生命周期的通知
-ServletContextAttributeListener：用于在ServletContext属性发生更改时发出通知
-ServletRequestListener：每当有资源请求时发出通知
-ServletRequestAttributeListener：用于在ServletRequest中添加、删除或更改属性时发出通知
-HttpSessionListener：用于在创建新会话和销毁新会话时发出通知
-HttpSessionAttributeListener：用于在会话中添加或删除新属性时发出通知

下面是如何使用ServletContextListener配置Web应用程序的示例：

```java
@WebListener
public class BankAppServletContextListener implements ServletContextListener {

    public void contextInitialized(ServletContextEvent sce) { 
        sce.getServletContext().setAttribute("ATTR_DEFAULT_LANGUAGE", "english"); 
    } 
    
    public void contextDestroyed(ServletContextEvent sce) { 
        // ... 
    } 
}
```

## 7. @ServletSecurity

当我们想要为我们的Servlet指定安全模型(包括角色、访问控制和身份验证要求)时，我们使用注解@ServletSecurity。

在此示例中，我们将使用@ServletSecurity注解限制对AccountServlet的访问：

```java
@WebServlet(
        name = "BankAccountServlet",
        description = "Represents a Bank Account and it's transactions",
        urlPatterns = {"/account", "/bankAccount" },
        initParams = { @WebInitParam(name = "type", value = "savings")})
@ServletSecurity(
        value = @HttpConstraint(rolesAllowed = {"Member"}),
        httpMethodConstraints = {@HttpMethodConstraint(value = "POST", rolesAllowed = {"Admin"})})
public class AccountServlet extends javax.servlet.http.HttpServlet {

    String accountType = null;

    public void init(ServletConfig config) throws ServletException {
        // ...
    }

    public void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        // ...
    }

    public void doPost(HttpServletRequest request, HttpServletResponse response) throws IOException {
        double accountBalance = 1000d;

        String paramDepositAmt = request.getParameter("dep");
        double depositAmt = Double.parseDouble(paramDepositAmt);
        accountBalance = accountBalance + depositAmt;

        PrintWriter writer = response.getWriter();
        writer.println("<html> Balance of " + accountType + " account is: " + accountBalance
                + "</html>");
        writer.flush();
    }
}
```

在这种情况下，当调用AccountServlet时，浏览器会弹出一个登录屏幕，让用户输入有效的用户名和密码。

我们可以使用@HttpConstraint和@HttpMethodConstraint注解来为@ServletSecurity注解的属性value和httpMethodConstraints指定值。

@HttpConstraint注解适用于所有HTTP方法，换句话说，它指定默认的安全约束。

@HttpConstraint有三个属性：

- value
- rolesAllowed
- transportGuarantee

在这些属性中，最常用的属性是rolesAllowed。在上面的示例代码片段中，属于角色Member的用户被允许调用所有HTTP方法。

@HttpMethodConstraint注解允许我们指定特定HTTP方法的安全约束。

@HttpMethodConstraint具有以下属性：

- value
- emptyRoleSemantic
- rolesAllowed
- transportGuarantee

在上面的示例代码片段中，它展示了如何限制doPost方法仅适用于属于Admin角色的用户，从而允许只有Admin用户才能执行存款功能。

## 8. @MultipartConfig

当我们需要标注一个Servlet来处理multipart/form-data请求(通常用于文件上传Servlet)时，使用此注解。

这将公开HttpServletRequest的getParts()和getPart(name)方法，可用于访问所有部分以及单个部分。

通过调用Part对象的write(fileName)可以将上传的文件写入磁盘。

现在我们看一个示例Servlet UploadCustomerDocumentsServlet来演示其用法：

```java
@WebServlet(urlPatterns = { "/uploadCustDocs" })
@MultipartConfig(
        fileSizeThreshold = 1024 * 1024 * 20,
        maxFileSize = 1024 * 1024 * 20,
        maxRequestSize = 1024 * 1024 * 25,
        location = "./custDocs")
public class UploadCustomerDocumentsServlet extends HttpServlet {

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        for (Part part : request.getParts()) {
            part.write("myFile");
        }
    }
}
```

@MultipartConfig有4个属性：

- fileSizeThreshold：这是临时保存上传文件的大小阈值，如果上传文件的大小大于此阈值，则将存储在磁盘上。否则，文件存储在内存中(大小以字节为单位)
- maxFileSize：这是上传文件的最大大小(以字节为单位)
- maxRequestSize：这是请求的最大大小，包括上传的文件和其他表单数据(以字节为单位)
- location：上传文件的存储目录

## 9. 总结

在本文中，我们研究了Servlet API 3.1中引入的一些Java EE注解及其用途和用法。