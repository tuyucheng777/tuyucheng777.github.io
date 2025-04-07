---
layout: post
title:  Apache Tapestry简介
category: webmodules
copyright: webmodules
excerpt: Apache Tapestry
---

## 1. 概述

如今，从社交网络到银行业务、医疗保健到政府服务，所有活动都可以在线进行。因此，它们严重依赖Web应用程序。

Web应用程序使用户能够使用/享受公司提供的在线服务。同时，它还充当后端软件的接口。

在本入门教程中，我们将探索Apache Tapestry Web框架并使用它提供的基本功能创建一个简单的Web应用程序。

## 2. Apache Tapestry

Apache Tapestry是一个基于组件的框架，用于构建可扩展的Web应用程序。

**它遵循约定优于配置范式**，并使用注解和命名约定进行配置。

所有组件都是简单的POJO，同时，它们是从头开发的，不依赖于其他库。

除了支持Ajax之外，Tapestry还具有出色的异常报告功能。它还提供了丰富的内置通用组件库。

在其他优秀功能中，最突出的是代码的热重载。因此，使用此功能，我们可以在开发环境中立即看到更改。

## 3. 设置

Apache Tapestry需要一组简单的工具来创建Web应用程序：

- Java 1.6或更高版本
- 构建工具(Maven或Gradle)
- IDE(Eclipse或IntelliJ)
- 应用服务器(Tomcat或Jetty)

在本教程中，我们将使用Java 8、Maven、Eclipse和Jetty Server的组合。

