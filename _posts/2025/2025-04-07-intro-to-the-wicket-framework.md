---
layout: post
title:  Wicket框架简介
category: webmodules
copyright: webmodules
excerpt: Wicket
---

## 1. 概述

[Wicket](https://wicket.apache.org/)是一个面向Java服务器端Web组件的框架，旨在通过引入桌面UI开发中已知的模式来简化Web界面的构建。

使用Wicket，仅使用Java代码和符合XHTML规范的HTML页面即可构建Web应用程序，无需Javascript或XML配置文件。

它在请求-响应周期上提供了一个层，屏蔽了低层工作并允许开发人员专注于业务逻辑。

在本文中，我们将通过构建HelloWorld Wicket应用程序来介绍基础知识，然后使用两个相互通信的内置组件提供完整的示例。

## 2. 设置

要运行Wicket项目，我们需要添加以下依赖：

```xml
<dependency>
    <groupId>org.apache.wicket</groupId>
    <artifactId>wicket-core</artifactId>
    <version>7.4.0</version>
</dependency>
```

可以在[Maven Central仓库](https://mvnrepository.com/artifact/org.apache.wicket/wicket)找到Wicket的最新版本。

## 3. HelloWorld Wicket

让我们首先对Wicket的WebApplication类进行子类化，这至少需要重写Class<? extends Page\> getHomePage()方法。

Wicket将使用此类作为应用程序的主入口点，在方法内部，只需返回名为HelloWorld的类的class对象：

```java
public class HelloWorldApplication extends WebApplication {
    @Override
    public Class<? extends Page> getHomePage() {
        return HelloWorld.class;
    }
}
```

Wicket更倾向于约定而非配置；向应用程序添加新网页需要创建两个文件：一个Java文件和一个HTML文件，它们的名称相同(但扩展名不同)，位于同一目录下。仅当你想要更改默认行为时才需要进行额外配置。

在源代码的包目录中，首先添加HelloWorld.java：

```java
public class HelloWorld extends WebPage {
    public HelloWorld() {
        add(new Label("hello", "Hello World!"));
    }
}
```

然后是HelloWorld.html：

```html
<html>
    <body>
        <span wicket:id="hello"></span>
    </body>
</html>
```

最后一步，在web.xml中添加过滤器定义：

```xml
<filter>
    <filter-name>wicket.examples</filter-name>
    <filter-class>
      org.apache.wicket.protocol.http.WicketFilter
    </filter-class>
    <init-param>
        <param-name>applicationClassName</param-name>
        <param-value>cn.tuyucheng.taketoday.wicket.examples.HelloWorldApplication</param-value>
    </init-param>
</filter>
```

就这样，我们编写了第一个Wicket Web应用程序。

通过构建war文件(从命令行执行mvn package)来运行项目，并将其部署到Servlet容器(例如Jetty或Tomcat)上。

我们在浏览器中访问[http://localhost:8080/HelloWorld/](http://localhost:8080/HelloWorld/)将出现一个空白页面，其中显示Hello World!消息。

## 4. Wicket组件

Wicket中的组件是由Java类、HTML标签和模型组成的三元组，模型是组件用来访问数据的门面。

这种结构提供了很好的关注点分离，并通过将组件与数据中心操作分离，提高了代码重用性。

以下示例演示了如何向组件添加Ajax行为，它由一个页面组成，该页面包含两个元素：下拉菜单和标签。当下拉列表选项发生变化时，标签(且只有标签)会更新。

HTML文件CafeSelector.html的主体非常小，仅包含两个元素、一个下拉菜单和一个标签：

```html
<select wicket:id="cafes"></select>
<p>
    Address: <span wicket:id="address">address</span>
</p>
```

在Java端，让我们创建标签：

```java
Label addressLabel = new Label("address", new PropertyModel<String>(this.address, "address"));
addressLabel.setOutputMarkupId(true);
```

Label构造函数中的第一个参数与HTML文件中指定的wicket:id匹配，第二个参数是组件的模型，即组件中呈现的底层数据的包装器。

setOutputMarkupId方法使组件可以通过Ajax进行修改，现在让我们创建下拉列表并向其中添加Ajax行为：

```java
DropDownChoice<String> cafeDropdown = new DropDownChoice<>(
    "cafes", 
    new PropertyModel<String>(this, "selectedCafe"), 
    cafeNames);
cafeDropdown.add(new AjaxFormComponentUpdatingBehavior("onchange") {
    @Override
    protected void onUpdate(AjaxRequestTarget target) {
        String name = (String) cafeDropdown.getDefaultModel().getObject();
        address.setAddress(cafeNamesAndAddresses.get(name).getAddress());
        target.add(addressLabel);
    }
});
```

创建方法与标签类似，构造函数接收wicket id、模型和cafeNames。

然后添加AjaxFormComponentUpdatingBehavior，并添加onUpdate回调方法，一旦发出ajax请求，该方法就会更新标签的模型。最后，将标签组件设置为刷新目标。

最后将标签组件设置为刷新的目标。

如你所见，所有内容都是Java，无需一行Javascript。为了更改标签显示的内容，我们只需修改POJO。修改Java对象转化为网页更改的机制是在幕后进行的，与开发人员无关。

Wicket提供了大量现成的支持AJAX的组件，[此处](https://wicket.apache.org/learn/examples/index.html)提供了包含实例的组件目录。

## 5. 总结

在这篇介绍性文章中，我们介绍了Java中基于组件的Web框架Wicket的基础知识。

Wicket提供了一个抽象层，旨在完全废除管道代码。
