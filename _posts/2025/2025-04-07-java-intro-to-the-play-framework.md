---
layout: post
title:  Play简介
category: webmodules
copyright: webmodules
excerpt: Play
---

## 1. 概述

本入门教程的目的是探索Play框架并了解如何使用它创建Web应用程序。

Play是一个高效的Web应用程序框架，适用于在JVM上编译和运行代码的编程语言，主要是Java和Scala，它集成了现代Web应用程序开发所需的组件和API。

## 2. Play框架设置

让我们前往Play框架的[官方页面](https://www.playframework.com/getting-started)并下载最新版本的发行版，在本教程中，最新版本是2.7。

我们将下载Play Java Hello World教程zip文件夹并将文件解压到方便的位置。在此文件夹的根目录中，我们将找到一个sbt可执行文件，可以使用它来运行该应用程序。或者，我们可以从其[官方页面](https://www.scala-sbt.org/)安装sbt。

要从下载的文件夹中使用sbt，请执行以下操作：

```shell
cd /path/to/folder/
./sbt run
```

请注意，我们正在当前目录中运行脚本，因此使用./语法。

如果我们安装了sbt，那么可以这样：

```shell
cd /path/to/folder/
sbt run
```

运行此命令后，我们将看到一条语句，内容为“(Server started, use Enter to stop and go back to the console...)”，这意味着我们的应用程序已准备就绪，因此我们现在可以转到[http://localhost:9000](http://localhost:9000/)，将在其中看到Play欢迎页面：

![](/assets/images/2025/webmodules/javaintrototheplayframework01.png)

## 3. Play应用程序的剖析

在本节中，我们将更好地了解Play应用程序的结构以及该结构中每个文件和目录的用途。

这些是我们在典型的Play Framework应用程序中找到的文件和文件夹：

```text
├── app                      → Application sources
│   ├── assets               → Compiled Asset sources
│   │   ├── javascripts      → Typically Coffee Script sources
│   │   └── stylesheets      → Typically LESS CSS sources
│   ├── controllers          → Application controllers
│   ├── models               → Application business layer
│   └── views                → Templates
├── build.sbt                → Application build script
├── conf                     → Configurations files and other non-compiled resources (on classpath)
│   ├── application.conf     → Main configuration file
│   └── routes               → Routes definition
├── dist                     → Arbitrary files to be included in your projects distribution
├── lib                      → Unmanaged libraries dependencies
├── logs                     → Logs folder
│   └── application.log      → Default log file
├── project                  → sbt configuration files
│   ├── build.properties     → Marker for sbt project
│   └── plugins.sbt          → sbt plugins including the declaration for Play itself
├── public                   → Public assets
│   ├── images               → Image files
│   ├── javascripts          → Javascript files
│   └── stylesheets          → CSS files
├── target                   → Generated files
│   ├── resolution-cache     → Information about dependencies
│   ├── scala-2.11            
│   │   ├── api              → Generated API docs
│   │   ├── classes          → Compiled class files
│   │   ├── routes           → Sources generated from routes
│   │   └── twirl            → Sources generated from templates
│   ├── universal            → Application packaging
│   └── web                  → Compiled web assets
└── test                     → source folder for unit or functional tests

```

### 3.1 app目录

该目录包含Java源代码、Web模板和编译资产的源-基本上是所有源和所有可执行资源。

app目录包含一些重要的子目录，每个子目录都打包了MVC架构模式的一部分：

- models：这是应用程序业务层，此包中的文件可能会为我们的数据库表建模，并使我们能够访问持久层
- views：所有可以呈现给浏览器的HTML模板都包含在此文件夹
- controllers：一个子目录，其中存放着我们的控制器。控制器是Java源文件，其中包含每个API调用要执行的操作，操作是处理HTTP请求并返回与HTTP响应相同的结果的公共方法
- assets：包含已编译资产(如CSS和Javascript)的子目录。上述命名约定非常灵活，我们可以创建自己的包，例如app/utils包。我们还可以自定义包命名app/cn/tuyucheng/taketoday/controllers

它还包含特定应用程序所需的可选文件和目录。

### 3.2 public目录

存储在public目录中的资源是静态资产，由Web服务器直接提供服务。

此目录通常包含三个子目录，分别用于存放图片、CSS和JavaScript文件。建议像这样组织资源文件，以确保所有Play应用中的一致性。

### 3.3 conf目录

conf目录包含应用程序配置文件，application.conf是我们将Play应用程序的大多数配置属性放置到的位置，我们将在routes中定义应用程序的端点。

如果应用程序需要任何额外的配置文件，则应将它们放在此目录中。

### 3.4 lib目录

lib目录是可选的，包含非托管库依赖，如果我们有任何未在构建系统中指定的jar，我们会将它们放在此目录中，它们将自动添加到应用程序类路径中。

### 3.5 build.sbt文件

build.sbt文件是应用程序构建脚本，在这里列出运行应用程序所需的依赖，例如测试和持久层库。

### 3.6 project目录

所有基于SBT配置构建过程的文件均位于project目录中。

### 3.7 target目录

该目录包含构建系统生成的所有文件-例如所有.class文件。

了解并探索了我们刚刚下载的Play框架Hello World示例的目录结构后，我们现在可以使用示例来了解该框架的基础知识。

## 4. 简单示例

在本节中，我们将创建一个非常基本的Web应用程序示例。

我们不下载示例项目并从中构建，而是看看**使用sbt new命令创建Play框架应用程序的另一种方法**。

让我们打开命令提示符，导航到选择的位置，然后执行以下命令：

```shell
sbt new playframework/play-java-seed.g8
```

为此，我们需要已经安装sbt，如[第2节](https://www.baeldung.com/java-intro-to-the-play-framework#anatomy)中所述。

上述命令首先会提示我们输入项目名称，接下来，它会要求输入将用于包的域(反向，这是Java中的包命名约定)。

使用此命令生成的应用程序具有与之前生成的应用程序相同的结构，因此，我们可以像以前一样继续运行该应用程序：

```shell
cd /path/to/folder/ 
sbt run
```

上述命令执行完成后，将在端口号9000上生成一个服务器来公开我们的API，**我们可以通过[http://localhost:9000](http://localhost:9000/)访问它，然后应该在浏览器中看到“Welcome to Play”的消息**。

**我们的新API有两个端点，可以从浏览器中依次试用它们**。第一个端点是根端点，它会加载一个带有“Welcome to Play!”消息的索引页。

第二个端点位于http://localhost:9000/assets，用于通过在路径中添加文件名来从服务器下载文件。我们可以通过在http://localhost:9000/assets/images/favicon.png获取随应用程序下载的favicon.png文件来测试此端点。

## 5. 操作和控制器

控制器类中的Java方法处理请求参数并产生要发送给客户端的结果，称为操作。

控制器是一个扩展play.mvc.Controller的Java类，它将可能与它们为客户端产生的结果相关的操作逻辑地分组在一起。

现在让我们转到app-parent-dir/app/controllers并关注HomeController.java。

HomeController的index操作返回一个带有简单欢迎消息的网页：

```java
public Result index() {
    return ok(views.html.index.render());
}
```

该网页是views包中的默认index模板：

```html
@main("Welcome to Play") {
  <h1>Welcome to Play!</h1>
}
```

如上所示，index页面调用main模板。然后，main模板处理页面标题和正文标签的呈现。它需要两个参数：一个用于页面标题的String和一个用于插入页面正文的Html对象。

```html
@(title: String)(content: Html)

<!DOCTYPE html>
<html lang="en">
    <head>
        @* Here's where we render the page title `String`. *@
        <title>@title</title>
        <link rel="stylesheet" media="screen" href="@routes.Assets.versioned("stylesheets/main.css")">
        <link rel="shortcut icon" type="image/png" href="@routes.Assets.versioned("images/favicon.png")">
    </head>
    <body>
        @* And here's where we render the `Html` object containing
         * the page content. *@
        @content

        <script src="@routes.Assets.versioned("javascripts/main.js")" type="text/javascript"></script>
    </body>
</html>
```

让我们稍微改变一下index文件中的文本：

```html
@main("Welcome to Tuyucheng") {
  <h1>Welcome to Play Framework Tutorial on Tuyucheng!</h1>
}
```

重新加载浏览器将会看到一个粗体标题：

```text
Welcome to Play Framework Tutorial on Tuyucheng!
```

**我们可以通过删除HomeController的index()方法中的render指令来完全废除模板，以便我们可以直接返回纯文本或HTML文本**：

```java
public Result index() {
    return ok("REST API with Play by Tuyucheng");
}
```

编辑代码后，如上所示，浏览器中将只有文本。这只是纯文本，没有任何HTML或样式：

```text
REST API with Play by Tuyucheng
```

我们也可以通过将文本包装在标题<h1\></h1\>标签中，然后将HTML文本传递给Html.apply方法来输出HTML。

让我们在路由中添加一个/tuyucheng/html端点：

```text
GET    /tuyucheng/html    controllers.HomeController.applyHtml
```

现在让我们创建处理此端点上的请求的控制器：

```java
public Result applyHtml() {
    return ok(Html.apply("<h1>This text will appear as a heading 1</h1>"));
}
```

当我们访问http://localhost:9000/tuyucheng/html时，我们将看到以HTML格式显示的上述文本。

我们通过自定义响应类型来操纵响应，我们将在后面的部分中更详细地介绍此功能。

我们还看到了Play框架的另外两个重要功能。

首先，重新加载浏览器会反映我们代码的最新版本；这是因为我们的代码更改是动态编译的。

其次，Play在play.mvc.Results类中为我们提供了标准HTTP响应的辅助方法。例如，ok()方法，它返回OK HTTP 200响应以及我们作为参数传递给它的响应主体。

Results类中还有更多辅助方法，例如notFound()和badRequest()。

## 6. 操纵结果

**Play会自动从响应主体中推断出响应内容类型**，这就是为什么我们能够在ok方法中返回文本的原因：

```java
return ok("text to display");
```

然后Play会自动将Content-Type标头设置为text/plain，虽然这在大多数情况下都有效，但我们可以接管控制并自定义Content-Type标头。

我们**将HomeController.customContentType操作的响应自定义为text/html**：

```java
public Result customContentType() {
    return ok("This is some text content").as("text/html");
}
```

此模式适用于所有类型的内容，根据传递给ok辅助方法的数据格式，我们可以用text/plain或application/json替换text/html。

也可以设置标头：

```java
public Result setHeaders() {
    return ok("This is some text content")
            .as("text/html")
            .withHeader("Header-Key", "Some value");
}
```

## 7. 总结

在本文中，我们使用Play创建了一个基本的Java Web应用程序。