要设置[最新](https://tapestry.apache.org/download.html)的Apache Tapestry项目，我们将使用[Maven原型](https://www.baeldung.com/maven-archetype#creating-archetype)并按照官方文档提供的[说明](https://tapestry.apache.org/getting-started.html)进行操作：

```shell
$ mvn archetype:generate -DarchetypeCatalog=http://tapestry.apache.org
```

或者，如果我们有一个现有项目，我们可以简单地将[tapestry-core](https://mvnrepository.com/artifact/org.apache.tapestry/tapestry-core) Maven依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>org.apache.tapestry</groupId>
    <artifactId>tapestry-core</artifactId>
    <version>5.4.5</version>
</dependency>
```

一旦我们完成设置，我们就可以通过以下Maven命令启动应用程序apache-tapestry：

```shell
$ mvn jetty:run
```

默认情况下，可以通过localhost:8080/apache-tapestry访问该应用程序：

![](/assets/images/2025/webmodules/apachetapestry01.png)

## 4. 项目结构

让我们探索一下Apache Tapestry创建的项目布局：

![](/assets/images/2025/webmodules/apachetapestry02.png)

我们可以看到一个类似Maven的项目结构，以及一些基于约定的包。

**Java类位于src/main/java中，并分为components、pages和services**。

同样，src/main/resources保存我们的模板(类似于HTML文件)-这些模板具有.tml扩展名。

**对于放置在components和pages目录下的每个Java类，都应该创建一个同名的模板文件**。

src/main/webapp目录包含图像、样式表和JavaScript文件等资源。同样，测试文件放在src/test中。

最后，src/site将包含文档文件。

为了更好地理解，我们来看看在Eclipse IDE中打开的项目结构：

![](/assets/images/2025/webmodules/apachetapestry03.png)

## 5. 注解

让我们讨论一下Apache Tapestry提供的一些日常使用的方便的[注解](https://tapestry.apache.org/annotations.html)。接下来，我们将在我们的实现中使用这些注解。

### 5.1 @Inject

[@Inject](https://tapestry.apache.org/current/apidocs/org/apache/tapestry5/ioc/annotations/Inject.html)注解在org.apache.tapestry5.ioc.annotations包中可用，并提供了一种在Java类中注入依赖的简便方法。

此注解对于注入资产、块、资源和服务非常方便。

### 5.2 @InjectPage

[@InjectPage](https://tapestry.apache.org/current/apidocs/org/apache/tapestry5/annotations/InjectPage.html)注解位于org.apache.tapestry5.annotations包中，它允许我们将页面注入另一个组件。此外，注入的页面始终是只读属性。

### 5.3 @InjectComponent

类似地，[@InjectComponent](http://tapestry.apache.org/current/apidocs/org/apache/tapestry5/annotations/InjectComponent.html)注解允许我们注入模板中定义的组件。

### 5.4 @Log

[@Log](http://tapestry.apache.org/current/apidocs/org/apache/tapestry5/annotations/Log.html)注解在org.apache.tapestry5.annotations包中可用，可方便地在任何方法上启用DEBUG级别日志记录，它记录方法的进入和退出以及参数值。

### 5.5 @Property

[@Property](https://tapestry.apache.org/current/apidocs/org/apache/tapestry5/annotations/Property.html)注解位于org.apache.tapestry5.annotations包中，它将字段标记为属性。同时，它会自动为属性创建Getter和Setter。

### 5.6 @Parameter

类似地，[@Parameter](https://tapestry.apache.org/current/apidocs/org/apache/tapestry5/annotations/Parameter.html)注解表示字段是组件参数。

## 6. 页面

现在，我们已准备好探索框架的基本功能，让我们在应用中创建一个新的主页。

首先，我们在src/main/java中的pages目录中定义一个Java类Home：

```java
public class Home {
}
```

### 6.1 模板

然后，我们将在src/main/resources下的pages目录中创建相应的Home.tml[模板](http://tapestry.apache.org/component-templates.html)。

扩展名为.tml(Tapestry标记语言)的文件类似于Apache Tapestry提供的带有XML标签的HTML/XHTML文件。

例如，让我们看一下Home.tml模板：

```html
<html xmlns:t="http://tapestry.apache.org/schema/tapestry_5_4.xsd">
    <head>
        <title>apache-tapestry Home</title>
    </head>
    <body>
        <h1>Home</h1>
    </body>   
</html>
```

只需重新启动Jetty服务器，我们就可以在localhost:8080/apache-tapestry/home访问Home页面：

![](/assets/images/2025/webmodules/apachetapestry04.png)

### 6.2 属性

让我们探索如何在Home页面上呈现属性。

为此，我们将在Home类中添加一个属性和一个Getter方法：

```java
@Property
private String appName = "apache-tapestry";

public Date getCurrentTime() {
    return new Date();
}
```

要在Home页面上呈现appName属性，我们只需使用${appName}。

类似地，我们可以编写${currentTime}来从页面访问getCurrentTime方法。

### 6.3 本地化

Apache Tapestry提供集成的[本地化](https://tapestry.apache.org/localization.html)支持。按照惯例，页面名称属性文件保存了要在页面上呈现的所有本地消息的列表。

例如，我们将在Home页面的pages目录中创建一个home.properties文件，并包含本地消息：

```properties
introMsg=Welcome to the Apache Tapestry Tutorial
```

消息属性与Java属性不同。

出于同样的原因，带有消息前缀的键名用于呈现消息属性-例如${message:introMsg}。

### 6.4 布局组件

让我们通过创建Layout.java类来定义一个基本的布局组件，我们将文件保存在src/main/java中的components目录中：

```java
public class Layout {
    @Property
    @Parameter(required = true, defaultPrefix = BindingConstants.LITERAL)
    private String title;
}
```

这里，title属性被标记为required，并且绑定的默认前缀设置为文字字符串。

然后我们在src/main/resources中的components目录下编写一个对应的模板文件Layout.tml：

```html
<html xmlns:t="http://tapestry.apache.org/schema/tapestry_5_4.xsd">
    <head>
        <title>${title}</title>
    </head>
    <body>
        <div class="container">
            <t:body />
            <hr/>
            <footer>
                <p>&copy; Your Company</p>
            </footer>
        </div>
    </body>
</html>
```

现在，让我们在Home页面使用布局：

```html
<html t:type="layout" title="apache-tapestry Home" 
    xmlns:t="http://tapestry.apache.org/schema/tapestry_5_4.xsd">
    <h1>Home! ${appName}</h1>
    <h2>${message:introMsg}</h2>
    <h3>${currentTime}</h3>
</html>
```

注意，[命名空间](http://tapestry.apache.org/schema/tapestry_5_4.xsd)用于标识Apache Tapestry提供的元素(t:type和t:body)。同时，命名空间还提供了组件和属性。

此处，t:type将设置主页上的布局，而t:body元素将插入页面内容。

让我们看一下带有布局的Home页面：

![](/assets/images/2025/webmodules/apachetapestry05.png)

## 7. 表单

让我们创建一个带有表单的登录页面，以允许用户登录。

正如已经探讨过的，我们首先创建一个Java类Login：

```java
public class Login {
    // ...
    @InjectComponent
    private Form login;

    @Property
    private String email;

    @Property
    private String password;
}
```

在这里，我们定义了两个属性-email和password。此外，我们还注入了一个用于登录的Form组件。

然后，我们创建一个相应的模板login.tml：

```html
<html t:type="layout" title="apache-tapestry com.example"
      xmlns:t="http://tapestry.apache.org/schema/tapestry_5_3.xsd"
      xmlns:p="tapestry:parameter">
    <t:form t:id="login">
        <h2>Please sign in</h2>
        <t:textfield t:id="email" placeholder="Email address"/>
        <t:passwordfield t:id="password" placeholder="Password"/>
        <t:submit class="btn btn-large btn-primary" value="Sign in"/>
    </t:form>
</html>
```

现在，我们可以访问localhost:8080/apache-tapestry/login的登录页面：

![](/assets/images/2025/webmodules/apachetapestry06.png)

## 8. 校验

Apache Tapestry提供了一些用于[表单校验](http://tapestry.apache.org/forms-and-validation.html)的内置方法，它还提供了处理表单提交成功或失败的方法。

内置方法遵循事件和组件名称的约定，例如，方法onValidationFromLogin将校验Login组件。

同样，onSuccessFromLogin和onFailureFromLogin等方法分别用于成功和失败事件。

因此，让我们将这些内置方法添加到Login类：

```java
public class Login {
    // ...

    void onValidateFromLogin() {
        if (email == null)
            System.out.println("Email is null);

        if (password == null)
            System.out.println("Password is null);
    }

    Object onSuccessFromLogin() {
        System.out.println("Welcome! Login Successful");
        return Home.class;
    }

    void onFailureFromLogin() {
        System.out.println("Please try again with correct credentials");
    }
}
```

## 9. 警告框

如果没有适当的警告，表单校验就不完整。当然，该框架也内置了对警告消息的支持。

为此，我们首先在Login类中注入[AlertManager](https://tapestry.apache.org/current/apidocs/org/apache/tapestry5/alerts/AlertManager.html)的实例来管理警报。然后，用警报消息替换现有方法中的println语句：

```java
public class Login {
    // ...
    @Inject
    private AlertManager alertManager;

    void onValidateFromLogin() {
        if(email == null || password == null) {
            alertManager.error("Email/Password is null");
            login.recordError("Validation failed"); //submission failure on the form
        }
    }

    Object onSuccessFromLogin() {
        alertManager.success("Welcome! Login Successful");
        return Home.class;
    }

    void onFailureFromLogin() {
        alertManager.error("Please try again with correct credentials");
    }
}
```

让我们看看登录失败时的警报情况：

![](/assets/images/2025/webmodules/apachetapestry07.png)

## 10. Ajax

到目前为止，我们已经探索了如何使用表单创建一个简单的主页。同时，我们还了解了校验和对警报消息的支持。

接下来，让我们探索一下Apache Tapestry对Ajax的内置支持。

首先，我们将在Home类中注入[AjaxResponseRenderer](https://tapestry.apache.org/current/apidocs/org/apache/tapestry5/services/ajax/AjaxResponseRenderer.html)和[Block](http://tapestry.apache.org/current/apidocs/org/apache/tapestry5/Block.html)组件的实例。然后，我们将创建一个方法onCallAjax来处理Ajax调用：

```java
public class Home {
    // ....

    @Inject
    private AjaxResponseRenderer ajaxResponseRenderer;

    @Inject
    private Block ajaxBlock;

    @Log
    void onCallAjax() {
        ajaxResponseRenderer.addRender("ajaxZone", ajaxBlock);
    }
}
```

另外，我们需要在Home.tml中做一些更改。

首先，我们将添加[eventLink](http://tapestry.apache.org/current/apidocs/org/apache/tapestry5/corelib/components/EventLink.html)来调用onCallAjax方法。然后，我们将添加一个id为ajaxZone的[zone](https://tapestry.apache.org/current/apidocs/org/apache/tapestry5/corelib/components/Zone.html)元素来呈现Ajax响应。

最后，我们需要一个block组件，它将被注入到Home类中并呈现为Ajax响应：

```html
<p><t:eventlink event="callAjax" zone="ajaxZone" class="btn btn-default">Call Ajax</t:eventlink></p>
<t:zone t:id="ajaxZone"></t:zone>
<t:block t:id="ajaxBlock">
    <hr/>
    <h2>Rendered through Ajax</h2>
    <p>The current time is: <strong>${currentTime}</strong></p>
</t:block>
```

我们来看看更新后的主页：

![](/assets/images/2025/webmodules/apachetapestry08.png)

然后，我们可以单击“Call Ajax”按钮并查看ajaxResponseRenderer的实际运行：

![](/assets/images/2025/webmodules/apachetapestry09.png)

## 11. 日志记录

要启用内置日志记录功能，需要注入[Logger](http://www.slf4j.org/api/org/slf4j/Logger.html)的实例。然后，我们可以使用它在任何级别(如TRACE、DEBUG和INFO)进行日志记录。

因此，让我们在Home类中进行必要的更改：

```java
public class Home {
    // ...

    @Inject
    private Logger logger;

    void onCallAjax() {
        logger.info("Ajax call");
        ajaxResponseRenderer.addRender("ajaxZone", ajaxBlock);
    }
}
```

现在，当我们点击“Call Ajax”按钮时，记录器将以INFO级别记录日志：

```text
[INFO] pages.Home Ajax call
```

## 12. 总结

在本文中，我们探索了Apache Tapestry Web框架。

首先，我们创建了一个快速启动Web应用程序，并使用Apache Tapestry的基本功能(如组件、页面和模板)添加了主页。

然后，我们研究了Apache Tapestry提供的一些方便的注解，用于配置属性和组件/页面注入。

最后，我们探讨了框架提供的内置Ajax和日志支持。
