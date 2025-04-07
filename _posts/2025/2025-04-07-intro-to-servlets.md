---
layout: post
title:  Java Servlet简介
category: webmodules
copyright: webmodules
excerpt: Jakarta EE
---

## 1. 概述

在本文中，我们将了解Java Web开发的一个核心方面-Servlet。

## 2. Servlet和容器

简单地说，Servlet是一个处理请求、处理请求并返回响应的类。

例如，我们可以使用Servlet通过HTML表单收集用户的输入、从数据库查询记录以及动态创建网页。

Servlet受另一个Java应用程序(称为**Servlet容器**)的控制，当Web服务器中运行的应用程序收到请求时，服务器会将请求传递给Servlet容器，后者再将其传递给目标Servlet。

## 3. Maven依赖

为了在我们的Web应用中添加Servlet支持，需要jakarta.servlet-api依赖：

```xml
<dependency>
    <groupId>jakarta.servlet</groupId>
    <artifactId>jakarta.servlet-api</artifactId>
    <version>6.1.0</version>
</dependency>
```

最新的maven依赖可以在[这里](https://mvnrepository.com/artifact/jakarta.servlet/jakarta.servlet-api)找到。

当然，我们还必须配置一个Servlet容器来部署我们的应用程序；[这是在Tomcat上部署WAR的一个好起点](https://www.baeldung.com/tomcat-deploy-war)。

## 4. Servlet生命周期

让我们来看看定义Servlet生命周期的一组方法。

### 4.1 init() 

init方法设计为仅调用一次，如果不存在Servlet实例，则Web容器：

1. 加载Servlet类
2. 创建Servlet类的实例
3. 通过调用init方法进行初始化

init方法必须成功完成，Servlet才能接收任何请求。如果init方法抛出ServletException或未在Web服务器定义的时间段内返回，Servlet容器将无法将Servlet投入使用。

```java
public void init() throws ServletException {
    // Initialization code like set up database etc....
}
```

### 4.2 service() 

仅当Servlet的init()方法成功完成后才会调用此方法。

容器调用service()方法来处理来自客户端的请求，解释HTTP请求类型(GET、POST、PUT、DELETE等)并根据需要调用doGet、doPost、doPut、doDelete等方法。

```java
public void service(ServletRequest request, ServletResponse response) throws ServletException, IOException {
    // ...
}
```

### 4.3 destroy() 

由Servlet容器调用以使Servlet停止服务。

该方法仅在Servlet的service方法中的所有线程都退出或超时后才会调用，容器调用此方法后，将不会在Servlet上再次调用服务方法。

```java
public void destroy() {
    // 
}
```

## 5. 示例Servlet

首先，将[上下文根](https://www.baeldung.com/tomcat-root-application)从jakarta-servlets-1.0-SNAPSHOT更改为/：

```xml
<Context path="/" docBase="jakarta-servlets-1.0-SNAPSHOT"></Context>
```

在$CATALINA_HOME\conf\server.xml中的Host标签下。

现在让我们构建一个使用表单处理信息的完整示例。

首先，让我们定义一个带有映射/calculateServlet的Servlet，它将捕获表单POST的信息并使用[RequestDispatcher](http://docs.oracle.com/javaee/6/api/javax/servlet/RequestDispatcher.html)返回结果：

```java
@WebServlet(name = "FormServlet", urlPatterns = "/calculateServlet")
public class FormServlet extends HttpServlet {

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String height = request.getParameter("height");
        String weight = request.getParameter("weight");

        try {
            double bmi = calculateBMI(
                    Double.parseDouble(weight),
                    Double.parseDouble(height));

            request.setAttribute("bmi", bmi);
            response.setHeader("Test", "Success");
            response.setHeader("BMI", String.valueOf(bmi));

            request.getRequestDispatcher("/WEB-INF/jsp/index.jsp").forward(request, response);
        } catch (Exception e) {
            request.getRequestDispatcher("/WEB-INF/jsp/index.jsp").forward(request, response);
        }
    }

    private Double calculateBMI(Double weight, Double height) {
        return weight / (height * height);
    }
}
```

如上所示，使用[@WebServlet](http://docs.oracle.com/javaee/6/api/javax/servlet/annotation/WebServlet.html)标注的类必须扩展[jakarta.servlet.http.HttpServlet](https://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpServlet.html)类。需要注意的是，[@WebServlet](http://docs.oracle.com/javaee/6/api/javax/servlet/annotation/WebServlet.html)注解仅在Java EE 6及更高版本中可用。

[@WebServlet](http://docs.oracle.com/javaee/6/api/javax/servlet/annotation/WebServlet.html)注解由容器在部署时处理，并在指定的URL模式中提供相应的Servlet。值得注意的是，通过使用注解来定义URL模式，我们可以避免使用名为web.xml的XML部署描述符进行Servlet映射。

如果我们希望映射不使用注解的Servlet，我们可以使用传统的web.xml：

```xml
<web-app ...>
    <servlet>
       <servlet-name>FormServlet</servlet-name>
       <servlet-class>com.root.FormServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>FormServlet</servlet-name>
        <url-pattern>/calculateServlet</url-pattern>
    </servlet-mapping>
</web-app>
```

接下来，让我们创建一个基本的HTML表单：

```html
<form name="bmiForm" action="calculateServlet" method="POST">
    <table>
        <tr>
            <td>Your Weight (kg) :</td>
            <td><input type="text" name="weight"/></td>
        </tr>
        <tr>
            <td>Your Height (m) :</td>
            <td><input type="text" name="height"/></td>
        </tr>
        <th><input type="submit" value="Submit" name="find"/></th>
        <th><input type="reset" value="Reset" name="reset" /></th>
    </table>
    <h2>${bmi}</h2>
</form>
```

最后- 为了确保一切按预期进行，我们来编写一个快速测试：

```java
public class FormServletLiveTest {

    @Test
    public void whenPostRequestUsingHttpClient_thenCorrect() throws Exception {
        HttpClient client = new DefaultHttpClient();
        HttpPost method = new HttpPost(
                "http://localhost:8080/calculateServlet");

        List<BasicNameValuePair> nvps = new ArrayList<>();
        nvps.add(new BasicNameValuePair("height", String.valueOf(2)));
        nvps.add(new BasicNameValuePair("weight", String.valueOf(80)));

        method.setEntity(new UrlEncodedFormEntity(nvps));
        HttpResponse httpResponse = client.execute(method);

        assertEquals("Success", httpResponse
                .getHeaders("Test")[0].getValue());
        assertEquals("20.0", httpResponse
                .getHeaders("BMI")[0].getValue());
    }
}
```

## 6. Servlet、HttpServlet和JSP

重要的是要理解**Servlet技术不仅限于HTTP协议**。

实际上几乎总是如此，但[Servlet](http://docs.oracle.com/javaee/7/api/javax/servlet/Servlet.html)是一个通用接口，而[HttpServlet](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpServlet.html)是该接口的扩展-添加HTTP特定支持，例如doGet和doPost等。

最后，Servlet技术也是许多其他Web技术(如[JSP–JavaServer Pages](https://www.baeldung.com/jsp)、Spring MVC等)的主要驱动力。

## 7. 总结

在这篇简短的文章中，我们介绍了Java Web应用程序中Servlet的基础知识。
