---
layout: post
title:  Struts 2快速介绍
category: webmodules
copyright: webmodules
excerpt: Struts
---

## 1. 简介

[Apache Struts 2](https://struts.apache.org/)是一个基于MVC的框架，用于开发企业Java Web应用程序，它是对原始Struts框架的完全重写，它具有开源API实现和丰富的功能集。

**在本教程中，我们将为初学者介绍Struts2框架的不同核心组件**。此外，我们还将展示如何使用它们。

## 2. Struts 2框架概述

Struts 2的一些特性如下：

- 基于POJO(普通旧Java对象)的操作
- 插件支持REST、AJAX、Hibernate、Spring等
- 约定优于配置
- 支持多种视图层技术
- 易于分析和调试

### 2.1 Struts 2的不同组件

Struts2是一个基于MVC的框架，因此所有Struts2应用程序中都会存在以下三个组件：

1. Action类：它是一个POJO类(POJO意味着它不属于任何类型层次结构，可以用作独立类)，在这里实现我们的业务逻辑
2. 控制器：在Struts2中，HTTP过滤器用作控制器；它们主要执行拦截和验证请求/响应等任务
3. 视图：用于呈现处理后的数据；通常是一个JSP文件

## 3. 设计我们的应用程序

让我们继续开发我们的Web应用程序，用户可以选择特定的汽车品牌并收到定制消息。

### 3.1 Maven依赖

让我们在pom.xml中添加以下条目：

```xml
<dependency>
    <groupId>org.apache.struts</groupId>
    <artifactId>struts2-core</artifactId>
    <version>2.5.10</version>
</dependency>
<dependency>
    <groupId>org.apache.struts</groupId>
    <artifactId>struts2-junit-plugin</artifactId>
    <version>2.5.10</version>
</dependency>
<dependency>
    <groupId>org.apache.struts</groupId>
    <artifactId>struts2-convention-plugin</artifactId>
    <version>2.5.10</version>
</dependency>
```

依赖的最新版本可以在[这里](https://mvnrepository.com/artifact/org.apache.struts)找到。

### 3.2 业务逻辑

让我们创建一个动作类CarAction，它返回特定输入值的消息。CarAction有两个字段-carName(用于存储来自用户的输入)和carMessage(用于存储要显示的自定义消息)：

```java
public class CarAction {

    private String carName;
    private String carMessage;
    private CarMessageService carMessageService = new CarMessageService();

    public String execute() {
        this.setCarMessage(this.carMessageService.getMessage(carName));
        return "success";
    }

    // getters and setters
}
```

CarAction类使用CarMessageService为Car品牌提供定制消息：

```java
public class CarMessageService {

    public String getMessage(String carName) {
        if (carName.equalsIgnoreCase("ferrari")){
            return "Ferrari Fan!";
        }
        else if (carName.equalsIgnoreCase("bmw")){
            return "BMW Fan!";
        }
        else {
            return "please choose ferrari Or bmw";
        }
    }
}
```

### 3.3 接收用户输入

让我们添加一个JSP，它是我们应用程序的入口点，这是input.jsp文件的内容：

```html
<body>
    <form method="get" action="/struts2/tutorial/car.action">
        <p>Welcome to Tuyucheng Struts 2 app</p>
        <p>Which car do you like !!</p>
        <p>Please choose ferrari or bmw</p>
        <select name="carName">
            <option value="Ferrari" label="ferrari" />
            <option value="BMW" label="bmw" />
         </select>
        <input type="submit" value="Enter!" />
    </form>
</body>
```

<form\>标签指定操作(在我们的例子中，它是必须发送GET请求的HTTP URI)。

### 3.4 控制器部分

StrutsPrepareAndExecuteFilter是控制器，它将拦截所有传入的请求，我们需要在web.xml中注册以下过滤器：

```xml
<filter>
    <filter-name>struts2</filter-name>
    <filter-class>org.apache.struts2.dispatcher.filter.StrutsPrepareAndExecuteFilter</filter-class>
</filter>

<filter-mapping>
    <filter-name>struts2</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

StrutsPrepareAndExecuteFilter将过滤每个传入请求，因为我们指定了与通配符匹配的URL <url-pattern>/*</url-pattern>

### 3.5 配置应用程序

让我们向CarAction添加以下注解：

```java
@Namespace("/tutorial")
@Action("/car")
@Result(name = "success", location = "/result.jsp")
```

让我们了解一下这个注解的逻辑，@Namespace用于对不同操作类的请求URI进行逻辑分离；我们需要在请求中包含这个值。

此外，@Action告诉请求URI的实际端点，该端点将命中我们的Action类。该Action类访问CarMessageService并初始化另一个成员变量carMessage的值，在execute()方法返回一个值(在本例中为“success”)后，它会匹配该值以调用result.jsp。

最后，@Result有两个参数，第一个参数name指定我们的Action类将返回的值；该值是从Action类的execute()方法返回的，**这是将要执行的默认方法名称**。

第二部分location表示在execute()方法返回值后要引用哪个文件，这里，我们指定当execute()返回值为“success”的字符串时，我们必须将请求转发到result.jsp。

通过提供XML配置文件可以实现相同的配置：

```xml
<struts>
    <package name="tutorial" extends="struts-default" namespace="/tutorial">
        <action name="car" class="cn.tuyucheng.taketoday.struts.CarAction" method="execute">
            <result name="success">/result.jsp</result>
        </action>
    </package>
</struts>
```

### 3.6 视图

这是result.jsp的内容，将用于向用户呈现消息：

```html
<%@ page contentType="text/html; charset=UTF-8" %>
<%@ taglib prefix="s" uri="/struts-tags" %>
<body>
    <p> Hello Tuyucheng User </p>
    <p>You are a <s:property value="carMessage"/></p>
</body>
```

这里有两件重要的事情需要注意：

- 在<@taglib prefix="s" uri="/struts-tags %\>中我们导入struts-tags库
- 在<s:property value="carMessage"/\>中，我们使用struts-tags库来打印属性carMessage的值

## 4. 运行应用程序

此Web应用程序可以在任何Web容器中运行，例如Apache Tomcat，以下是完成此操作所需的步骤：

1. 部署Web应用程序后，打开浏览器并访问以下URL：http://www.localhost.com:8080/MyStrutsApp/input.jsp
2. 选择两个选项之一并提交请求
3. 将被转发到result.jsp页面，其中包含根据所选输入选项定制的消息

## 5. 总结

在本教程中，我们逐步介绍了如何创建第一个Struts2 Web应用程序，我们介绍了Struts2领域中与MVC相关的不同方面，并展示了如何将它们组合在一起进行开发。